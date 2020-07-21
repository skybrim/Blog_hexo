---
title: swift 集合协议
comments: true
date: 2020-07-07 14:15:41
tags: [iOS, swift]
---

《swift advance》 读书笔记：swift 集合协议

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

    **不允许向集合中添加和删除元素**

* RangeReplaceableCollection

    提供了替换集合中一个连续区间的元素的能力。

    通过扩展，衍生出 append 和 remove 等方法

    Array/String 支持

    Set/Dictionary 不支持

* BidirectionalCollection

    提供了从集合尾部想集合头部遍历的能力

* RandomAccessCollection

    提供了更高效的索引计算能力：计算索引的距离或者移动索引位置都是常数时间操作。

    举例：Array 是随机访问的集合，但是 String 不是，因为计算两个字符之间的举例是一个线性时间的操作。

* LazySequenceProtocol

    定义了一个只有在开始遍历时才计算其中元素的**序列**。

    可以接受一个无穷序列，从中筛选元素，然后读取结果中的前几个记录。

* LazyCollectionProtocol

    与 LazySequenceProtocol 类似，定义一个只有在开始遍历时才计算其中元素的**集合类型**
    
## Sequence

Sequence 1: 有一个关联类型 Element; 2: 有一个创建迭代器的方法。

```swift
// 简化的 Sequence 定义
protocol Sequence {
    associatedtype Element where Self.Element == Self.Iterator.Element
    associatedtype Iterator:  IteratorProtocol

    func makeIterator() -> Self.Iterator
    // ...
}
```

* Iterator（迭代器）

序列通过创建迭代器来提供对元素的访问。

迭代器每次产生序列中的一个值，并对序列的遍历状态管理。

迭代器遵循 IteratorProtocol。

```swift
protocol IteratorProtocol {
    associatedtype Element
    mutating func next() -> Element?
}
```

IteratorProtocol 的关联类型 Element 指定了迭代器产生的值得类型。
同时在 Sequence 协议中，通过序列的关联类型 ELement 的类型约束```where Self.Element == Self.Iterator.Element```，指定迭代器及对应序列的元素相同

IteratorProtocol 协议唯一的方法 next() ，这个方法在每次被调用时返回序列的下一个值，当序列结束，返回 nil。

迭代器作用：for 循环是通过迭代器实现的

```swift
// for 循环的实质
var iterator = someSequence.makeIterator()
while let element = iterator.next() {
    doSomething(whit:element)
}
```

迭代器特点：

单向结构，只能按照增加的方向前进，不能后退或重置。

通过迭代器不断迭代，可以创建一个无限的序列。

* 遵循 Sequence 创建序列

```swift
struct PrefixIterator: IteratorProtocol {
    let string: String
    var offset: String.Index

    init(string: String) {
        self.string = string
        offset = string.startIndex
    }

    mutating func next() -> Substring? {
        guard offset < string.endIndex else { return nil }
        offset = string.index(after: offset)
        return string[...<offset]
    }
}

struct PrefixSequence: Sequence {
    let string: String
    func makeIterator() -> PrefixIterator {
        return PrefixIterator(string: string)
    }
}

for prefix in PrefixSequence(string: "abc") {
    print(prefix)
}
/*
a
ab
abc
*/
```

* 一些特性

1. 迭代器与值语义：
   标准库中大部分迭代器都具有值语义，复制后所有状态都会被复制，而且相互独立；

   AnyIterator 是一个队别的迭代器进行封装的迭代器，封装原理是将另外的迭代器包装到一个内部的盒子对象中，而这个对象是引用类型。
   因此 AnyIterator 对象的复制，可能会导致多个迭代器共享同一个引用。

2. 基于函数的迭代器和序列
   AnyIterator 有可以接受 next 函数作为参数的初始化方法。

   配合 AnySequence，可以在不定义新的类型的情况下，创建迭代器和序列。

3. 单词遍历序列
   Sequence 文档指出，序列并不能保证可以被多次遍历（如网络包流），collection 协议才能保证多次迭代是安全的

4. 序列与迭代器关系
   支持多次遍历的序列，需要独立的遍历状态，所以分离迭代器与序列

   迭代器也可以视为由他们返回的元素所组成的单次遍历序列。
   只要声明迭代器遵循 Sequence 协议，它就是一个序列类型，Sequence 会为迭代器提供一个默认的 makeIterator 实现，这个方法返回的 self 本身。


## Collection

集合类型（Collection）指的是那些可以被多次遍历且保持一致的序列。

除了线性遍历意外，集合中的元素也可以通过下标索引的方式访问。

集合类型不能是无限的。

每一个集合类型都有一个关联类型 SubSequence，表示集合中一段连续内容的切片。

Collection 继承了 Sequence。

实现 Collection 协议，最难的是选取合适的索引类型来表达集合类型中的位置。

实现 Collection 的类型:

swift 标准库：Array/Dictionary/Set/String/[Closed]Range/UnsafeBufferPointer

Foundation: Data/IndexSet

```swift
// 除了 Index 和 Element 意外，其他关联类型都是有默认值得，大部分方法、属性、下标也是如此。
protocol Collection : Sequence {
    // 继承自 Sequence
    associatedtype Element
    associatedtype Iterator = IndexingIterator<Self>
    override func makeIterator() -> Self.Iterator

    associatedtype Index : Comparable // 省略大量的 where 语句
    var startIndex: Self.Index { get }
    var endIndex: Self.Index { get }

    // 使用递归约束表明 SubSequence 自身也是一个集合类型
    // 且 Collection 元素与 SubSequence 元素相同
    // 且 SubSequence 中的 SubSequence 类型与自身相同
    associatedtype SubSequence : Collection = Slice<Self> where Self.Element == Self.SubSequence.Element, Self.SubSequence == Self.SubSequence.SubSequence

    subscript(position: Self.Index) -> Self.Element { get }
    subscript(bounds: Range<Self.Index>) -> Self.SubSequence { get }

    // Indices 也是集合类型
    // Collection 中的 Index 是这个 Indices 的元素类型、索引类型以及 Indices.SubSequence 中的索引类型
    associatedtype Indices : Collection = DefaultIndices<Self> where Self.Indices == Self.Indices.SubSequence

    var indices: Self.Indices { get }
    var isEmpty: Bool { get }
    var count: Int { get }

    func index(_ i: Self.Index, offsetBy distance: Int) -> Self.Index
    func index(_ i: Self.Index, offsetBy distance: Int, limitedBy limit: Self.Index) -> Self.Index?
    func distance(from start: Self.Index, to end: Self.Index) -> Int
    func index(after i: Self.Index) -> Self.Index
    func formIndex(after i: inout Self.Index)
}
```

* 自定义集合类型

实现一个队列（Queue），遵循 Collection

```swift
//先展示一个简单的先进先出队列
struct FIFOQueue<Element> {
    private var left: [Element] = []
    private var right: [Element] = []

    // 将元素添加到队列最后 O(1)
    mutating func enqueue(_ newElement: Element) {
        right.append(newElement)
    } 

    // 从队列前端移除一个元素，队列为空时返回 nil O(1)
    mutating func dequeue() -> Element? {
        if left.isEmpty {
            left = right.reversed()
            right.removeAll()
        }
        return left.popLast()
    }
}
```

根据上面 Collection 协议的源码分析，我们最后需要实现如下：

```swift
protocol Collection: Sequence {

    //一个表示序列中元素的类型
    associatedtype Element

    // 一个表示集合中位置的类型
    associatedtype Index: Comparable

    // 一个非空集合中首个元素的位置
    var startIndex: Index { get }

    // 比最后一个有效下标大 1 的值
    var endIndex: Index { get }

    // 返回在给定索引之后的位置
    func index(after i: Index) -> Index

    // 下标访问特定位置的元素
    subscript(position: Index) -> Element { get }
}
```

最终，实现 FIFOQueue，满足 Collection，代码如下：

```swift
extension FIFOQueue: Collection {
    public var startIndex: Int { return 0 }
    public var endIndex: Int { return left.count + right.count }

    public func index(after i: Int) -> Int {
        precondition( i >= startIndex && i < endIndex, "Index out of bounds")
        return i + 1
    }

    public subscript(position: Int) -> Element {
        precondition((startIndex..<endIndex).contains(position), "Index out of bounds")
        if position < left.endIndex {
            return left[left.count - position - 1]
        } else {
            return right[position - left.count]
        }
    }
}
```

* 数组字面量

让队列实现 ExpressibleByArrayLiteral 协议，这样可以通过字面量来创建一个队列

```swift
extension FIFOQueue: ExpressibleByArrayLiteral {
    public init(arrayLiteral elements: Element...) {
        self.init(left:elements.reversed(), right:[])
    }
}

let foo: FIFOQueue = [1, 2, 3]
```

## Index

索引（Index），表示了集合中的位置，集合类型的索引，必须实现 Comparable 协议，即索引必须要有确定的顺序。

使用字典举例：

Dictionary，它的索引是 DictionaryIndex 类型，是一个指向字典内部存储缓冲区的不透明值。

平时使用建访问字典的 subscript(_ key: Key) 方法，是对定义在 Dictionary 中的 subscript 方法的重载。

字典的 ```subscript(key: Key) -> Value?``` ，返回的是一个可选值

集合类型的 subscript 方法，在 Collection 中，返回的是非可选值 ```subscript(position: Index) -> Element { get }```

注意，Element 类型在字典中是一个元组：(key: Key, value: Value)，所以，Dictionary 的下标访问返回的是一个键值对，也因此对 Dictionary 遍历我们得到的是键值对。

* 索引失效

当集合发生改变时，索引可能会失效：

1. 索引本身仍然有效，但是指向了另外的元素
2. 索引本身无效，通过索引访问集合将造成崩溃

* 索引步进

当前 swift 版本，通过给定索引计算新的索引，由集合本身负责。

这样可以提高性能：

String ，由于 Character 在 Swift 中的尺寸是可以改变的，因此计算 String 的索引，必须考虑 Character 的实际内容。

* 自定义集合索引

分割集合类型，```split```方法通常最合适，但是这个方法会计算整个数组。如果数组很大，但是只需要前几个元素，这样做效率低。

下面用 String(英文) 举例：

目标是，构建一个 words 集合，它能够让我们不一次性计算出所有单词，而是可以用延迟加载的方式进行迭代。

首先，从 SubString 中寻找第一个单词的范围。

使用空格作为单词的边界。
```swift
extension Substring {
    var nextWordRange: Range<Index> {
        // 移除所有前置的空格
        let start = drop(while: { $0 == " "} )
        // 寻找结束空格，如果没有，则使用 endIndex
        let end = start.firstIndex(where: { $0 == " "}) ?? endIndex
        return start.startIndex..<end
    }
}
```

第二，定义索引的类型。

通过索引下标访问某个元素，应该是一个 O(1) 的操作，因此封装 Range<Substring.Index> 来作为索引类型。

索引类型需要满足 Comparable(继承自 Equatable)，此时我们采用 range 的下边界作为比较对象。
```swift
struct WordsIndex: Comparable {
    fileprivate let range: Range<Substring.Index>

    fileprivate init(_ value: Range<Substring.Index>) {
        self.range = value
    }

    static func <(lhs: Words.Index, rhs: Words.Index) -> Bool {
        return lhs.range.lowerBound < rhs.range.lowerBound
    }

    static func ==(lhs: Words.Index, rhs: Words.Index) -> Bool {
        return lhs.range == rhs.range
    }
}
```

第三，构建 Words 集合类型。

在底层将 String 视为 SubString 存储；

提供两个属性： startIndex 、 endIndex；

同时，需要遵循 Collection 协议，并实现 subscript 下标方法。
```swift
struct Words {
    let string: Substring
    let startIndex: WordsIndex
    public var endIndex: WordsIndex {
        let e = string.endIndex
        return WordsIndex(e..<e)
    }

    init(_ s: String) {
        self.init(s[...])
    }

    private init(_ s: Substring) {
        self.string = s
        self.startIndex = WordsIndex(string.nextWordRange)
    }
}

extension Words {
    subscript(index: WordsIndex) -> Substring {
        return string[index.range]
    }
}

extension Words: Collection {
    public func index(after i: WordsIndex) -> WordsIndex {
        guard i.range.upperBound < string.endIndex else {
            return endIndex
        }
        let remainder = string[i.range.upperBound...]
        return WordsIndex(remainder.nextWordRange)
    }
}
```

最后，使用自定义索引

```swift
Array(Words("hello world test")) // ["hello", "world", "test"]

Array(Words("Hello world test").prefix(2)) // ["hello", "world"]
```

补充：创建自定义集合类型时，优先考虑能否使用其本身作为自己的 SubSequence 使用

```swift
extension Words {
    // 重载 subscript(range:Range<Index>)
    subscript(range:Range<WordsIndex>) -> Words {
        let start = range.lowerBound.range.lowerBound
        let end = range.upperBound.range.upperBound
        return Words(string[start..<end])
    }
}
```

## SubSequence

在 Swift5 中，SubSequence 被定义到了 Collection 协议中，是一个关联类型，表示集合中一个连续的子区间。

默认情况下，Collection 把 Slice<Self> 作为自己的 SubSequence 类型。

子序列和原始的集合类型共享内部存储。

```swift
extension Collection {
    associatedtype SubSequence: Collection = Slice<Self> 
        where Element == SubSequence.Element, SubSequence == SubSequence.SubSequence
}
```

扩展：以每 n 个元素切割集合

```swift
extension Collection {
    public func split(batchSize: Int) -> [SubSequence] {
        var result: [SubSequence] = []
        var batchStart = startIndex
        while batchStart < endIndex {
            let batchEnd = index(batchStart, offsetBy: batchSize, limitedBy: endIndex) ?? endIndex
            let batch = self[batchStart..<batchEnd]
            result.append(batch)
            batchStart = batchEnd
        }
        return result
    }
}

let letters = "abcdefg"
let batches = letters.split(batchSize: 3) // ["abc", "def", "g"]
```

* 切片

所有集合类型，都有切片操作的默认实现，并有一个下标方法：```subscript(range:Range<Index>) -> Slice<Base>```

Slice 非常适合作为默认的切片类型，不过当创建自定义集合时，优先考虑集合本身作为切片类型。详见 [Index](#Index)。

Slice 是基于任意集合类型的一个轻量级封装：

```swift
struct Slice<Base: Collection>: Collection {
    typealias Index = Base.Index
    let collection: Base

    var startIndex: Index
    var endIndex: Index

    init(base: Base, bounds: Range<Index>) {
        collection = base
        startIndex = bounds.lowerBound
        endIndex = bounds.upperBound
    }

    func index(after i:Index) -> Index {
        return collection.index(after: i)
    }

    subscript(position: Index) -> Base.Element {
        return collection[position]
    }

    subscript(bounds: Range<Index>) -> Slice<Base> {
        return Slice(base: collection, bounds: bounds)
    }
 }
```

* 切片与原集合共享索引

Collection 协议要求切片的索引和原集合的索引互换使用

集合类型和它的切片拥有相同的索引。

只要集合和它的切片在切片被创建后没有改变，切片中的某个索引位置上的元素，应当也存在于原集合中同样的索引位置上。

推荐使用索引的方式： ```for index in collection.indices```，但不要在迭代时修改集合。


## 专门的集合类型 

Collection 有两个限制：

1. 无法往回移动索引；
2. 没有提供像插入、移除或替换元素这样的改变集合内容的功能。

* BidirectionalCollection

一个既支持向前又支持向后遍历的集合，继承 Collection

提供```index(before:)```方法把索引往回移动一个位置

有了向前遍历集合的能力，BidirectionalCollection 实现了一些可以高效执行的方法:suffix，removeLast，reversed

```swift
extension BidirectionalCollection {
    // 集合中的最后一个元素
    public var last: Element? {
        return isEmpty ? nil : self[index(before: endIndex)]
    }
    //  返回一个表达集合中元素逆序排列的视图，复杂度 O(1)
    // ReversedCollection 并不会真的把元素逆序排列，而是会持有原来的集合
    // 并使用一个定制的索引类型实现逆序遍历
    public func reversed() -> ReversedCollection<Self> {
        return ReversedCollection(_ base: self)
    }
}
```

* RandomAccessCollection

一个支持高效随机访问索引进行遍历的集合，继承 BidirectionalCollecton

可以在常数时间内跳转到任意索引，需要在常数时间内完成两个操作：第一，可以任意距离移动一个索引；第二，测量任意两个索引之间的距离。

实现```index(_:offsetBy:)```和```distance(from:to:)```，或者 Index 类型满足 Strideable(比如 Int)

RandomAccessCollection 以更严格的约束重新声明了关联的 Indices 和 SubSequence 类型，这两个类型自身也必须是可以进行随机存取的。

补充：二分搜索算法，必须搭配随机存取集合。

* MutableCollection

一个支持下标赋值的集合，继承自 Collection

要求：单个元素的下标访问方法```subscript```现在必须提供一个 setter：

```swift
// 将 FIFOQueue 的协议由 Collection 扩展为 MutableCollection
// 队列支持原地的元素更改
extension FIFOQueue: MutableCollection {
    public var startIndex: Int { return 0 }
    public var endIndex: Int { return left.count + right.count }

    public func index(after i: Int) -> Int {
        precondition( i >= startIndex && i < endIndex, "Index out of bounds")
        return i + 1
    }

    public subscript(position: Int) -> Element {
        get {
            precondition((0..<endIndex).contains(position), "Index out of bounds")
            if position < left.endIndex {
                return left[left.count - position - 1]
            } else {
                return right[position - left.count]
            }
        }
        set {
            precondition((0..<endIndex).contains(position), "Index out of bounds")
            if position < left.endIndex {
                left[left.count - position - 1] = newValue
            } else {
                return right[position - left.count] = newValue
            }
        }
    }
}
```

* RangeReplaceableCollection

一个支持将任意范围的元素用别的集合中的元素进行替换的集合，继承自 Collection

此协议两个要求：

1. 一个空的初始化方法；（在泛型函数中很有用，允许一个函数创建相同类型的空集合）
2. 一个 replaceSubrange(_:with:)方法。（接受一个要替换的范围以及一个用来进行替换的集合）

只需要实现一个灵活的 replaceSubrange(_:with:)方法，协议扩展就可以引申一系列有用的方法：

1. append(_:) 和 append(contentsOf:)
2. remove(at:) 和 removeSubrange(_:)
3. insert(at:) 和 insert(contentsOf:at:)
4. removeAll

```swift
//举例 FIFOQueue
extension FIFOQueue: RangeReplaceableCollection {
    mutating func replaceSubrange<C:Collection> (_ subrange: Range<Int>, with newElements: C)
        where C.Element == Element {
            right = left.reversed() + right
            left.removeAll()
            right.replaceSubrange(subrange, with: newElements)
        }
}
```

* 组合能力

```swift
extension MutableCollection 
    where Self: RandomAccessCollection, Element: Comparable {
        // 原地对集合进行排序
        public mutating func sort() { ... }
    }
```

## 延迟序列

LazySequenceProtocol 和 LazyCollectionProtocol。

延迟编程：结果只有在真正需要的时候才会计算出来。

swift 标准库为支持延迟编程，提供了两个协议：LazySequenceProtocol 和 LazyCollectionProtocol。

LazySequenceProtocol 继承自 Sequence。

LazyCollectionProtocol 继承自 LazySequenceProtocol。

```swift
// 需要标准输入收到 EOF 信号才会显示
let filtered = standardIn.filter {
    $0.split(separator: " ").count > 3
}
for line in filtered { print(line) }

// 每读一行符合条件的结果，就打印一个消息
for line in standardIn {
    guard line.split(separator: " ").count > 3 else { continue }
    print(line)
}

// 想保留函数式风格，需要在获取到值得时候就生成筛选后的结果
// .lazy 返回值类型是 LazySequence<Self>
// LazySequence 的 filter 方法会返回一个 LazyFilterSequence<AnySequence<String>>
let filtered = standardIn.lazy.filter {
    $0.solit(separator: " ").count > 3
}
for line in filtered { print(line) }
```

风格比较

```swift
// 函数式
let result = Array(standardIn.lazy.filter {
    $0.split(separator: " ").count > 3
}.prefix(2))

// 命令式
var result: [String] = []
for line in standardIn {
    guard line.split(separator: " ").count > 3 else { continue }
    result.append(line)
    if result.count == 2 { break }
}
```

* 集合的延迟处理

当在一个常规集合类型（Array）上，串联多个操作时，可以延迟处理的序列，会更效率。

```swift
(1..<100).map { $0 * $0 }.filter {$0 > 10 }.map { "\($0)" }
```

函数式编程，代码清晰易于理解，但是存在效率不佳的问题：

每一次调用 map 和 filter，都会创建一个新的包含中间结果的数组，这个数组在返回的时候就被销毁了。

通过在这个调用链的开始，插入 .lazy，就不会产生任何保存中间结果的数组，更有效率

```swift
(1..<100).lazy.map { $0 * $0 }.filter {$0 > 10 }.map { "\($0)" }
```

LazyCollectionProtocol 扩展了 LazySequence，它要求实现它的类型也是一个实现了 Collection 的类型。

在 LazySquence 中，我们只能逐个生成 Sequence 中的每个元素。

在 LazyCollection 中，我们可以直接按需生成指定的某个元素。

```swift
// 当对一个延迟加载的集合，应用 map 方法
// 通过下标访问其中的元素
// map 只会对你访问的那个结果执行变换。 O(1)
let allNumbers = 1..<1_000_000
let allSquares = allNumbers.lazy.map { $0 * $0 }
print(allSquares[50]) // 2500

// 注意
// 当使用下标访问元素的时候，每一次的结果都是计算出来的
// 比如下面的代码， 50 已经被过滤掉了
// 为了获取值，需要计算前 50 个的值，然后才能获取到 50 的值（第 51 个）
// 显然这是一个 O(n) 的操作
let largeSquares = allNumbers.lazy.filter { $0 > 1000 }.map { $0 * $0 }
print(largeSquares[50]) // 2500
```

