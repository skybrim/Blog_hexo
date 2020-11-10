---
title: apply-runtime
comments: true
date: 2020-11-10 10:24:10
tags: iOS
---

runtime 在实际开发中的一些使用
<!--more-->

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
