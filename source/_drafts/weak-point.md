---
title: weak-point
comments: true
date: 2019-10-21 11:10:56
tags: iOS
---

weak 指针学习
<!--more-->

## 前置学习

[Objective-C 引用计数](https://skybrim.top/2017/10/19/iOS/reference-counting/)


## __weak

__weak 修饰的指针指向对象，对象的引用计数不会增加

当对象释放时，__weak 修饰的指向对象的指针，会变成 nil


## 先上结论

Runtime 维护着一个全局的 SideTables 定长哈希表，key 是对象的地址，值是对象对应的 SideTable

SideTable 里维护这对象的 引用计数表（RefcountMap）和 **弱指针全局表（weak_table_t）**

weak_table_t 中，通过对象的地址，获取到对象的 **弱指针数组（weak_entry_t *）**

创建 weak 指针，就是向这个数组里添加 weak 指针的地址

对象释放，weak 指针置为 nil，就是清空该数组


## SideTables 结构图

![SideTables](https://raw.githubusercontent.com/skybrim/AllImages/dev/weak_point_0.png)


## weak 指针的创建

### objc_initWeak

```objectivec
// location： weak 指针地址，类型 id*
// newObj: 所指向的对象， 类型 id
id
objc_initWeak(id *location, id newObj)
{
    if (!newObj) {
        *location = nil;
        return nil;
    }

    return storeWeak<DontHaveOld, DoHaveNew, DoCrashIfDeallocating>
        (location, (objc_object*)newObj);
}
```

### storeWeak

```objectivec
// Update a weak variable.
// If HaveOld is true, the variable has an existing value 
//   that needs to be cleaned up. This value might be nil.
// If HaveNew is true, there is a new value that needs to be 
//   assigned into the variable. This value might be nil.
// If CrashIfDeallocating is true, the process is halted if newObj is 
//   deallocating or newObj's class does not support weak references. 
//   If CrashIfDeallocating is false, nil is stored instead.
enum CrashIfDeallocating {
    DontCrashIfDeallocating = false, DoCrashIfDeallocating = true
};
template <HaveOld haveOld, HaveNew haveNew,
          CrashIfDeallocating crashIfDeallocating>
static id 
storeWeak(id *location, objc_object *newObj)
{
    assert(haveOld  ||  haveNew);
    if (!haveNew) assert(newObj == nil);

    Class previouslyInitializedClass = nil;
    id oldObj;
    SideTable *oldTable;
    SideTable *newTable;

    // Acquire locks for old and new values.
    // Order by lock address to prevent lock ordering problems. 
    // Retry if the old value changes underneath us.
 retry:

    // weak指针 之前引用过一个 obj，取出 obj 对应的 SideTable，赋值给 oldTable
    // 没有引用过对象，则 oldTable = nil
    if (haveOld) {
        oldObj = *location;
        oldTable = &SideTables()[oldObj];
    } else {
        oldTable = nil;
    }

    // weak指针 要引用一个新的 obj，取出 obj 对应的 SideTable，赋值 newTable
    if (haveNew) {
        newTable = &SideTables()[newObj];
    } else {
        newTable = nil;
    }

    // 加锁操作
    SideTable::lockTwo<haveOld, haveNew>(oldTable, newTable);

    // 如果 location 与 oldObj 不同，可能被其他线程所修改
    if (haveOld  &&  *location != oldObj) {
        SideTable::unlockTwo<haveOld, haveNew>(oldTable, newTable);
        goto retry;
    }

    // Prevent a deadlock between the weak reference machinery
    // and the +initialize machinery by ensuring that no 
    // weakly-referenced object has an un-+initialized isa.
    if (haveNew  &&  newObj) {
        Class cls = newObj->getIsa();

        // 如果cls还没有初始化，先初始化，再尝试设置弱引用
        if (cls != previouslyInitializedClass  &&  
            !((objc_class *)cls)->isInitialized()) 
        {
            SideTable::unlockTwo<haveOld, haveNew>(oldTable, newTable);
            _class_initialize(_class_getNonMetaClass(cls, (id)newObj));

            // If this class is finished with +initialize then we're good.
            // If this class is still running +initialize on this thread 
            // (i.e. +initialize called storeWeak on an instance of itself)
            // then we may proceed but it will appear initializing and 
            // not yet initialized to the check above.
            // Instead set previouslyInitializedClass to recognize it on retry.
            // 完成初始化后进行标记
            previouslyInitializedClass = cls;

            // newObj 初始化后，重新获取一遍newObj
            goto retry;
        }
    }

    // Clean up old value, if any.
    // 如果 weak指针 之前弱引用过别的对象oldObj
    // weak_unregister_no_lock，移除 oldTable->weak_table->weak_entries 中该 weak指针 地址
    if (haveOld) {
        weak_unregister_no_lock(&oldTable->weak_table, oldObj, location);
    }

    // Assign new value, if any.
    // 如果 weak指针 需要弱引用新的对象 newObj
    // weak_register_no_lock，将 weak指针 的地址添加到 newObj 对应的 newTable->weak_table->weak_entries
    if (haveNew) {
        newObj = (objc_object *)
            weak_register_no_lock(&newTable->weak_table, (id)newObj, location, 
                                  crashIfDeallocating);
        // weak_register_no_lock returns nil if weak store should be rejected

        // Set is-weakly-referenced bit in refcount table.
        // 更新 newObj 的 isa指针 的 weakly_referenced bit 标志位
        if (newObj  &&  !newObj->isTaggedPointer()) {
            newObj->setWeaklyReferenced_nolock();
        }

        // Do not set *location anywhere else. That would introduce a race.
        // *location 赋值，也就是将weak指针直接指向了newObj，而且没有将newObj的引用计数+1
        *location = (id)newObj;
    }
    else {
        // No new value. The storage is not changed.
    }
    
    SideTable::unlockTwo<haveOld, haveNew>(oldTable, newTable);

    return (id)newObj;
}
```

### weak_register_no_lock

添加弱引用

```objectivec
id 
weak_register_no_lock(weak_table_t *weak_table, id referent_id, 
                      id *referrer_id, bool crashIfDeallocating) {

    // referent_id: 对象
    // referrer_id: weak指针
    objc_object *referent = (objc_object *)referent_id;   // 对象的地址
    objc_object **referrer = (objc_object **)referrer_id; // weak指针的地址

    // 如果对象 referent 为 nil 或者 对象是 TaggedPointer 计数方式，则直接返回
    if (!referent  ||  referent->isTaggedPointer()) return referent_id;

    // ensure that the referenced object is viable
    // 确保对象可用：没有在析构，支持weak弱引用
    bool deallocating;
    if (!referent->ISA()->hasCustomRR()) {
        deallocating = referent->rootIsDeallocating();
    }
    else {
        BOOL (*allowsWeakReference)(objc_object *, SEL) = 
            (BOOL(*)(objc_object *, SEL))
            object_getMethodImplementation((id)referent, 
                                           SEL_allowsWeakReference);
        if ((IMP)allowsWeakReference == _objc_msgForward) {
            return nil;
        }
        deallocating =
            ! (*allowsWeakReference)(referent, SEL_allowsWeakReference);
    }

    // 如果对象正在析构，不能够被弱引用
    if (deallocating) {
        if (crashIfDeallocating) {
            _objc_fatal("Cannot form weak reference to instance (%p) of "
                        "class %s. It is possible that this object was "
                        "over-released, or is in the process of deallocation.",
                        (void*)referent, object_getClassName((id)referent));
        } else {
            return nil;
        }
    }

    // now remember it and where it is being stored
    // 在 weak_table 中找到对象 referent 对应的 weak_entries，并将 referrer 加入到 weak_entries 中
    weak_entry_t *entry;
    if ((entry = weak_entry_for_referent(weak_table, referent))) {

        // 如果能找到 weak_entries，插入 referrer
        append_referrer(entry, referrer);
    } 
    else {

        // 如果找不到 weak_entries，新建一个 weak_entries
        weak_entry_t new_entry(referent, referrer);
        weak_grow_maybe(weak_table);
        weak_entry_insert(weak_table, &new_entry);
    }

    // Do not set *referrer. objc_storeWeak() requires that the 
    // value not change.

    return referent_id;
}
```

### weak_unregister_no_lock

移除弱引用

```objectivec
void 
weak_unregister_no_lock(weak_table_t *weak_table, id referent_id, 
                        id *referrer_id) {

    // referent_id: 对象
    // referrer_id: weak指针
    objc_object *referent = (objc_object *)referent_id;   // 对象的地址
    objc_object **referrer = (objc_object **)referrer_id; // weak指针的地址

    weak_entry_t *entry;

    if (!referent) return;

    // 查找到以前弱引用的对象 referent 所对应的 weak_entries
    if ((entry = weak_entry_for_referent(weak_table, referent))) {

        // 在对象 referent 所对应的 weak_entries 数组中，移除弱引用 referrer
        remove_referrer(entry, referrer);

        // 移除元素之后， 要检查一下 weak_entries 数组是否已经空了
        bool empty = true;
        if (entry->out_of_line()  &&  entry->num_refs != 0) {
            empty = false;
        }
        else {
            for (size_t i = 0; i < WEAK_INLINE_COUNT; i++) {
                if (entry->inline_referrers[i]) {
                    empty = false; 
                    break;
                }
            }
        }

        // 如果 weak_entries 的hash数组已经空了，则需要将 weak_entries 从 weak_table 中移除
        if (empty) {
            weak_entry_remove(weak_table, entry);
        }
    }

    // Do not set *referrer = nil. objc_storeWeak() requires that the 
    // value not change.
}
```

## weak指针的置空

当对象释放时，weak 指针会自动置空

### rootDealloc

当对象的引用计数为0时，底层会调用 _objc_rootDealloc 方法对对象进行释放

在 _objc_rootDealloc 方法里面会调用 rootDealloc 方法

```objectivec
inline void
objc_object::rootDealloc() {

    // TaggedPointer 类型的对象，直接返回
    if (isTaggedPointer()) return;  // fixme necessary?

    //  当对象 
    // 1使用优化的 isa 计数；            isa.nonpointer
    // 2没有weak引用；                 !isa.weakly_referenced
    // 3没有关联对象；                  !isa.has_assoc
    // 4没有c++析构方法；               !isa.has_cxx_dtor
    // 5没有使用SideTable记录引用计数    !isa.has_sidetable_rc
    // 直接 free
    if (fastpath(isa.nonpointer  &&  
                 !isa.weakly_referenced  &&  
                 !isa.has_assoc  &&  
                 !isa.has_cxx_dtor  &&  
                 !isa.has_sidetable_rc))
    {
        assert(!sidetable_present());
        free(this);
    } 
    else {
        object_dispose((id)this);
    }
}
```
### object_dispose

```objectivec
void *objc_destructInstance(id obj) 
{
    if (obj) {
        // Read all of the flags at once for performance.
        bool cxx = obj->hasCxxDtor();
        bool assoc = obj->hasAssociatedObjects();

        // This order is important.
        // c++ 析构方法
        if (cxx) object_cxxDestruct(obj);
        // 移除关联对象 
        if (assoc) _object_remove_assocations(obj);
        // 执行 clearDeallocating
        obj->clearDeallocating();
    }

    return obj;
}
```

### clearDeallocating

```objectivec
inline void 
objc_object::clearDeallocating()
{
    if (slowpath(!isa.nonpointer)) {
        // Slow path for raw pointer isa.
        // 没有使用优化的 isa 指针，清除 SideTable 中的引用计数的数据
        sidetable_clearDeallocating();
    }
    else if (slowpath(isa.weakly_referenced  ||  isa.has_sidetable_rc)) {
        // Slow path for non-pointer isa with weak refs and/or side table data.
        // 有弱指针 或者 使用 SideTable 管理引用计数
        clearDeallocating_slow();
    }

    assert(!sidetable_present());
}
```

### clearDeallocating_slow

```objectivec
NEVER_INLINE void
objc_object::clearDeallocating_slow()
{
    assert(isa.nonpointer  &&  (isa.weakly_referenced || isa.has_sidetable_rc));

    // 在全局的 SideTables 中，以this指针为key，找到对应的 SideTable
    SideTable& table = SideTables()[this]; 

    table.lock();

    // 如果对象被弱引用
    // 在 SideTable 的 weak_table 中对this进行清理工作
    if (isa.weakly_referenced) { 
        weak_clear_no_lock(&table.weak_table, (id)this); 
    }

    // 如果采用了 SideTable 管理引用计数
    // 在 SideTable 的引用计数中移除
    if (isa.has_sidetable_rc) { 
        table.refcnts.erase(this); 
    }

    table.unlock();
}
```

### weak_clear_no_lock

```objectivec
void 
weak_clear_no_lock(weak_table_t *weak_table, id referent_id) 
{
    // 获取对象的地址
    objc_object *referent = (objc_object *)referent_id;

    // 找到 referent 在 weak_table 中对应的 weak_entries
    weak_entry_t *entry = weak_entry_for_referent(weak_table, referent); 
    if (entry == nil) {
        /// XXX shouldn't happen, but does with mismatched CF/objc
        //printf("XXX no entry for clear deallocating %p\n", referent);
        return;
    }

    // zero out references
    weak_referrer_t *referrers;
    size_t count;
    
    // 找出对象的 weak指针地址数组 以及 数组长度
    if (entry->out_of_line()) {
        referrers = entry->referrers;
        count = TABLE_SIZE(entry);
    } 
    else {
        referrers = entry->inline_referrers;
        count = WEAK_INLINE_COUNT;
    }
    
    for (size_t i = 0; i < count; ++i) {
        objc_object **referrer = referrers[i]; // 取出每个weak ptr的地址
        if (referrer) {
            if (*referrer == referent) { // 如果weak ptr确实weak引用了referent，则将weak ptr设置为nil，这也就是为什么weak 指针会自动设置为nil的原因
                *referrer = nil;
            }
            else if (*referrer) { // 如果所存储的weak ptr没有weak 引用referent，这可能是由于runtime代码的逻辑错误引起的，报错
                _objc_inform("__weak variable at %p holds %p instead of %p. "
                             "This is probably incorrect use of "
                             "objc_storeWeak() and objc_loadWeak(). "
                             "Break on objc_weak_error to debug.\n", 
                             referrer, (void*)*referrer, (void*)referent);
                objc_weak_error();
            }
        }
    }
    
    weak_entry_remove(weak_table, entry); // 由于referent要被释放了，因此referent的weak_entry_t也要移除出weak_table
}
```