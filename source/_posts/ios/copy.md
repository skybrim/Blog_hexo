---
title: Copy
date: 2019/1/1
tags: iOS
comments: true
---

Copy 在 Objective-C 中的使用，一些个人理解。

<!--more-->

## Object Copying

OC 中，一个**对象**，如果遵循 NSCopying 协议，且实现了 ```copyWihtZone:``` 方法，就可以被拷贝。

如果**类**具有不可变和可变两种变体，通过遵循 NSMutableCopying 协议，并实现 ``` mutableCopyWithZone:``` 来确保**可变对象**的拷贝也是可变的

Copy 分为 shadow copy 和 deep copy，这两种 copy 的区别，主要在堆指针的处理：

1. shadow copy 仅复制对象的引用指针
2. deep copy 复制新的对象

![](https://cdn.jsdelivr.net/gh/skybrim/AllImages@dev/copy_01.png)

**注意**：

对象的 copy，不能仅仅看内存地址来分辨是 shadow 还是 deep。

比如 NSString，对其使用 copy，返回地址与原对象相同，但是两个对象是完全独立的，相互修改不影响

## Object Copying Memory Management

对象复制，会返回一个引用计数为 1 的新对象。

这个复制的对象的内存管理，谁引用谁负责释放。

## copy 修饰属性

copy 作为 内存管理语义 ，在 @property 中声明，其目的是：复制外部的传过来的值并隔断之后外界值修改对属性的影响。

对应的， strong(retain) 修饰词，只是将外部值或者指针赋值给属性（shadow copy），如果是对象指针，之后外部值修改会影响属性。

```objc
//copy 修饰
@property (nonatomic, copy) NSString *copyStr;
- (void)setCopyStr:(NSString *)copyStr {
    if (_copyStr != copyStr) {
        [_copyStr release];
        _copyStr = [copyStr copy];
    }
}

// strong(retain) 修饰
@property (nonatomic, strong) NSString *strongStr;
- (void)setStrongStr:(NSString *)strongStr {
    if (_strongStr != strongStr) {
        [_strongStr release];
        [strongStr retain];
        _strongStr = strongStr;
    }
}
```

观察赋值方法，引申出一个小知识点：

使用```self.foo = foo;```语法赋值，调用了 setter 方法；使用```_foo = foo;```则只是改变指针指向。

如果只是简单的变更指针指向，可能会导致可变对象和不可变对象混淆。

因此，更推荐使用```self.```语法进行赋值

* NSString

在 OC 中，因为父类指针可以指向子类对象，使用 copy 的目的是为了让本对象的属性不受外界影响。

使用 copy 修饰 NSString，无论传入是一个可变对象还是不可对象，对象持有的就是一个不可变的副本.  

注意：无论原来的 string 类型是否可变，copy 后的类型均为不可变类型。

* collection

NSArray、NSDictionary、NSSet 属于 collection 集合类对象。

这些集合类的属性，使用 copy 修饰，只能保证容器本身的不变，如果容器内的引用对象，发生了变化，这些容器属性中的对象也会发生变化。

* block

block 使用 copy 是从 MRC 遗留下来的“传统”。

在 MRC 中,方法内部的 block 是在栈区的,使用 copy 可以把它放到堆区。

在 ARC 中写不写都行：对于 block 使用 copy 还是 strong 效果是一样的，但写上 copy 可以提示：编译器自动对 block 进行了 copy 操作。

如果block块没有访问处于栈区的变量（比如局部变量），也没有访问堆区的变量（比如我们alloc创建的对象），那就存在代码区(低地址)，即使访问了全局变量，也依然存在代码区。

如果访问了栈区或者堆区的变量，那就会被存在堆区（实际存在栈区，ARC下会自动拷贝到堆区）。

## copy mutableCopy 方法

主动调用 ```copy``` 方法，其**目的**是为了 copy 一个新对象。

此时需要根据对象的类实现的 ```copyWithZone:``` 方法来判断是 shadow 还是 deep。

* 非集合类对象

对 immutable 对象进行 copy 操作， 执行结果类似 retain 操作，但操作的目的是为了 copy 对象。

对 immutable 对象进行 mutableCopy 操作时内容复制。

对 mutable 对象进行 copy 和 mutableCopy 都是内容复制。

```objectivec
[immutableObject copy] // 类似 retain
[immutableObject mutableCopy] //深复制
[mutableObject copy] //深复制
[mutableObject mutableCopy] //深复制
```

* 集合类对象

和上面非集合类对象十分类似。

但是无论是 copy 还是 mutableCopy，集合中的元素，都是仅仅执行了一次 retain 操作，依然是原对象

```objectivec
[immutableCollection copy]
[immutableCollection mutableCopy] //深复制
[mutableCollection copy] //深复制
[mutableCollection mutableCopy] //深复制
```

## copy collection 

当我们对集合类型的对象，使用 copy 或者 mutableCopy，都是对集合对象本身的拷贝，集合中的元素，都仅仅是一个 retain 操作。

![array copy](https://cdn.jsdelivr.net/gh/skybrim/AllImages@dev/CopyingCollections_2x.png)  

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

**首先，需要集合对象中的元素，实现 NSCopying 协议**

使用 ```initWithArray:copyItems:```

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

**注意**：```initWithArray:copyItems:``` 这个方法，只能对一元数组进行 deep copy。

apple 推荐使用 NSKeyedArchiver 实现 true deep copy。**需要集合对象中的元素，实现 NSCoding 协议**

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

## 参考文档

[Object Copying](https://developer.apple.com/library/archive/documentation/General/Conceptual/DevPedia-CocoaCore/ObjectCopying.html)

[Copying Collections](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/Collections/Articles/Copying.html#//apple_ref/doc/uid/TP40010162-SW3)  
