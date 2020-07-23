---
title: autorelease
comments: true
date: 2017-06-01 13:40:30
tags: iOS
---

[黑幕背后的Autorelease](https://blog.sunnyxx.com/2014/10/15/behind-autorelease/)
<!--more-->

## autorelease 对象释放时机

在没有手加Autorelease Pool的情况下，Autorelease对象是在当前的runloop迭代结束时释放的，而它能够释放的原因是系统在每个runloop迭代中都加入了自动释放池Push和Pop

## autorelease 原理

### AutoreleasePoolPage

ARC下，我们使用@autoreleasepool{}来使用一个AutoreleasePool，随后编译器将其改写成下面的样子：

```objectivec
void *context = objc_autoreleasePoolPush();
// {}中的代码
objc_autoreleasePoolPop(context);
```
AutoreleasePoolPage是一个C++实现的类

```objectivec
#define PROTECT_AUTORELEASEPOOL 0

class AutoreleasePoolPage 
{

#define POOL_SENTINEL 0
    static pthread_key_t const key = AUTORELEASE_POOL_KEY;
    static uint8_t const SCRIBBLE = 0xA3;  // 0xA3A3A3A3 after releasing
    static size_t const SIZE = 

    // AutoreleasePoolPage 每个对象会开辟4096字节内存
#if PROTECT_AUTORELEASEPOOL
        4096;  // must be multiple of vm page size
#else
        4096;  // size and alignment, power of 2
#endif
    static size_t const COUNT = SIZE / sizeof(id);

    magic_t const magic;
    id *next;                               // 指向栈顶最新add进来的autorelease对象的下一个位置
    pthread_t const thread;                 // 当前线程
    AutoreleasePoolPage * const parent;     // 双向链表，pre 指针
    AutoreleasePoolPage *child;             // 双向链表，next 指针
    uint32_t const depth;
    uint32_t hiwat;

    // 其他相关方法
    // ...
}
```

![](https://cdn.jsdelivr.net/gh/skybrim/AllImages@dev/autorelease_0.jpg)

向一个对象发送- autorelease消息，就是将这个对象加入到当前AutoreleasePoolPage的栈顶next指针指向的位置

### 释放

* objc_autoreleasePoolPush()

每当进行一次 objc_autoreleasePoolPush 调用时，runtime向当前的AutoreleasePoolPage中add进一个哨兵对象，值为0（也就是个nil）

objc_autoreleasePoolPush的返回值正是这个哨兵对象的地址

![](https://cdn.jsdelivr.net/gh/skybrim/AllImages@dev/autorelease_1.jpg)

* objc_autoreleasePoolPop(哨兵对象)

1. 根据传入的哨兵对象地址找到哨兵对象所处的page

2. 在当前page中，将晚于哨兵对象插入的所有 autorelease 对象都发送一次 - release 消息，并向回移动next指针到正确位置

3. 补充2：从最新加入的对象一直向前清理，可以向前跨越若干个page，直到哨兵所在的page

![](https://cdn.jsdelivr.net/gh/skybrim/AllImages@dev/autorelease_2.jpg)

### 嵌套的 AutoreleasePool

pop的时候总会释放到上次push的位置为止，多层的pool就是多个哨兵对象而已，就像剥洋葱一样，每次一层，互不影响