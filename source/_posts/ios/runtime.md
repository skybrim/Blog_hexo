---
title: Objective-C runtime
comments: true
date: 2020-05-22 14:02:51
tags: iOS
---

runtime
源码基于 [objc4-781](https://opensource.apple.com/tarballs/objc4/objc4-781.tar.gz)
<!--more-->


## 基本定义

### 对象

![](https://cdn.jsdelivr.net/gh/skybrim/AllImages@dev/run_time-1.jpg)

* 对象：objc_object 结构体，isa 指向类

* 类：  objc_class 结构体，继承自 objc_object，isa 指向元类

* 元类：objc_class 结构体，isa 指向**根元类**，
  
* **根元类的 isa 指向自身**，根元类的父类指向根类（NSObject），根类（NSObject）的父类指向 nil

```objectivec
typedef struct objc_class *Class;
typedef struct objc_object *id;

struct objc_object {
private:
    isa_t isa;

public:

    // ISA() assumes this is NOT a tagged pointer object
    Class ISA();

    // getIsa() allows this to be a tagged pointer object
    Class getIsa();

    // 省略一些方法
    ...
};

struct objc_class : objc_object {
    // Class ISA;
    Class superclass;          // objec_class 类型的指针，指向父类的 objc_class
    cache_t cache;             // 已经调用过的方法的缓存
    class_data_bits_t bits;    // uintptr_t(unsigned long) 类型， 存储了 class_rw_t 的地址，objc_class 内部定义的一些函数，通过操作 bits 实现

    class_rw_t *data() { 
        return bits.data();
    }
    // 省略一些方法
    ...
};
```

### Method

```objectivec
typedef struct method_t *Method;

struct method_t {
    // 函数名，本质是字符串
    SEL name;
    // 函数类型，返回值与参数的编码 
    const char *types;
    // 函数指针
    IMP imp;

    struct SortBySELAddress :
        public std::binary_function<const method_t&,
                                    const method_t&, bool>
    {
        bool operator() (const method_t& lhs,
                         const method_t& rhs)
        { return lhs.name < rhs.name; }
    };
};
```

[函数类型编码](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtTypeEncodings.html)

### Property

```objectivec
typedef struct property_t *objc_property_t;

struct property_t {
    const char *name;
    const char *attributes;
};
```

## Ivar

```objectivec
typedef struct ivar_t *Ivar;

struct ivar_t {
    int32_t *offset; // 一个 int 指针，而不是 int，通过这样设计，即使父类新增变量，子类不需要重新编译
    const char *name;
    const char *type;
    // alignment is sometimes -1; use alignment() instead
    uint32_t alignment_raw;
    uint32_t size;

    uint32_t alignment() const {
        if (alignment_raw == ~(uint32_t)0) return 1U << WORD_SHIFT;
        return 1 << alignment_raw;
    }
};
```

[Objective-C 类成员变量深度剖析](http://quotation.github.io/objc/2015/05/21/objc-runtime-ivar-access.html)

补充：

```objectivec
- (void)doSomething:(SomeClass *)obj
{
    obj->ivar1 = 42;         // 访问obj对象的public成员变量
    int n = self->ivar2;     // 访问当前类实例的成员变量
    ivar2 = n + 1;           // 访问当前类的成员变量
}
```

为基类**动态增加成员变量**会导致所有已创建出的子类实例都无法使用，所以成员变量是在编译时决定的

但是可以**动态添加方法和属性**，因为方法和属性属于类，成员变量属于实例

### Category

```objectivec
typedef struct category_t *Category;

struct category_t {
    // 所属的类名，而不是Category的名字
    const char *name;
    // 所属的类，这个类型是编译期的类，这时候类还没有被重映射
    classref_t cls;
    // 实例方法列表
    struct method_list_t *instanceMethods;
    // 类方法列表
    struct method_list_t *classMethods;
    // 协议列表
    struct protocol_list_t *protocols;
    // 实例属性列表
    struct property_list_t *instanceProperties;
    // Fields below this point are not always present on disk.
    // 类属性列表
    struct property_list_t *_classProperties;

    method_list_t *methodsForMeta(bool isMeta) {
        if (isMeta) return classMethods;
        else return instanceMethods;
    }

    property_list_t *propertiesForMeta(bool isMeta, struct header_info *hi);
};
```

### Protocol

```objectivec
struct protocol_t : objc_object {
    const char *mangledName;
    struct protocol_list_t *protocols;
    method_list_t *instanceMethods;
    method_list_t *classMethods;
    method_list_t *optionalInstanceMethods;
    method_list_t *optionalClassMethods;
    property_list_t *instanceProperties;
    uint32_t size;   // sizeof(protocol_t)
    uint32_t flags;
    // Fields below this point are not always present on disk.
    const char **_extendedMethodTypes;
    const char *_demangledName;
    property_list_t *_classProperties;

    const char *demangledName();

    const char *nameForLogging() {
        return demangledName();
    }

    bool isFixedUp() const;
    void setFixedUp();

    bool isCanonical() const;
    void clearIsCanonical();

#   define HAS_FIELD(f) (size >= offsetof(protocol_t, f) + sizeof(f))

    bool hasExtendedMethodTypesField() const {
        return HAS_FIELD(_extendedMethodTypes);
    }
    bool hasDemangledNameField() const {
        return HAS_FIELD(_demangledName);
    }
    bool hasClassPropertiesField() const {
        return HAS_FIELD(_classProperties);
    }

#   undef HAS_FIELD

    const char **extendedMethodTypes() const {
        return hasExtendedMethodTypesField() ? _extendedMethodTypes : nil;
    }

    property_list_t *classProperties() const {
        return hasClassPropertiesField() ? _classProperties : nil;
    }
};
```

##  runtime 结构体分析

### isa_t

```objectivec
union isa_t {
    isa_t() { }
    isa_t(uintptr_t value) : bits(value) { }

    Class cls;
    uintptr_t bits;
#if defined(ISA_BITFIELD)
    struct {
        ISA_BITFIELD;  // defined in isa.h
    };
#endif
};

// isa.h ISA_BITFIELD 的定义
#if SUPPORT_PACKED_ISA

    // extra_rc must be the MSB-most field (so it matches carry/overflow flags)
    // nonpointer must be the LSB (fixme or get rid of it)
    // shiftcls must occupy the same bits that a real class pointer would
    // bits + RC_ONE is equivalent to extra_rc + 1
    // RC_HALF is the high bit of extra_rc (i.e. half of its range)

    // future expansion:
    // uintptr_t fast_rr : 1;     // no r/r overrides
    // uintptr_t lock : 2;        // lock for atomic property, @synch
    // uintptr_t extraBytes : 1;  // allocated with extra bytes

# if __arm64__
#   define ISA_MASK        0x0000000ffffffff8ULL
#   define ISA_MAGIC_MASK  0x000003f000000001ULL
#   define ISA_MAGIC_VALUE 0x000001a000000001ULL
#   define ISA_BITFIELD                                                                               \
      uintptr_t nonpointer        : 1;  /*0 表示普通的 isa 指针；1 表示优化后的 isa 指针，存储引用计数*/      \
      uintptr_t has_assoc         : 1;  /*表示该对象是否包含 associated object，如果没有，则析构时会更快*/    \
      uintptr_t has_cxx_dtor      : 1;  /*表示该对象是否有 C++ 或 ARC 的析构函数，如果没有，则析构时更快*/     \
      uintptr_t shiftcls          : 33; /*类的指针 MACH_VM_MAX_ADDRESS 0x1000000000*/                  \
      uintptr_t magic             : 6;  /*固定值为 0xd2，用于在调试时分辨对象是否未完成初始化。*/             \
      uintptr_t weakly_referenced : 1;  /*表示该对象是否有过 weak 对象，如果没有，则析构时更快*/              \
      uintptr_t deallocating      : 1;  /*表示该对象是否正在析构*/                                        \
      uintptr_t has_sidetable_rc  : 1;  /*表示该对象的引用计数值是否过大无法存储在 isa 指针  */               \
      uintptr_t extra_rc          : 19  /*存储引用计数值减一后的结果*/ 
#   define RC_ONE   (1ULL<<45)
#   define RC_HALF  (1ULL<<18)

# elif __x86_64__
#   define ISA_MASK        0x00007ffffffffff8ULL
#   define ISA_MAGIC_MASK  0x001f800000000001ULL
#   define ISA_MAGIC_VALUE 0x001d800000000001ULL
#   define ISA_BITFIELD                                                        \
      uintptr_t nonpointer        : 1;                                         \
      uintptr_t has_assoc         : 1;                                         \
      uintptr_t has_cxx_dtor      : 1;                                         \
      uintptr_t shiftcls          : 44; /*MACH_VM_MAX_ADDRESS 0x7fffffe00000*/ \
      uintptr_t magic             : 6;                                         \
      uintptr_t weakly_referenced : 1;                                         \
      uintptr_t deallocating      : 1;                                         \
      uintptr_t has_sidetable_rc  : 1;                                         \
      uintptr_t extra_rc          : 8 
#   define RC_ONE   (1ULL<<56)
#   define RC_HALF  (1ULL<<7)

# else
#   error unknown architecture for packed isa
# endif

// SUPPORT_PACKED_ISA
#endif
```

### class_ro_t 与 class_rw_t

```objectivec
struct class_ro_t {
    uint32_t flags;
    uint32_t instanceStart;
    uint32_t instanceSize;
#ifdef __LP64__
    uint32_t reserved;
#endif
    // strong修饰的ivars
    const uint8_t * ivarLayout;
    
    const char * name;
    method_list_t * baseMethodList;
    protocol_list_t * baseProtocols;
    const ivar_list_t * ivars;
    
    // weak修饰的ivars
    const uint8_t * weakIvarLayout;
    property_list_t *baseProperties;

    method_list_t *baseMethods() const {
        return baseMethodList;
    }
};

struct class_rw_t {
    // Be warned that Symbolication knows the layout of this structure.
    uint32_t flags;
    uint32_t version;

    const class_ro_t *ro;

    method_array_t methods;
    property_array_t properties;
    protocol_array_t protocols;

    Class firstSubclass;
    Class nextSiblingClass;

    char *demangledName;

    //省略了一些方法
    ...
};
```

* class_ro_t 编译时已经决定，只读

* class_rw_t 运行时创建


## class 初始化

1. 编译器取出 bits 中的 class_ro_t，创建 class_rw_t，将 ro 赋值给 rw

2. 初始化父类

3. 初始化元类

4. 初始化方法，将 ro 中的  method_list_t、property_list_t、protocol_list_t 赋值给 rw

### Tagged Pointer

iPhone 5s 起，iOS 引入了 64 位处理器，指针长度为 8 字节。

为了优化内存，推出了 **Tagged Pointer**

Tagged Pointer 中，指针就是**值**

### isa_t

isa_t 同 Tagged Pointer，在 OC2.0 之后，直接本身就存储着对象的信息，查找对象信息时，直接对 isa_t 操作即可。

因此，我们最好通过 ISA() 方法来获取 isa，而不是直接访问 isa。


## 对象初始化

### alloc 

```objectivec
+ (id)alloc {
    // 按照注释 [cls alloc] 调用的是 objc_alloc，不过最终都会走到 callAlloc 方法
    return _objc_rootAlloc(self);
}

// Base class implementation of +alloc. cls is not nil.
// Calls [cls alloc]. 
id
objc_alloc(Class cls)
{
    return callAlloc(cls, true/*checkNil*/, false/*allocWithZone*/);
}

// Call [cls alloc] or [cls allocWithZone:nil], with appropriate 
// shortcutting optimizations.
static ALWAYS_INLINE id
callAlloc(Class cls, bool checkNil, bool allocWithZone=false)
{
#if __OBJC2__
    // 重点关注这一部分，最终都会走到这里
    // 检查是否为 nil 对象调用
    if (slowpath(checkNil && !cls)) return nil;
    // 调用 _objc_rootAllocWithZone
    if (fastpath(!cls->ISA()->hasCustomAWZ())) {
        return _objc_rootAllocWithZone(cls, nil);
    }
#endif

    // No shortcuts available.
    if (allocWithZone) {
        return ((id(*)(id, SEL, struct _NSZone *))objc_msgSend)(cls, @selector(allocWithZone:), nil);
    }
    return ((id(*)(id, SEL))objc_msgSend)(cls, @selector(alloc));
}

NEVER_INLINE
id
_objc_rootAllocWithZone(Class cls, malloc_zone_t *zone __unused)
{
    // allocWithZone under __OBJC2__ ignores the zone parameter
    return _class_createInstanceFromZone(cls, 0, nil,
                                         OBJECT_CONSTRUCT_CALL_BADALLOC);
}

/***********************************************************************
* class_createInstance
* fixme
* Locking: none
*
* Note: this function has been carefully written so that the fastpath
* takes no branch.
**********************************************************************/
static ALWAYS_INLINE id
_class_createInstanceFromZone(Class cls, size_t extraBytes, void *zone,
                              int construct_flags = OBJECT_CONSTRUCT_NONE,
                              bool cxxConstruct = true,
                              size_t *outAllocatedSize = nil)
{
    // 是否实现 cls
    ASSERT(cls->isRealized());

    // Read class's info bits all at once for performance
    // cls 是否有 c++ 的构建方法
    bool hasCxxCtor = cxxConstruct && cls->hasCxxCtor();
    // cls 是否有 c++ 的析构方法
    bool hasCxxDtor = cls->hasCxxDtor();
    // cls 是否开启了 isa 优化
    bool fast = cls->canAllocNonpointer();
    size_t size;

    // 获取实例的内存大小，至少 16 字节，参数 extraBytes 是 0
    size = cls->instanceSize(extraBytes);
    // outAllocatedSize 默认 nil
    if (outAllocatedSize) *outAllocatedSize = size;

    id obj;
    // zone 是 nil
    if (zone) {
        obj = (id)malloc_zone_calloc((malloc_zone_t *)zone, 1, size);
    } else {
        // 根据 size 开辟内存
        obj = (id)calloc(1, size);
    }
    if (slowpath(!obj)) {
        if (construct_flags & OBJECT_CONSTRUCT_CALL_BADALLOC) {
            return _objc_callBadAllocHandler(cls);
        }
        return nil;
    }

    if (!zone && fast) {
        // fast 参数，开启了 isa 优化，初始化 isa_t
        obj->initInstanceIsa(cls, hasCxxDtor);
    } else {
        // Use raw pointer isa on the assumption that they might be
        // doing something weird with the zone or RR.
        // 使用为优化的 isa 指针
        obj->initIsa(cls);
    }

    if (fastpath(!hasCxxCtor)) {
        return obj;
    }

    construct_flags |= OBJECT_CONSTRUCT_FREE_ONFAILURE;
    return object_cxxConstructFromClass(obj, cls, construct_flags);
}
```

### init 

```objectivec
- (id)init {
    return _objc_rootInit(self);
}

id _objc_rootInit(id obj)
{
    // In practice, it will be hard to rely on this function.
    // Many classes do not properly chain -init calls.
    return obj;
}
```

### dealloc

* rootDealloc

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

* object_dispose

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

* clearDeallocating

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

* clearDeallocating_slow

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

## runtime 加载

### 动态库

iOS 应用会用到很多系统的库，所有 iOS 应用都共用同一套，这些库是动态加载的，称动态库。

优点：
1、防止重复。所有应用共用，防止除服占内存。
2、减少应用包体积。打包时可以省略这一部分库
3、动态更新。

### 加载过程 

1. app 启动，dyld 将应用加载，并完成一些文件的初始化；

2. runtime 向 dyld 中注册回调函数；

3. ImageLoader 将所有 image 加载到内存；

4. dyld 在 image 发生改变时，主动调用回调函数；

5. runtime 接收到 dyld 的回调，执行 map_images、load_images 操作，并调用 +load 方法；

6. 调用 main() 函数

其中 3、4、5 会多次执行，每次加载 image 都会执行

### map_images

map_images 中包含 runtime 大量初始化方法，其核心函数是 _read_images

1. 加载所有类到类的gdb_objc_realized_classes表中。

2. 对所有类做重映射。

3. 将所有SEL都注册到namedSelectors表中。

4. 修复函数指针遗留。

5. 将所有Protocol都添加到protocol_map表中。

6. 对所有Protocol做重映射。

7. 初始化所有非懒加载的类，进行rw、ro等操作。

8. 遍历已标记的懒加载的类，并做初始化操作。

9. 处理所有Category，包括Class和Meta Class。

10. 初始化所有未初始化的类。

### load_images

load_images 的主要内容：

1. 用 prepare_load_methods 函数，将 class load list 和 category load list 准备好

2. 用 call_load_methods 函数，调用上面两个方法表

load 方法的特点：

* 比 main() 函数要更早

* 整个 app 运行期只会执行一次

* 类和类别（category）的 load 方法都会执行，不会覆盖

* 调用顺序 **父类 -> 子类 -> 类别**

* 不是通过 objc_msgSend 调用

### initialize

向对象发送消息时，lookUpImpOrForward 函数会判断当前类是否是

特点

* 第一次调用类所属的方法时，才调用 initialize

* 只会执行一次，也可能整个 app 声明周期都没有调用

* 调用顺序 **父类 -> 子类**，类别中的 initialize 方法会覆盖原方法

* 通过 objc_msgSend 调用


## runtime 消息发送

### objc_msgSend

```objectivec
objc_msgSend(void /* id self, SEL op, ... */)
```

objc_msgSend 的两个隐藏参数：

* self，当前调用对象

* _cmd，当前调用方法的 SEL

当前对象调用任何方法，接收者都是当前对象，即使是 super 调用

当向一个空对象（nil）发送消息，objc_msgSend 会判断接收者为空，直接返回 nil

注意 NSNull 和 nil 的区别：

* NSNull 是继承 NSObject 的对象，只有一个方法 [NSNull null]，向它发送别的消息（方法），会 crash

* nil 是空，向它发消息不会报错


### 发送流程

![](https://cdn.jsdelivr.net/gh/skybrim/AllImages@dev/runtime-2.png)

1. 对象接收到消息
   
2. 按照对象的 isa 指针，找到对象的类
   
3. 先在类的 cache 中查找
   
4. 在类的 method_list 中查找 selecter
   
5. 如果在类中没有找到，沿着继承链，向父类查找，直到 NSObject
   
6. 还未找到，进入**动态方法解析** resolveInstanceMethod: resolveClassMethod:
   
7. 动态方法解析未实现，进入**动态消息转发** 
   
8. forwardingTargetForSelector: 中，将未找到的消息，转发给其他对象
   
9.  forwardingTargetForSelector: 未实现，先调用 methodSignatureForSelector: 在方法内部生成 NSMethodSignature 类型的方法签名对象 forwardInvocation: 中转发消息
    
10. 还未对消息处理，crash

### lookUpImpOrForward 源码

```objectivec
IMP lookUpImpOrForward(Class cls, SEL sel, id inst, 
                       bool initialize, bool cache, bool resolver)
{
    IMP imp = nil;
    bool triedResolver = NO;

    runtimeLock.assertUnlocked();

    // 如果cache是YES，则从缓存中查找IMP。如果是从cache3函数进来，则不会执行cache_getImp()函数
    if (cache) {
        // 通过cache_getImp函数查找IMP，查找到则返回IMP并结束调用
        imp = cache_getImp(cls, sel);
        if (imp) return imp;
    }

    // runtimeLock is held during isRealized and isInitialized checking
    // to prevent races against concurrent realization.

    // runtimeLock is held during method search to make
    // method-lookup + cache-fill atomic with respect to method addition.
    // Otherwise, a category could be added but ignored indefinitely because
    // the cache was re-filled with the old value after the cache flush on
    // behalf of the category.

    runtimeLock.read();

    // 判断类是否已经被创建，如果没有被创建，则将类实例化 (class 的创建时机：第一次接收消息)
    if (!cls->isRealized()) {
        // Drop the read-lock and acquire the write-lock.
        // realizeClass() checks isRealized() again to prevent
        // a race while the lock is down.
        runtimeLock.unlockRead();
        runtimeLock.write();
        
        // 对类进行实例化操作
        realizeClass(cls);

        runtimeLock.unlockWrite();
        runtimeLock.read();
    }

    // 第一次调用当前类的话，执行 initialize 的代码
    if (initialize  &&  !cls->isInitialized()) {
        runtimeLock.unlockRead();
        // 对类进行初始化，并开辟内存空间
        _class_initialize (_class_getNonMetaClass(cls, inst));
        runtimeLock.read();
        // If sel == initialize, _class_initialize will send +initialize and 
        // then the messenger will send +initialize again after this 
        // procedure finishes. Of course, if this is not being called 
        // from the messenger then it won't happen. 2778172
    }

    
 retry:    
    runtimeLock.assertReading();

    // 尝试获取这个类的缓存
    imp = cache_getImp(cls, sel);
    if (imp) goto done;

    {
        // 如果没有从cache中查找到，则从方法列表中获取Method
        Method meth = getMethodNoSuper_nolock(cls, sel);
        if (meth) {
            // 如果获取到对应的Method，则加入缓存并从Method获取IMP
            log_and_fill_cache(cls, meth->imp, sel, inst, cls);
            imp = meth->imp;
            goto done;
        }
    }

    // Try superclass caches and method lists.
    {
        unsigned attempts = unreasonableClassCount();
        // 循环获取这个类的 缓存IMP 或 方法列表的IMP
        for (Class curClass = cls->superclass;
             curClass != nil;
             curClass = curClass->superclass)
        {
            // Halt if there is a cycle in the superclass chain.
            if (--attempts == 0) {
                _objc_fatal("Memory corruption in class list.");
            }
            
            // Superclass cache.
            // 获取父类 缓存的IMP
            imp = cache_getImp(curClass, sel);
            if (imp) {
                if (imp != (IMP)_objc_msgForward_impcache) {
                    // Found the method in a superclass. Cache it in this class.
                    // 如果发现父类的方法，并且不再缓存中，在下面的函数中缓存方法
                    log_and_fill_cache(cls, imp, sel, inst, curClass);
                    goto done;
                }
                else {
                    // Found a forward:: entry in a superclass.
                    // Stop searching, but don't cache yet; call method 
                    // resolver for this class first.
                    break;
                }
            }
            
            // Superclass method list.
            // 在父类的方法列表中，获取method_t对象。如果找到则缓存查找到的IMP
            Method meth = getMethodNoSuper_nolock(curClass, sel);
            if (meth) {
                log_and_fill_cache(cls, meth->imp, sel, inst, curClass);
                imp = meth->imp;
                goto done;
            }
        }
    }

    // No implementation found. Try method resolver once.

    // 如果没有找到，则尝试动态方法解析
    if (resolver  &&  !triedResolver) {
        runtimeLock.unlockRead();
        _class_resolveMethod(cls, sel, inst);
        runtimeLock.read();
        // Don't cache the result; we don't hold the lock so it may have 
        // changed already. Re-do the search from scratch instead.
        triedResolver = YES;
        goto retry;
    }

    // No implementation found, and method resolver didn't help. 
    // Use forwarding.

    // 如果没有IMP被发现，并且动态方法解析也没有处理，则进入消息转发阶段
    imp = (IMP)_objc_msgForward_impcache;
    cache_fill(cls, sel, imp, inst);

 done:
    runtimeLock.unlockRead();

    return imp;
}
```

### 消息转发的方法的区别

forwardingTargetForSelector:， 仅支持一个对象的返回，也就是说消息只能被转发给一个对象、无法处理消息的内容，比如参数和返回值。

forwardInvocation:， 可以将消息同时转发给任意多个对象

```objectivec
- (id)forwardingTargetForSelector:(SEL)aSelector {
    NSString *selectorName = NSStringFromSelector(aSelector);
    if ([selectorName isEqualToString:@"selector"]) {
        return object;
    }
    return [super forwardingTargetForSelector:aSelector];
}

- (void)forwardInvocation:(NSInvocation *)anInvocation {
    if ([object respondsToSelector:[anInvocation selector]]) {
        [anInvocation invokeWithTarget:object];
    } else {
        [super forwardInvocation:anInvocation];
    }
}
```

### 应用

* 阻止因调用未实现的方法而导致崩溃的情况

* 模拟多继承


## method swizzling

### method_exchangeImplementations

核心 API method_exchangeImplementations

### aspects

[源码分析](https://skybrim.top/2019/01/01/iOS/Aspects/)

aspects 合理的规避了 method swizzling 的一些风险

### 注意

* +load 方法中调用

* dispath_once

### 类簇 与 method swizzling

需要找到真实的类，来进行交换操作


## runtime 应用

### __attribute__

```objectivec
// 被修饰的类不能被其他类继承
__attribute__((objc_subclassing_restricted))

// 子类必须调用被修饰的方法
__attribute__((objc_requires_super))

// main函数执行之前
__attribute__((constructor)) 

// main函数执行之后
__attribute__((destructor)) 

// 允许定义多个同名但不同参数类型的函数
__attribute__((overloadable))

// 在编译时，将Class或Protocol指定为另一个名字
__attribute__((objc_runtime_name("xxxx")))

// 消除 unused 警告
NSObject *object __attribute__((unused)) = [[NSObject alloc] init];
```

### ORM

object relational mapping

* MJExtension

* Mantle


## 面试题

### 1

下面的代码输出什么？

```objectivec
@implementation Son : Father
- (id)init {
    self = [super init];
    if (self) {
        NSLog(@"%@", NSStringFromClass([self class]));
        NSLog(@"%@", NSStringFromClass([super class]));
    }
    return self;
}
@end
```

答案：都输出Son。

第一个NSLog输出Son肯定是不用说的。

第二个输出中，[super class]会被转换为下面代码。

```objectivec
struct objc_super objcSuper = {
    self,
    class_getSuperclass([self class]),
};
id (*sendSuper)(struct objc_super*, SEL) = (void *)objc_msgSendSuper;
sendSuper(&objcSuper, @selector(class));
```

super的调用会被转换为objc_msgSendSuper的调用，并传入一个objc_super类型的结构体。结构体有两个参数，第一个就是接受消息的对象，第二个是[super class]对应的父类。

```objectivec
struct objc_super {
    __unsafe_unretained _Nonnull id receiver;
    __unsafe_unretained _Nonnull Class super_class;
};
```

由此可知，虽然调用的是[super class]，但是接受消息的对象还是self。然后来到父类Father的class方法中，输出self对应的类Son。

### 2

下面代码的结果？

```objectivec
BOOL res1 = [(id)[NSObject class] isKindOfClass:[NSObject class]];
BOOL res2 = [(id)[NSObject class] isMemberOfClass:[NSObject class]];
BOOL res3 = [(id)[Sark class] isKindOfClass:[Sark class]];
BOOL res4 = [(id)[Sark class] isMemberOfClass:[Sark class]];
```

答案：
除了第一个是YES，其他三个都是NO。

在推测结果之前，首先要明白两个问题。isKindOfClass和isMemberOfClass的区别是什么？
isKindOfClass:class，调用该方法的对象所属的类，继承者链中包含传入的class则返回YES。
isMemberOfClass:class，调用改方法的对象所属的类，必须是传入的class则返回YES。

```objectivec
+ (BOOL)isMemberOfClass:(Class)cls {
    return object_getClass((id)self) == cls;
}

- (BOOL)isMemberOfClass:(Class)cls {
    return [self class] == cls;
}

+ (BOOL)isKindOfClass:(Class)cls {
    for (Class tcls = object_getClass((id)self); tcls; tcls = tcls->superclass) {
        if (tcls == cls) return YES;
    }
    return NO;
}

- (BOOL)isKindOfClass:(Class)cls {
    for (Class tcls = [self class]; tcls; tcls = tcls->superclass) {
        if (tcls == cls) return YES;
    }
    return NO;
}
```

平时开发过程中只会接触到对象方法的isKindOfClass和isMemberOfClass，但是在NSObject类中还隐式的实现了类方法版本。不只这两个方法，其他NSObject中的对象方法，都有其对应的类方法版本。因为在OC中，类和元类也都是对象。这四个调用由于都是类对象发起调用的，所以最终执行的都是类方法版本。

### 3

下面代码会？Compile Error / Runtime Crash / NSLog…?

```objectivec
@interface NSObject (Sark)
+ (void)foo;
@end

@implementation NSObject (Sark)
- (void)foo {
    NSLog(@"IMP: -[NSObject (Sark) foo]");
}
@end

// 测试代码
[NSObject foo];
[[NSObject new] performSelector:@selector(foo)];
```

答案：
全都正常输出，编译和运行都没有问题。

这道题和上一道题很相似，第二个调用肯定没有问题，第一个调用后会从元类中查找方法，然而方法并不在元类中，所以找元类的superclass。方法定义在是NSObject的Category，由于NSObject的对象模型比较特殊，元类的superclass是类对象，所以从类对象中找到了方法并调用。

### 4

下面的代码会？Compile Error / Runtime Crash / NSLog…?

```objectivec
@interface Sark : NSObject
@property (nonatomic, copy) NSString *name;
@end

@implementation Sark
- (void)speak {
    NSLog(@"my name's %@", self.name);
}
@end

// 测试代码
@implementation ViewController
- (void)viewDidLoad {
    [super viewDidLoad];
    id cls = [Sark class];
    void *obj = &cls;
    [(__bridge id)obj speak];
}
@end
```

答案：
正常执行，不会导致Crash。

执行[Sark class]后获取到类对象，然后通过obj指针指向获取到的类对象首地址，这就构成了对象的基本结构，可以进行正常调用。

### 5

为什么MRC下没有weak？

其实MRC下并不是没有weak，在MRC环境下也可以通过Runtime源码调用weak源码的。weak源码定义在Private Headers私有文件夹下，需要引入#import "objc-internal.h"文件。


以以下ARC的源码为例，定义了一个TestObject类型的对象，并用一个weak指针指向已创建对象。

```objectivec
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        TestObject *object = [[TestObject alloc] init];
        __weak TestObject *newObject = object;
    }
    return 0;
}
```

这段代码会被编译器转移为下面代码，这段代码中的两个函数就是weak的实现函数，在MRC下也可以调用这两个函数。

```objectivec
objc_initWeak(&newObject, object);
objc_destroyWeak(&newObject);
```

### 6

相同的一个类，创建不同的对象，怎样实现指定的某个对象在dealloc时打印一段文字？

这个问题最简单的方法就是在类的.h文件里，定义一个标记属性，如果属性被赋值为YES，则在dealloc中打印文字。但是，这种实现方式显然不是面试官想要的，会被直接pass~

可以参考KVO的实现方案，在运行时动态创建一个类，这个类是对象的子类，将新创建类的dealloc实现指向自定义的IMP，并在IMP中打印一段文字。将对象的isa设置为新创建的类，当执行dealloc方法时就会执行isa所指向的新类。


## 参考

[<简书 — 刘小壮> https://www.jianshu.com/p/ce97c66027cd](https://www.jianshu.com/p/ce97c66027cd) 

