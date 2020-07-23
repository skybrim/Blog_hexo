---
title: Objective-C 引用计数
comments: true
date: 2019-10-19 11:08:50
tags: iOS
---

Objective-C 引用计数
<!--more-->

## 存储引用计数

* TaggedPointer

    直接将其指针值作为引用计数返回

* 64位 && Objective-C 2.0

    isa 指针的一部分会用来存储引用计数

    如果 isa 的引用计数溢出，使用一张 hash 表来存储


### TaggedPointer

判断当前对象是否在使用 TaggedPointer 是看标志位是否为 1 ：

```objectivec
#if SUPPORT_MSB_TAGGED_POINTERS
#   define TAG_MASK (1ULL<<63)
#else
#   define TAG_MASK 1

inline bool 
objc_object::isTaggedPointer()  // objc_object * 就是 id
{
#if SUPPORT_TAGGED_POINTERS
    return ((uintptr_t)this & TAG_MASK);
#else
    return false;
#endif
}
```

### isa 指针

优化的 isa 指针并不是就一定会存储引用计数，毕竟用 19bit （iOS 系统）保存引用计数不一定够。

需要注意的是这 19 位保存的是 **引用计数的值减一**。

has_sidetable_rc 的值如果为 1，那么引用计数会存储在一个叫 SideTable 的类的属性中

看源码

```objectivec
// Define SUPPORT_NONPOINTER_ISA=1 to enable extra data in the isa field.
#if !__LP64__  ||  TARGET_OS_WIN32  ||  TARGET_IPHONE_SIMULATOR  ||  __x86_64__
#   define SUPPORT_NONPOINTER_ISA 0 // iOS 中 arm64 支持优化后的指针
#else
#   define SUPPORT_NONPOINTER_ISA 1
#endif

union isa_t 
{
    isa_t() { }
    isa_t(uintptr_t value) : bits(value) { }

    Class cls;
    uintptr_t bits;

#if SUPPORT_NONPOINTER_ISA
# if __arm64__
#   define ISA_MASK        0x00000001fffffff8ULL
#   define ISA_MAGIC_MASK  0x000003fe00000001ULL
#   define ISA_MAGIC_VALUE 0x000001a400000001ULL
    struct {
        uintptr_t nonpointer        : 1;  // 0 表示普通的 isa 指针；1 表示优化后的 isa 指针，存储引用计数
        uintptr_t has_assoc         : 1;  // 表示该对象是否包含 associated object，如果没有，则析构时会更快
        uintptr_t has_cxx_dtor      : 1;  // 表示该对象是否有 C++ 或 ARC 的析构函数，如果没有，则析构时更快
        uintptr_t shiftcls          : 33; // MACH_VM_MAX_ADDRESS 0x1000000000 类的指针
        uintptr_t magic             : 6;  // 固定值为 0xd2，用于在调试时分辨对象是否未完成初始化。
        uintptr_t weakly_referenced : 1;  // 表示该对象是否有过 weak 对象，如果没有，则析构时更快
        uintptr_t deallocating      : 1;  // 表示该对象是否正在析构
        uintptr_t has_sidetable_rc  : 1;  // 表示该对象的引用计数值是否过大无法存储在 isa 指针  
        uintptr_t extra_rc          : 19; // 存储引用计数值减一后的结果
#       define RC_ONE   (1ULL<<45)
#       define RC_HALF  (1ULL<<18)
    };

# elif __x86_64__
#   define ISA_MASK        0x00007ffffffffff8ULL
#   define ISA_MAGIC_MASK  0x0000000000000001ULL
#   define ISA_MAGIC_VALUE 0x0000000000000001ULL
    struct {
        uintptr_t indexed           : 1;
        uintptr_t has_assoc         : 1;
        uintptr_t has_cxx_dtor      : 1;
        uintptr_t shiftcls          : 44; // MACH_VM_MAX_ADDRESS 0x7fffffe00000
        uintptr_t weakly_referenced : 1;
        uintptr_t deallocating      : 1;
        uintptr_t has_sidetable_rc  : 1;
        uintptr_t extra_rc          : 14;
#       define RC_ONE   (1ULL<<50)
#       define RC_HALF  (1ULL<<13)
    };

# else
    // Available bits in isa field are architecture-specific.
#   error unknown architecture
# endif

// SUPPORT_NONPOINTER_ISA
#endif
};
```

### hash 表存储

![SideTables](https://cdn.jsdelivr.net/gh/skybrim/AllImages@dev/weak_point_0.png)

从上往下分析结构

* SideTables

    SideTables 是一个64（iPhone真机的情况下是 8）个元素长度的 hash 表，里面存储了 SideTable

    键是对象的地址，值是对象的 SideTable

    所以，**一个 SideTable 可能对应多个对象**

    SideTables 在系统中是全局唯一的。

    SideTables的类型是是template<typename T> class StripedMap，StripedMap<SideTable> 。
    
    可以简单的理解为一个64 * sizeof(SideTable) 的哈希线性数组

    ```objectivc

    // SideTabls 可以通过全局的静态函数获取，实质是 StripedMap
    static StripedMap<SideTable>& SideTables() {
        return *reinterpret_cast<StripedMap<SideTable>*>(SideTableBuf);
    }

    // ...

    enum { CacheLineSize = 64 };

    // StripedMap 定义
    // void* -> T
    // StripedMap<T> is a map of void* -> T, sized appropriately 
    // for cache-friendly lock striping. 
    // For example, this may be used as StripedMap<spinlock_t>
    // or as StripedMap<SomeStruct> where SomeStruct stores a spin lock.

    template<typename T>
    class StripedMap {
    #if TARGET_OS_IPHONE && !TARGET_OS_SIMULATOR
        enum { StripeCount = 8 }; // 真机 8
    #else
        enum { StripeCount = 64 };
    #endif

        // 封装 T 
        struct PaddedT {
            T value alignas(CacheLineSize);
        };

        // PaddedT 类型的数组，长度是 StripeCount
        PaddedT array[StripeCount];

        // hash 定位算法
        // 以 void* 作为 key，返回 void* 对应的 SideTable 的位置
        static unsigned int indexForPointer(const void *p) {
            uintptr_t addr = reinterpret_cast<uintptr_t>(p);
            return ((addr >> 4) ^ (addr >> 9)) % StripeCount; // 对 StripeCount 取余，防止越界
        }

    public:
        // 对外的存取数据的方法
        T& operator[] (const void *p) { 
            return array[indexForPointer(p)].value; 
        }
        const T& operator[] (const void *p) const { 
            return const_cast<StripedMap<T>>(this)[p]; 
        }

        // 锁的相关操作，核心都是直接操作 array[i].value
        // Shortcuts for StripedMaps of locks.
        void lockAll() {
            for (unsigned int i = 0; i < StripeCount; i++) {
                array[i].value.lock();
            }
        }

        void unlockAll() {
            for (unsigned int i = 0; i < StripeCount; i++) {
                array[i].value.unlock();
            }
        }

        void forceResetAll() {
            for (unsigned int i = 0; i < StripeCount; i++) {
                array[i].value.forceReset();
            }
        }

        void defineLockOrder() {
            for (unsigned int i = 1; i < StripeCount; i++) {
                lockdebug_lock_precedes_lock(&array[i-1].value, &array[i].value);
            }
        }

        void precedeLock(const void *newlock) {
            // assumes defineLockOrder is also called
            lockdebug_lock_precedes_lock(&array[StripeCount-1].value, newlock);
        }

        void succeedLock(const void *oldlock) {
            // assumes defineLockOrder is also called
            lockdebug_lock_precedes_lock(oldlock, &array[0].value);
        }

        const void *getLock(int i) {
            if (i < StripeCount) return &array[i].value;
            else return nil;
        }
        
    #if DEBUG
        StripedMap() {
            // Verify alignment expectations.
            uintptr_t base = (uintptr_t)&array[0].value;
            uintptr_t delta = (uintptr_t)&array[1].value - base;
            ASSERT(delta % CacheLineSize == 0);
            ASSERT(base % CacheLineSize == 0);
        }
    #else
        constexpr StripedMap() {}
    #endif
    };   
    ```

* SideTable

从上面的 SideTables 的结构，可以得出一个结论：

**一个 SideTable 可以对应多个对象**

RefcountMap 是一个 hash 表，key 是对象的地址，值是引用计数减一


    ```objectivec
    struct SideTable {
        spinlock_t slock;        // 保证原子操作的自旋锁
        RefcountMap refcnts;     // 保存引用计数的 hash 表，也就是 DenseMap
        weak_table_t weak_table; // 保存弱引用的 hash 表

        // 构造函数
        SideTable() {
            memset(&weak_table, 0, sizeof(weak_table));
        }

        // 析构函数 （实际上不能被析构）
        ~SideTable() {
            _objc_fatal("Do not delete SideTable.");
        }

        // 一些锁的操作
        void lock() { slock.lock(); }
        void unlock() { slock.unlock(); }
        void forceReset() { slock.forceReset(); }

        // Address-ordered lock discipline for a pair of side tables.

        template<HaveOld, HaveNew>
        static void lockTwo(SideTable *lock1, SideTable *lock2);
        template<HaveOld, HaveNew>
        static void unlockTwo(SideTable *lock1, SideTable *lock2);
    }
    ```

* RefcountMap

    ```objectivec
    struct RefcountMapValuePurgeable {
        static inline bool isPurgeable(size_t x) {
            return x == 0;
        }
    };

    typedef objc::DenseMap<DisguisedPtr<objc_object>,size_t,RefcountMapValuePurgeable> RefcountMap;
    ```

* DenseMap

    key：DisguisedPtr<objc_object>，是对 objc_object * 指针及其一些操作进行的封装

    value：__darwin_size_t，等价于 unsigned long，内容也是 **引用计数减一**

* weak_table_t

    关于 weak_table_t 的详细分析，见 [Objective-C 弱引用](https://skybrim.top/2019/10/21/iOS/weak-point/)，这里只展示源码

    <details>
    <summary>weak_table_t</summary>

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

    </details>
    

* weak_entry_t

    关于 weak_entry_t 的详细分析，见 [Objective-C 弱引用](https://skybrim.top/2019/10/21/iOS/weak-point/)，这里展示下源码

    <details>
    <summary>weak_entry_t</summary>

    ```objectivec
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
    
    </details>    

## 获取引用计数

### MRC

```objectivec
- (NSUInteger)retainCount {
    return ((id)self)->rootRetainCount();
}
```

### ARC

Core Foundation 库的 CFGetRetainCount()

Runtime 的 _objc_rootRetainCount(id obj)

## 修改引用计数

### retain

引用计数 +1

_objc_rootRetain(id obj)

### release

引用计数 -1

_objc_rootRelease(id obj)


### alloc, new, copy, mutableCopy

引用计数 +1

### autorelease

[autorelease](https://skybrim.top/2017/06/01/iOS/autorelease/)


## 引用&参考

[Objective-C 引用计数原理 by:杨潇玉](http://yulingtianxia.com/blog/2015/12/06/The-Principle-of-Refenrence-Counting/)
[Objective-C runtime机制(5)](https://blog.csdn.net/u013378438/article/details/80733391)
[Objective-C runtime机制(7)](https://blog.csdn.net/u013378438/article/details/82790332) 