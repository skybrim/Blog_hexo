---
title: iOS 中 Copy 修饰词
date: 2019/1/1
tags: iOS
comments: true
---

Copy 关键词在 Objective-C 中的一些知识点。

<!--more-->

## 修饰属性

copy 作为 内存管理语义 ，在 @property 中声明。

* NSString、NSArray、NSDictionary

在 OC 中，因为父类指针可以指向子类对象，使用 copy 的目的是为了让本对象的属性不受外界影响，使用 copy 无论给我传入是一个可变对象还是不可对象,本身持有的就是一个不可变的副本.  
使用 copy 修饰 NSString、NSArray、NSDictionary，是因为他们有对应的可变类型的子类：NSMutableString、NSMutableArray、NSMutableDictionary。  
注意：无论原来的类型是否可变，copy 后的类型均为不可变类型。

* block

block 使用 copy 是从 MRC 遗留下来的“传统”。  
在 MRC 中,方法内部的 block 是在栈区的,使用 copy 可以把它放到堆区。  
在 ARC 中写不写都行：对于 block 使用 copy 还是 strong 效果是一样的，但写上 copy 也无伤大雅，还能时刻提醒我们：编译器自动对 block 进行了 copy 操作。

>如果block块没有访问处于栈区的变量（比如局部变量），也没有访问堆区的变量（比如我们alloc创建的对象），那就存在代码区(低地址)，即使访问了全局变量，也依然存在代码区。
>如果访问了栈区或者堆区的变量，那就会被存在堆区（实际存在栈区，ARC下会自动拷贝到堆区）。

## copy & mutableCopy

* 非集合类对象

对 immutable 对象进行 copy 操作，是指针复制，mutableCopy 操作时内容复制；  
对 mutable 对象进行 copy 和 mutableCopy 都是内容复制。用代码简单表示如下：

```objectivec
[immutableObject copy] // 浅复制
[immutableObject mutableCopy] //深复制
[mutableObject copy] //深复制
[mutableObject mutableCopy] //深复制
```

* 集合类对象

和上面非集合类对象十分类似。  
但是无论是浅显复制还是深复制，集合中的元素，都是指针复制

```objectivec
[immutableObject copy] // 浅复制
[immutableObject mutableCopy] //深复制
[mutableObject copy] //深复制
[mutableObject mutableCopy] //深复制
```

## shadow copy 和 deep copy

先上文档：
[Copying Collections](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/Collections/Articles/Copying.html#//apple_ref/doc/uid/TP40010162-SW3)  

shadow copy，浅复制，实质就是指针的复制。  
deep copy，深复制，实质是对象复制。  

从集合类型的复制，具体展示一下 shadow copy 与 deep copy
![array copy](https://raw.githubusercontent.com/skybrim/AllImages/dev/CopyingCollections_2x.png)  

当我们对集合类型的对象，使用 copy 或者 mutableCopy，都是对集合对象的浅拷贝或者深拷贝，集合中的元素，都是指针拷贝（浅拷贝）。代码如下：

```objectivec
    UIView *view1 = [UIView new];
    UIView *view2 = [UIView new];
    UIView *view3 = [UIView new];

    NSArray *array = @[view1, view2, view3,];
    NSArray *copyArray = [array copy];
    NSMutableArray *mCopyArray = [array mutableCopy];

    NSLog(@"array %p \n %@", array, array);
    NSLog(@"copy %p \n %@", copyArray, copyArray);
    NSLog(@"mutableCopy %p \n %@", mCopyArray, mCopyArray);

    输出结果：
    array 0x600003da2ca0
    (
        "<ObjectCopyingCoding: 0x6000033feca0>",
        "<ObjectCopyingCoding: 0x6000033fede0>",
        "<ObjectCopyingCoding: 0x6000033feda0>"
    )
    copy 0x600003da2ca0
    (
        "<ObjectCopyingCoding: 0x6000033feca0>",
        "<ObjectCopyingCoding: 0x6000033fede0>",
        "<ObjectCopyingCoding: 0x6000033feda0>"
    )
    mutableCopy 0x600003da2af0
    (
        "<ObjectCopyingCoding: 0x6000033feca0>",
        "<ObjectCopyingCoding: 0x6000033fede0>",
        "<ObjectCopyingCoding: 0x6000033feda0>"
    )
```

如何对集合对象进行 deep copy 呢？

```objectivec
    NSArray *oneLevelDeepCopy = [[NSArray alloc] initWithArray:array copyItems:YES];
    NSLog(@"oneLevelDeepCopy %p \n %@", oneLevelDeepCopy, oneLevelDeepCopy);

    输出结果：
    oneLevelDeepCopy 0x600003d9ef10
    (
        "<ObjectCopyingCoding: 0x6000033b47a0>",
        "<ObjectCopyingCoding: 0x6000033b4780>",
        "<ObjectCopyingCoding: 0x6000033b4760>"
    )
```

**但是**， initWithArray:copyItems: 这个方法，只能对一位数组进行 deep copy。  
apple 推荐使用 NSKeyedArchiver 实现 true deep copy。  
**需要集合对象中的元素，也实现 NSCopying 和 NSCoding 协议**

```objectivec
    NSArray *twoLevelArray = @[@[obj1, obj2, obj3,]];
    NSLog(@"twoLevelArray %p \n %@", twoLevelArray, twoLevelArray);
    NSArray *oneLevelDeepCopy2 = [[NSArray alloc] initWithArray:twoLevelArray copyItems:YES];
    NSLog(@"oneLevelDeepCopy2 %p \n %@", oneLevelDeepCopy2, oneLevelDeepCopy2);

    NSArray* trueDeepCopyArray = [NSKeyedUnarchiver unarchiveObjectWithData:
    [NSKeyedArchiver archivedDataWithRootObject:array]];
    NSLog(@"trueDeepCopyArray %p \n %@", trueDeepCopyArray, trueDeepCopyArray);

    输出结果
    twoLevelArray 0x6000031f07e0
    (
            (
            "<ObjectCopyingCoding: 0x6000033feca0>",
            "<ObjectCopyingCoding: 0x6000033fede0>",
            "<ObjectCopyingCoding: 0x6000033feda0>"
        )
    )
    oneLevelDeepCopy2 0x6000031e3c20
    (
            (
            "<ObjectCopyingCoding: 0x6000033feca0>",
            "<ObjectCopyingCoding: 0x6000033fede0>",
            "<ObjectCopyingCoding: 0x6000033feda0>"
        )
    )
    trueDeepCopyArray 0x600003d9ae80
    (
        "<ObjectCopyingCoding: 0x6000033e6e00>",
        "<ObjectCopyingCoding: 0x6000033e6e20>",
        "<ObjectCopyingCoding: 0x6000033e4e40>"
    )
```

## 自定义对象使用 copy

如果想让自定义的对象具有拷贝功能，则需实现 NSCopying 协议。  
如果自定义的对象分为可变版本与不可变版本，那么就要同时实现 NSCopying 与 NSMutableCopying 协议。

* 实现步骤

1 需声明该类遵从 NSCopying 协议  
2 实现 NSCopying 协议。

 ```objectivec
 - (id)copyWithZone:(NSZone *)zone;
 ```
