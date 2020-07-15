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

## SubSequence


## 专门的集合类型 


## LazySequenceProtocol
