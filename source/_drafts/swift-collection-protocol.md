---
title: swift 集合协议
comments: true
date: 2020-07-07 14:15:41
tags: [iOS, swift]
---

swift 集合协议
<!--more-->

## 标准库中的结合协议架构

![标准库中的结合协议架构](https://raw.githubusercontent.com/skybrim/AllImages/dev/swift-collection-protocol-1.jpg)

* Sequence

    提供了迭代的能力，允许创建一个迭代器

* Collection

    支持多次遍历

    提供通过索引访问元素

    通过 SubSequence 提供了集合切片的能力，切片自身也是一个集合

* MutableCollection

    提供了常数时间内，通过下标更改集合的能力

    不允许向集合中添加和删除元素

* RangeReplaceableCollection

    提供了替换集合中一个连续区间的元素的能力。

    通过扩展，衍生出 append 和 remove 等方法

    Array/String 支持

    Set/Dictionary 不支持

* BidirectionalCollection

    提供了从集合尾部想集合头部遍历的能力


* RandomAccessCollection

    提供了更高效的索引计算能力：计算索引的距离或者移动索引位置都是常数时间操作。

    举例：Array 是随机访问的集合，但是 String 不是。

* LazySequenceProtocol


* LazyCollectionProtocol

    
## Sequence

