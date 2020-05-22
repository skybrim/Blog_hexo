---
title: iOS runtime
comments: true
date: 2020-05-22 14:02:51
tags: iOS
---

runtime
<!--more-->


## 基本定义

### 对象

![](https://raw.githubusercontent.com/skybrim/AllImages/dev/run_time-1.jpg)

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
    int32_t *offset;
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

##  runtime 结构体分析

### isa_t

```objectivec
union isa_t 
{
    isa_t() { }
    isa_t(uintptr_t value) : bits(value) { }

    Class cls;
    uintptr_t bits;

    # if __arm64__
#   define ISA_MASK        0x0000000ffffffff8ULL
#   define ISA_MAGIC_MASK  0x000003f000000001ULL
#   define ISA_MAGIC_VALUE 0x000001a000000001ULL
    struct {
        uintptr_t nonpointer        : 1;
        uintptr_t has_assoc         : 1;
        uintptr_t has_cxx_dtor      : 1;
        uintptr_t shiftcls          : 33; // MACH_VM_MAX_ADDRESS 0x1000000000
        uintptr_t magic             : 6;
        uintptr_t weakly_referenced : 1;
        uintptr_t deallocating      : 1;
        uintptr_t has_sidetable_rc  : 1;
        uintptr_t extra_rc          : 19;
#       define RC_ONE   (1ULL<<45)
#       define RC_HALF  (1ULL<<18)
    };

# elif __x86_64__
    ...
# else
    ...
# endif
};
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
    return _objc_rootAlloc(self);
}

id _objc_rootAlloc(Class cls)
{
    return callAlloc(cls, false/*checkNil*/, true/*allocWithZone*/);
}

static ALWAYS_INLINE id
callAlloc(Class cls, bool checkNil, bool allocWithZone=false)
{
    if (fastpath(!cls->ISA()->hasCustomAWZ())) {
        // No alloc/allocWithZone implementation. Go straight to the allocator.
        // fixme store hasCustomAWZ in the non-meta class and 
        // add it to canAllocFast's summary
        if (fastpath(cls->canAllocFast())) {
            // No ctors, raw isa, etc. Go straight to the metal.
            bool dtor = cls->hasCxxDtor();
            id obj = (id)calloc(1, cls->bits.fastInstanceSize());
            if (slowpath(!obj)) return callBadAllocHandler(cls);
            obj->initInstanceIsa(cls, dtor);
            return obj;
        }
        else {
            // Has ctor or raw isa or something. Use the slower path.
            id obj = class_createInstance(cls, 0);
            if (slowpath(!obj)) return callBadAllocHandler(cls);
            return obj;
        }
    }
}

id class_createInstance(Class cls, size_t extraBytes)
{
    return _class_createInstanceFromZone(cls, extraBytes, nil);
}

static __attribute__((always_inline)) 
id
_class_createInstanceFromZone(Class cls, size_t extraBytes, void *zone, 
                              bool cxxConstruct = true, 
                              size_t *outAllocatedSize = nil)
{
    if (!cls) return nil;

    assert(cls->isRealized());

    // Read class's info bits all at once for performance
    bool hasCxxCtor = cls->hasCxxCtor();
    bool hasCxxDtor = cls->hasCxxDtor();
    bool fast = cls->canAllocNonpointer();  //获取对象原始大小，编译时决定，对象大小至少 16 字节

    size_t size = cls->instanceSize(extraBytes);
    if (outAllocatedSize) *outAllocatedSize = size;

    id obj;
    if (!zone  &&  fast) {
        obj = (id)calloc(1, size);
        if (!obj) return nil;
        obj->initInstanceIsa(cls, hasCxxDtor);
    } 
    else {
        if (zone) {
            obj = (id)malloc_zone_calloc ((malloc_zone_t *)zone, 1, size);
        } else {
            obj = (id)calloc(1, size);
        }
        if (!obj) return nil;

        // Use raw pointer isa on the assumption that they might be 
        // doing something weird with the zone or RR.
        obj->initIsa(cls);
    }

    if (cxxConstruct && hasCxxCtor) {
        obj = _objc_constructOrFree(obj, cls);
    }

    return obj;
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

runtime 调用 objc_object::rootDealloc，然后 object_dispose()

object_dispose() 通过 objc_destructInstance() 实现

```objectivec
void *objc_destructInstance(id obj) 
{
    if (obj) {
        // Read all of the flags at once for performance.
        // 判断是否有OC或C++的析构函数
        bool cxx = obj->hasCxxDtor();
        // 对象是否有相关联的引用
        bool assoc = obj->hasAssociatedObjects();

        // This order is important.
        // 对当前对象进行析构
        if (cxx) object_cxxDestruct(obj);
        // 移除所有对象的关联，例如把weak指针置nil
        if (assoc) _object_remove_assocations(obj);
        obj->clearDeallocating();
    }

    return obj;
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

### 发送流程

![](https://raw.githubusercontent.com/skybrim/AllImages/dev/runtime-2.png)

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

### aspects 源码

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

