---
title: swift 集合类型
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
let words = ["a", "b"]
let nums = ["1", "2"]
let result = words.flatMap { word in
    nums.map { num in
        (word, num)
    }
}// [("a", "1"), ("a", "2"), ("b", "1"), ("b", "2")]

// 对数组中元素执行操作，不生成新数组
// NOTE: forEach 使用 return 时不会中断外部函数，此时请使用 for in 代替
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

// slice，类型是 ArraySlice
// slice 和 array 使用相同索引来随机访问元素
let slice = array[1...]
```

自己实现一些不在标准库中的函数

```swift
extension Array {
    // 累加，将所有元素合并到一个数组，保留合并时每一步的值
    // [1, 2, 3, 4].accumulate(0, +) // [1, 3, 6, 10]
    func accumulate<Result>(_ initialResult: Result,
                            _ nextPartialResult:(Result, Element) -> Result) -> [Result] {
        var running = initialResult
        return map { next in
            running = nextPartialResult(running, next)
            return running
        }
    }
    // 计算满足条件的元素的个数
    func count(where predicate: (Element) -> Bool) -> Int {
        var result = 0
        for element in self {
            if predicate(element) { result += 1 }
        }
        return result
    }
    // 返回一个满足某个条件的所有元素的索引列表
    func indices(where predicate: (Element) -> Bool) -> [Int] {
        var result = [Int]()
        for (index, element) in self.enumerated() {
            if predicate(element) { result.append(index) }
        }
        return result
    }
}
```

## Dictionry

```swift
var fooDic = ["a": 1, "b": 2]

// 根据键值对创建字典
let temp = Dictionary("abb".map{(String($0), 1)}, uniquingKeysWith: +) // ["a": 1, "b": 2]

// merge
// 第一个参数：需要合并的字典
// 第二个参数：相同键的两个值如何合并的函数
fooDic.merge([temp, uniquingKeysWith: { $1 }) // fooDic = ["a": 1, "b": 2]

// 对字典的值做映射
// 保持字典结构，只对值进行变换
fooDic.mapValues( { doSomething() } )
```

```swift
// 计算序列的频率
extension Sequence where Element: Hashable {
    var frequencies: [Element: Int] {
        return Dictionary(self.map {($0, 1)}, uniquingKeysWith: +)
    }
}
```

## Set

```swift
let iPods: Set = ["iPod touch", "iPod nano", "iPod mini", "iPod shuffle", "iPod Classic"]
let discontinuedIPods: Set = ["iPod mini", "iPod Classic", "iPod nano", "iPod shuffle"]

// 补集
let currentIPods = iPods.subtracting(discontinuedIPods) // ["iPod touch"]

// 交集
let touchscreen: Set = ["iPhone", "iPad", "iPod touch", "iPod nano"]
let iPodsWithTouch = iPods.intersection(touchscreen)

// 并集
var discontinued: Set = ["iBook", "Powerbook", "Power Mac"]
discontinued.formUnion(discontinuedIPods) // ["iPod mini", "iPod Classic", "iPod nano", "iPod shuffle", "iBook", "Powerbook", "Power Mac"]

// 获取序列中所有唯一的元素
extension Sequence where Element: Hashable {
    func unique() -> [Element] {
        var seen: Set<Element> = []
        return filter { element in
            if seen.contains(element) {
                return false
            } else {
                seen.insert(element)
                return true
            }
        }
    }
}
```

Swift 的标准库中，Set 和 OptionSet 实现了 SetAlgebra 协议。

Foundation 中， IndexSet 和 CharacterSet 也实现了 SetAlgebra 协议。

* IndexSet 
  
    IndexSet 是一个正整数的集合。其内部使用了一组范围列表进行实现，比 Set<Int> 更高效。

* CharacterSet

    CharacterSet 是一个高效存储 Unicode 编码的集合。


## Range

```swift
let singleDigitNumbers = 0..<10
let lowercaseLetters = Character("a")...Character("z")

// 根据 range 创建 array
let array = Array(singleDigitNumbers) // [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

// 半开 range 表示 空间隔
5..<5

// 闭合 range 可以包含 最大值
0...Int.max

// 对 range 遍历
// Range 要满足 collection 协议，它的元素要满足 Strideable 协议，且 stride step 是整数
// Strideable 协议，可以通过增加偏移从一个元素移动到另一个元素
// Character 没有实现 Strideable 协议，所以 for c in lowercaseLetters 编译错误
for i in 0..<10 {
    doSomething()
}

// 范围表达式
let arr = [1, 2, 3, 4]
arr[2...] // [3, 4]
arr[..<1] // [1]
arr[1...2] // [2, 3]
arr[...] // [1, 2, 3, 4]
```