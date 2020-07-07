---
title: swift-collection
comments: true
date: 2020-07-07 14:50:24
tags: [iOS, swift]
---

swift 集合类型
<!--more-->

## Array

```swift
// array 方法
var array = [Int]()
// 指定数组容量大小
array.reserveCapacity(20)
// 数组是否为空
array.isEmpty

// 迭代
for x in array {}

// 迭代除了第一个元素以外的数组
for x in array.dropFirst() {}

// 迭代除了最后 5 个元素以外的数组
for x in array.dropLast(5) {}

// 迭代索引和元素
for (index, element) in array.enumerated() {}

// 根据匹配逻辑找指定内容
if let index = array.firstIndex{ someMatchingLogic($0) } {}
if let index = array.lastIndex { someMatchingLogic($0) } {}
if let element = array.first { someMatchingLogic($0) } {}
if let element = array.last { someMatchingLogic($0) } {}
if array.contains { someMatchingLogic($0) } {}

// 对数组元素进行变形，生成新数组
array.map { someTransformation($0) }

// flatMap
// 场景 1：压平数组
let values = [[1,3,5,7],[9]]
let flattenResult = values.flatMap{ $0 } // [1, 3, 5, 7, 9]
// 场景 2：过滤 nil
let values:[Int?] = [1,3,5,7,9,nil]
let flattenResult = values.flatMap{ $0 } /// [1, 3, 5, 7, 9]
// 场景 3：不同数组合并
let words = ["a", "b", "c"]
let nums = ["1", "2", "3"]
let result = words.flatMap { word in
    nums
}

// 对数组中元素执行操作，不生成新数组
// NOTE: 使用 return 时不会中断外部函数
array.forEach { doSomething($0) }

// 测试所有元素
array.allSatisfy {}

// 筛选符合规定的元素
array.filter { someCriteria($0) }

// 将元素聚合成一个值
array.reduce(0) { total, num in total + num }
array.reduce(0, +)

// 排序
// sort 无返回值，修改原数组
sort(by:)
// sorted 返回新的排好序的数组
sorted(by:)
// 按照顺序比较两个集合元素的大小。
lexicographicallyPrecedes(_:by:)
// 根据条件把集合里的元素重新排序，符合条件的元素移动到最后，返回两个部分分界元素的索引
partition(by:)

// 最大值和最小值
min(by:)
max(by:)

// 判断两个队列的是否拥有相同的元素，并且顺序是一致的
elementsEqual(_:by:)
// 序列的前几个元素是否和另一个序列相同
starts(with:)

// 分割数组
split(whereSeparator:)

// 从头取元素知道条件不成立
prefix(while:)

// 从头遍历，条件为真，丢弃元素；条件不成立，返回剩余元素
drop(while:)

// 删除所有条件为真的元素
removeAll(where:)
```

## Set

## Dictionry