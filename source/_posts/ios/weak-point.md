---
title: Objective-C 弱引用
comments: true
date: 2019-10-21 11:10:56
tags: iOS
---

weak 指针学习
<!--more-->

## 前置学习

[Objective-C 引用计数](https://skybrim.top/2019/10/19/iOS/reference-counting/)


## __weak

__weak 修饰的指针指向对象，对象的引用计数不会增加

当对象释放时，__weak 修饰的指向对象的指针，会变成 nil


## 先上结论

Runtime 维护着一个全局的 SideTables 定长哈希表，key 是对象的地址，值是对象对应的 SideTable

SideTable 里维护这对象的 引用计数表（RefcountMap）和 **弱指针表（weak_table_t）**

weak_table_t 中，通过对象的地址，获取到对象的 **弱指针数组（weak_entry_t *）**

创建 weak 指针，就是向这个数组里添加 weak 指针的地址

对象释放，weak 指针置为 nil，就是清空该数组


## SideTables 结构图

**详细的结构分析，见上一篇博文 [Objective-C 引用计数](https://skybrim.top/2019/10/19/iOS/reference-counting/)**

此文重点是 weak_table_t 与 weak_entry_t 的分析

![SideTables](https://raw.githubusercontent.com/skybrim/AllImages/dev/weak_point_0.png)


### weak_table_t

weak_table_t 是一个全局的弱引用哈希表。

weak_table_t 的 key 是对象地址，value 是 weak_entry_t

由于 SideTables 一共只有 64（iPhone真机为8）个节点，App 中对象肯定不止，所以每个 SideTable 对应多个对象。

所以，weak_table_t 也是对应着多个对象的弱引用信息，即 weak_entries 中存放着多个对象的弱引用信息（weak_entry_t）。

而 weak_table_t，则是通过对 **对象指针** 进行哈希计算，获取到 **对象的 weak_entry_t**

weak_table_t 定义
```objectivec
/**
 * The global weak references table. Stores object ids as keys,
 * and weak_entry_t structs as their values.
 */
struct weak_table_t {
    weak_entry_t *weak_entries;
    size_t    num_entries;
    uintptr_t mask;
    uintptr_t max_hash_displacement;
};
```

分析 weak_entry_for_referent 方法，可以得知 weak_table 通过 对象指针 获取到对象的弱引用信息（weak_entry_t）

```objectivec
/** 
 * Return the weak reference table entry for the given referent. 
 * If there is no entry for referent, return NULL. 
 * Performs a lookup.
 *
 * @param weak_table 
 * @param referent The object. Must not be nil.
 * 
 * @return The table of weak referrers to this object. 
 */
static weak_entry_t *
weak_entry_for_referent(weak_table_t *weak_table, objc_object *referent)
{
    ASSERT(referent);

    weak_entry_t *weak_entries = weak_table->weak_entries;

    if (!weak_entries) return nil;

    // 计算出对象的 weak_entry_t 在 weak_entries 中的索引
    size_t begin = hash_pointer(referent) & weak_table->mask;
    size_t index = begin;
    size_t hash_displacement = 0;
    while (weak_table->weak_entries[index].referent != referent) {
        index = (index+1) & weak_table->mask;
        if (index == begin) bad_weak_table(weak_table->weak_entries);
        hash_displacement++;
        if (hash_displacement > weak_table->max_hash_displacement) {
            return nil;
        }
    }
    
    return &weak_table->weak_entries[index];
}
```

### weak_entry_t

weak_entry_t 维护着对象的弱引用信息的哈希集合

是一个 union 类型，如果 out_of_line_ness != REFERRERS_OUT_OF_LINE ，用一个内联数组代替

这是因为对象可能不止一个弱指针，根据不同情况来优化

通过分析 remove_referrer 和 append_referrer 源码可以得知

weak_entry_t 通过对 弱指针的指针 进行哈希计算，可以找到 弱指针 在 weak_referrer_t->referrers 中的位置

```objectivec
// The address of a __weak variable.
// These pointers are stored disguised so memory analysis tools
// don't see lots of interior pointers from the weak table into objects.
typedef DisguisedPtr<objc_object *> weak_referrer_t;

#if __LP64__
#define PTR_MINUS_2 62
#else
#define PTR_MINUS_2 30
#endif

/**
 * The internal structure stored in the weak references table. 
 * It maintains and stores
 * a hash set of weak references pointing to an object.
 * If out_of_line_ness != REFERRERS_OUT_OF_LINE then the set
 * is instead a small inline array.
 */
#define WEAK_INLINE_COUNT 4

// out_of_line_ness field overlaps with the low two bits of inline_referrers[1].
// inline_referrers[1] is a DisguisedPtr of a pointer-aligned address.
// The low two bits of a pointer-aligned DisguisedPtr will always be 0b00
// (disguised nil or 0x80..00) or 0b11 (any other address).
// Therefore out_of_line_ness == 0b10 is used to mark the out-of-line state.
#define REFERRERS_OUT_OF_LINE 2

struct weak_entry_t {
    DisguisedPtr<objc_object> referent;
    union {
        struct {
            weak_referrer_t *referrers;
            uintptr_t        out_of_line_ness : 2;
            uintptr_t        num_refs : PTR_MINUS_2;
            uintptr_t        mask;
            uintptr_t        max_hash_displacement;
        };
        struct {
            // out_of_line_ness field is low bits of inline_referrers[1]
            weak_referrer_t  inline_referrers[WEAK_INLINE_COUNT];
        };
    };

    bool out_of_line() {
        return (out_of_line_ness == REFERRERS_OUT_OF_LINE);
    }

    weak_entry_t& operator=(const weak_entry_t& other) {
        memcpy(this, &other, sizeof(other));
        return *this;
    }

    weak_entry_t(objc_object *newReferent, objc_object **newReferrer)
        : referent(newReferent)
    {
        inline_referrers[0] = newReferrer;
        for (int i = 1; i < WEAK_INLINE_COUNT; i++) {
            inline_referrers[i] = nil;
        }
    }
};
```

```objectivec
/** 
 * Add the given referrer to set of weak pointers in this entry.
 * Does not perform duplicate checking (b/c weak pointers are never
 * added to a set twice). 
 *
 * @param entry The entry holding the set of weak pointers. 
 * @param new_referrer The new weak pointer to be added.
 */
static void append_referrer(weak_entry_t *entry, objc_object **new_referrer)
{
    if (! entry->out_of_line()) {
        // Try to insert inline.
        for (size_t i = 0; i < WEAK_INLINE_COUNT; i++) {
            if (entry->inline_referrers[i] == nil) {
                entry->inline_referrers[i] = new_referrer;
                return;
            }
        }

        // Couldn't insert inline. Allocate out of line.
        weak_referrer_t *new_referrers = (weak_referrer_t *)
            calloc(WEAK_INLINE_COUNT, sizeof(weak_referrer_t));
        // This constructed table is invalid, but grow_refs_and_insert
        // will fix it and rehash it.
        for (size_t i = 0; i < WEAK_INLINE_COUNT; i++) {
            new_referrers[i] = entry->inline_referrers[i];
        }
        entry->referrers = new_referrers;
        entry->num_refs = WEAK_INLINE_COUNT;
        entry->out_of_line_ness = REFERRERS_OUT_OF_LINE;
        entry->mask = WEAK_INLINE_COUNT-1;
        entry->max_hash_displacement = 0;
    }

    ASSERT(entry->out_of_line());

    if (entry->num_refs >= TABLE_SIZE(entry) * 3/4) {
        return grow_refs_and_insert(entry, new_referrer);
    }

    // 对 弱指针的指针 进行哈希计算，拿到索引，将对应的 弱指针 插入到 weak_referrer_t->referrers 中
    size_t begin = w_hash_pointer(new_referrer) & (entry->mask);
    size_t index = begin;
    size_t hash_displacement = 0;
    while (entry->referrers[index] != nil) {
        hash_displacement++;
        index = (index+1) & entry->mask;
        if (index == begin) bad_weak_table(entry);
    }
    if (hash_displacement > entry->max_hash_displacement) {
        entry->max_hash_displacement = hash_displacement;
    }
    weak_referrer_t &ref = entry->referrers[index];
    ref = new_referrer;
    entry->num_refs++;
}

/** 
 * Remove old_referrer from set of referrers, if it's present.
 * Does not remove duplicates, because duplicates should not exist. 
 * 
 * @todo this is slow if old_referrer is not present. Is this ever the case? 
 *
 * @param entry The entry holding the referrers.
 * @param old_referrer The referrer to remove. 
 */
static void remove_referrer(weak_entry_t *entry, objc_object **old_referrer)
{
    if (! entry->out_of_line()) {
        for (size_t i = 0; i < WEAK_INLINE_COUNT; i++) {
            if (entry->inline_referrers[i] == old_referrer) {
                entry->inline_referrers[i] = nil;
                return;
            }
        }
        _objc_inform("Attempted to unregister unknown __weak variable "
                     "at %p. This is probably incorrect use of "
                     "objc_storeWeak() and objc_loadWeak(). "
                     "Break on objc_weak_error to debug.\n", 
                     old_referrer);
        objc_weak_error();
        return;
    }

    size_t begin = w_hash_pointer(old_referrer) & (entry->mask);
    size_t index = begin;
    size_t hash_displacement = 0;
    while (entry->referrers[index] != old_referrer) {
        index = (index+1) & entry->mask;
        if (index == begin) bad_weak_table(entry);
        hash_displacement++;
        if (hash_displacement > entry->max_hash_displacement) {
            _objc_inform("Attempted to unregister unknown __weak variable "
                         "at %p. This is probably incorrect use of "
                         "objc_storeWeak() and objc_loadWeak(). "
                         "Break on objc_weak_error to debug.\n", 
                         old_referrer);
            objc_weak_error();
            return;
        }
    }
    entry->referrers[index] = nil;
    entry->num_refs--;
}
```

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

        // append_referrer() 的源码在文章上半部分
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
        // 移除关联对象，并将其自身从Association Manager的map中移除
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