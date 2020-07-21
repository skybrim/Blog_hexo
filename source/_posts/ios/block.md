---
title: block
comments: true
date: 2019-03-04 15:17:38
tags: iOS
---

Blocks
<!--more-->

## 定义

Blocks，带有自动变量（局部变量）的匿名函数。

C 语言中的变量：

* 自动变量（局部变量）
* 函数的参数
* 静态变量（静态局部变量）
* 静态全局变量
* 全局变量

其中，静态变量（静态局部变量）、静态全局变量、全局变量 可以在函数的多次调用之间传递值。

## 语法

Objective-C 中 blocks 的语法

```objc
// ^ 返回值类型 参数列表 表达式
^void (int a) { NSLog("%d", a); }

// 返回值类型省略，表达式有 return 使用返回值类型，没有 return 使用 void，多个 return 必须类型相同
^(int count) { return count + 1; }

// 不使用参数，参数列表也可以省略
^{ NSLog("abc"); }

// 声明 block 类型
typedef int (^blk_t)(int);
```

## Block 实质

clang(LLVM编译器)，将 block 语法的代码，转变为 C++ 的源代码，观察 block 实质。

```bash
clang -rewrite-objc main.m
```

### 没有截获自动变量的 block

```objc
// block 
#include <stdio.h>
int main() { 
    void (^blk)(void) = ^{ printf("hello\n"); };
    blk();
    return 0;
}
```

```c
// 编译器转为 C++ 

// __main_block_impl_0 内部 impl 的类型
struct __block_impl {
  void *isa;
  int Flags;
  int Reserved;
  void *FuncPtr;
};

// __main_block_impl_0 内部 Desc 的类型
static struct __main_block_desc_0 {
  size_t reserved;
  size_t Block_size;
} __main_block_desc_0_DATA = { 0, sizeof(struct __main_block_impl_0)};

// 表达式转换的 C函数 __main_block_func_0 的参数 cself 的类型
struct __main_block_impl_0 {

  struct __block_impl impl;
  struct __main_block_desc_0* Desc;

  // __main_block_impl_0 的构造函数
  __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, int flags=0) {
    impl.isa = &_NSConcreteStackBlock; /* isa 指针 */
    impl.Flags = flags;
    impl.FuncPtr = fp; /* 函数的指针 */
    Desc = desc;
  }
};

// 表达式 ^{printf("hello");}; 转变为 __main_block_func_0
static void __main_block_func_0(struct __main_block_impl_0 *__cself) {
    printf("hello\n"); 
}

int main() 
{   
    
    // struct __main_block_impl_0 tmp = __main_block_impl_0(__main_block_func_0, /* Block语法中表达式转换的C 函数的指针 */ 
                                                            &__main_block_desc_0_DATA);
    // struct __main_blcok_impl_0 *blk = &tmp; /* __main_blcok_impl_0 的指针赋值给 blk */

    void (*blk)(void) = ((void (*)())&__main_block_impl_0((void *)__main_block_func_0, &__main_block_desc_0_DATA));
    ((void (*)(__block_impl *))((__block_impl *)blk)->FuncPtr)((__block_impl *)blk);
    return 0;
}
```

### 截获自动变量的 block

```objc
// 源码
#include <stdio.h>
int main() {
    int dmy = 256;
    int val = 10;
    const char *fmt = "val = %d\n";
    void (^blk)(void) = ^{ printf(fmt, val); };
    blk();
    return 0;
}
```

```c
// __main_block_impl_0 内部 impl 的类型
struct __block_impl {
  void *isa;
  int Flags;
  int Reserved;
  void *FuncPtr;
};

// __main_block_impl_0 内部 Desc 的类型
static struct __main_block_desc_0 {
  size_t reserved;
  size_t Block_size;
} __main_block_desc_0_DATA = { 0, sizeof(struct __main_block_impl_0)};

// 表达式转换的 C函数 __main_block_func_0 的参数 cself 的类型
struct __main_block_impl_0 {

  struct __block_impl impl;
  struct __main_block_desc_0* Desc;
  // 截获的自动变量
  const char *fmt;
  int val;
  // 构造函数
  __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, const char *_fmt, int _val, int flags=0) : fmt(_fmt), val(_val) {
    impl.isa = &_NSConcreteStackBlock;
    impl.Flags = flags;
    impl.FuncPtr = fp;
    Desc = desc;
  }
};
// 表达式 ^{ printf(fmt, val); }; 转变为 __main_block_func_0
// __cself->val，此 val 是在创建 blcok 时保存到 block 中的自动变量的值
// 并不能拿到自动变量，所以我们只能使用，而不能赋值给自动变量
static void __main_block_func_0(struct __main_block_impl_0 *__cself) {
  const char *fmt = __cself->fmt; // bound by copy
  int val = __cself->val; // bound by copy
 printf(fmt, val); }


int main() {
    int dmy = 256;
    int val = 10;
    const char *fmt = "val = %d\n";

    // 构造 block 的时候，将 block 使用的自动变量的值，保存到 block 的结构体实例（即 block 自身）中
    void (*blk)(void) = ((void (*)())&__main_block_impl_0((void *)__main_block_func_0, &__main_block_desc_0_DATA, fmt, val));
    ((void (*)(__block_impl *))((__block_impl *)blk)->FuncPtr)((__block_impl *)blk);
    return 0;
}
```

### 静态变量 & 静态全局变量 & 全局变量

```objc
#include <stdio.h>
int global_val = 1;
static int static_global_val = 2;
int main() {
    static int static_val = 3;
    void (^blk)(void) = ^{
        global_val *= 1;
        static_global_val *= 2;
        static_val *= 3;
    };
    blk();
    return 0;
}
```

```c
// 全局变量
int global_val = 1;
static int static_global_val = 2;

struct __block_impl {
  void *isa;
  int Flags;
  int Reserved;
  void *FuncPtr;
};

static struct __main_block_desc_0 {
  size_t reserved;
  size_t Block_size;
} __main_block_desc_0_DATA = { 0, sizeof(struct __main_block_impl_0)};

struct __main_block_impl_0 {
  struct __block_impl impl;
  struct __main_block_desc_0* Desc;
  // 静态变量的指针
  int *static_val;
  __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, int *_static_val, int flags=0) : static_val(_static_val) {
    impl.isa = &_NSConcreteStackBlock;
    impl.Flags = flags;
    impl.FuncPtr = fp;
    Desc = desc;
  }
};
static void __main_block_func_0(struct __main_block_impl_0 *__cself) {
  int *static_val = __cself->static_val; // bound by copy
        // 直接操作 全局变量 和 静态全局变量
        global_val *= 1;
        static_global_val *= 2;
        // 通过指针，操作静态变量
        (*static_val) *= 3;
    }

int main() {
    static int static_val = 3;
    // 参数 &static_val，构建 block 时，将静态变量的指针赋值到 block 中
    void (*blk)(void) = ((void (*)())&__main_block_impl_0((void *)__main_block_func_0, &__main_block_desc_0_DATA, &static_val));
    ((void (*)(__block_impl *))((__block_impl *)blk)->FuncPtr)((__block_impl *)blk);
    return 0;
}
```

补充：

为什么操作静态变量可以使用指针，而操作自动变量没有这么做？

因为 block 调用时，可能已经超过自动变量的作用域，自动变量的指针，超出其自身的作用域后就会被废弃，所以操作指针的方式只适用于静态变量。

### __block

```objc
#include <stdio.h>
int main() {
    __block int val = 10;
    void (^blk)(void) = ^{
        val = 1;
    };
    blk();
    return 0;
}
```

```c
struct __block_impl {
  void *isa;
  int Flags;
  int Reserved;
  void *FuncPtr;
};

static struct __main_block_desc_0 {
  size_t reserved;
  size_t Block_size;
  void (*copy)(struct __main_block_impl_0*, struct __main_block_impl_0*);
  void (*dispose)(struct __main_block_impl_0*);
} __main_block_desc_0_DATA = { 0, sizeof(struct __main_block_impl_0), __main_block_copy_0, __main_block_dispose_0};

static void __main_block_copy_0(struct __main_block_impl_0*dst, struct __main_block_impl_0*src) {_Block_object_assign((void*)&dst->val, (void*)src->val, 8/*BLOCK_FIELD_IS_BYREF*/);}

static void __main_block_dispose_0(struct __main_block_impl_0*src) {_Block_object_dispose((void*)src->val, 8/*BLOCK_FIELD_IS_BYREF*/);}

// __block 修饰 int，转换为 C++ 的源码
struct __Block_byref_val_0 {
  void *__isa;
__Block_byref_val_0 *__forwarding; // 自身类型的指针
 int __flags;
 int __size;
 int val;
};

struct __main_block_impl_0 {
  struct __block_impl impl;
  struct __main_block_desc_0* Desc;
  __Block_byref_val_0 *val; // by ref
  __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, __Block_byref_val_0 *_val, int flags=0) : val(_val->__forwarding) {
    impl.isa = &_NSConcreteStackBlock;
    impl.Flags = flags;
    impl.FuncPtr = fp;
    Desc = desc;
  }
};

static void __main_block_func_0(struct __main_block_impl_0 *__cself) {
    // __block 对象
  __Block_byref_val_0 *val = __cself->val; // bound by ref
        // 为什么需要一个额外的指针，而不是直接取值？解释在下面。
        (val->__forwarding->val) = 1;
}

int main() {
    // 构造 __block
    __attribute__((__blocks__(byref))) __Block_byref_val_0 val = {(void*)0,(__Block_byref_val_0 *)&val, 0, sizeof(__Block_byref_val_0), 10};
    // 构造 block
    void (*blk)(void) = ((void (*)())&__main_block_impl_0((void *)__main_block_func_0, &__main_block_desc_0_DATA, (__Block_byref_val_0 *)&val, 570425344));
    // block 执行
    ((void (*)(__block_impl *))((__block_impl *)blk)->FuncPtr)((__block_impl *)blk);
    return 0;
}
```

* block 的存储域
  
  block 是 objective-c 的对象。block 的类有三种，_NSConcreteStackBlock、_NSConcreteGlobalBlock、_NSConcreteMallocBlock。

  | 类 | 设置对象的存储域 |
  | ---- | ---- |
  | _NSConcreteStackBlock | 栈 |
  | _NSConcreteGlobalBlock | 程序的数据区域（.data 区）|
  | _NSConcreteMallocBlock | 堆 |

  ![](https://raw.githubusercontent.com/skybrim/AllImages/dev/block_0.png)

* _NSConcreteGlobalBlock
  
   1. block 声明在全局变量的地方
   2. block 表达式中没有使用自动变量，虽然 clang 转换的源码通常是_NSConcreteStackBlock，但实现上不同

  即，block 存储在程序的数据区域中。

* _NSConcreteStackBlock

  除了上述两种 global 情况外，其他 block 为 _NSConcreteStackBlock 对象，存储在栈上。

* _NSConcreteMallocBlock

  何时使用堆上的 _NSConcreteMallocBlock?

  设置在栈上的 block 和 __block 对象，在离开了所属变量的作用域时，就会被废弃。

  Blocks 提供了将 block 和 __block 对象，从栈上赋值到堆上的方法来解决这个问题。

  复制到堆上的 block 将 _NSConcreteMallocBlock 类对象写入 block 的结构体实例的变量 isa

  impl.isa = &_NSConcreteMallocBlock;

  ![](https://raw.githubusercontent.com/skybrim/AllImages/dev/block_1.png)

  当 block 被复制到堆上时，block 使用的所有 __block 变量，也会被复制到堆上，此时 block 持有 __block 变量。

  __block 对象的 __forwarding 指针，则会在复制到堆上后，指向堆上的 __block 结构体实例的地址。

  ![](https://raw.githubusercontent.com/skybrim/AllImages/dev/block_2.png)

* 什么时候栈上的 block 会被复制到堆上呢？

  - 调用 block 的 copy 实例方法；
  - block 作为函数的返回值
  - 将 block 赋值给 __strong 修饰的 id 类型的类或 block 类型成员变量时；
  - 在方法名中含有 usingBlock 的 Cocoa 框架方法，或者 GCD 的 API 中传递 block 时。


## 循环引用

如果在 block 中使用 strong 修饰符的对象的自动变量，那么当 block 从栈复制到堆上时，改对象被 block 持有。

此时可能产生循环引用。

```objc
// block 循环引用实例代码

typedef void(^blk_t)(void);

@interface MyObject: NSObject
{
  blk_t blk_;
}
@end

@implementation MyObject
- (id)init {
  self = [super init];
  blk_ = ^{ NSLog(@"self = %@", self); };
  return self;
}
- (void)execBlock {
  blk_();
}
- (void)dealloc {
  NSLog(@"dealloc");
}
@end

int main() {
  id foo = [[MyObject alloc] init];
  NSLog(@"%@", foo);
  return 0;
}
```

上述代码中，dealloc 方法没有被调用。

1. foo 强引用 blk_
2. blk_ 强引用 foo。

解决方法：

* 使用 weak
  
  ```objc
  id __weak tmp = self;
  blk_ = ^{ NSLog(@"self = %@", tmp); };
  ```

  1. foo 强引用 blk_
  2. blk_ 弱引用 foo。

* 使用 __block

  ```objc
  __block id tmp = self;
  blk_ = ^{
    NSLog(@"self = %@", self);
    tmp = nil; // 置空 tmp
  };
  // ...
  // 必须执行 blk_()
  [foo execBlock];
  ```

  1. foo 强引用 blk_;
  2. blk_ 强引用 __block 修饰的 tmp;
  3. tmp 强引用 foo;
  4. **执行完 blk_(), tmp 对 foo 没有引用**

  简单起见，使用 weak 方法。

