---
title: swift string
comments: true
date: 2020-07-09 16:49:40
tags: [iOS, swift]
---

swift string

<!--more-->

Swift 中的 String 是 Character 值的集合，Character 是人在阅读文字时所理解的单个字符，与该字符由多少个 Unicode 标量组成无关。

因此，count、prefix(5) 等标准 Collection 操作也会在人所理解的字符上进行。

但是，也带来了一些性能问题：String 不支持随机访问，即跳到字符串某个随机的字符不是一个 O(1) 的操作，必须查看前面的所有字符，才能确定这个字符的位置。


## Unicode

下面列出 Unicode 的一些基本概念

**编码点（code point）**：Unicode 中最基础的原件叫做编码点。举例：欧元符合（€）：U+20AC

**Unicode 标量（Unicode scalar）**：编码点中除了 0xD800 - 0xDFFF 之外的值，都可以叫做 Unicode 标量。

**代理编码点（surrogate code points）**：0xD800 - 0xDFFF 这 2048 个值叫做代理编码点，在 UTF-16 编码中用于表示值大于 65535 的字符。

**编码单元（code units）**：Unicode 编码方式中使用的最小实体叫做编码单元。举例：UTF-8 的编码单元的宽度是 8 比特。

**（扩展）字位簇（(extended)grapheme cluster）**：即用户在屏幕上看到的“单个字符”，可能是一个或者多个 Unicode 标量组合起来的。

Unicode 是一个可变长格式：

    1. 一个 Unicode 字符（扩展字位簇），由一个或多个 Unicode 标量组成
    2. 一个 Unicode 标量，可以被编码成一个或多个编码单元


## 标准等价

* 合并标记

é，这个字位簇，在 Unicode 中可以使单一的标量（U+00E9），也可以是是普通字符 e 后面跟着（U+0301）。

Unicode 规范将上述称作**标准等价**。

Swift 中，Unicode 标量的形式如 \u{xxxx}，类型是 Unicode.Scalar，是一个 struct。举例：欧元符合（€）：\u{20AC}

```swift
let single = "Pok\u{00E9}mon"
let double = "Poke\u{0301}mon"
// 显示一样 都是 Pokémon
(single, douuble)
// 字符数一样
single.count // 7
double.count // 7
// 比较，结果相等
single == double // true

// Unicode 标量，不一样
single.unicodeScalars.count // 7
double.unicodeScalars.count // 8
```

如果将两个字符串转为 Foundation 框架的 NSString，两个字符串**不相等**，length 也不相同。

因为 NSString（也包括其他语言大部分字符串 API），会在 UTF-16 编码单元的层面上，按照字面量比较，而不会将不同字符组合起来的等价性纳入考虑。优势就是**速度快**。如果按照标准等价比较，需要使用 NSString.compare(_:)方法。

* Emoji

很多 Emoji 的 Unicode 标量，无法通过单个 UTF-16 编码单元来表示，因此 java 等其他语言会认为 😂 是两个“字符”长。

但是 swift 可以正确处理：

```swift
let oneEmoji = "😂" //U+1F602
oneEmoji.count // 1
```

## 字符串和集合

Swift 4 后，String 重新成为了 Collection

* 边界情况

两个集合相连接，一般假设新集合的长度是两个相连集合的长度之和。

但是对于字符串，如果前集合的末尾和后集合的开头可以组成一个新的字位簇，则可能不相等。

* 双向索引，而非随机访问

String 不支持随机访问，因此 String 只实现了 BidirectionalCollection。

String 可以从字符串的头或者尾部开始，向后或者向前移动，每次只能迭代一个字符。

```swift
// 获取字符串索引的集合 indices 是一个 O(n) 的操作
// 但是获取索引后的 map 中的下标操作就是 O(1) 了
extension String {
    var allPrefixes: [Substring] {
        return [""] + indices.map{ index in self[...index] }
    }
}
"Hello".allPrefixes // ["", "H", "He", "Hel", "Hell", "Hello"]
```

* 范围可替换，而非可变

String 还实现了 RangeReplaceableCollection 协议。

```swift
var greeting = "Hello, world!"
if let comma = greeting.index(of:",") {
    greeting[..<comma] // hello
    greeting.replaceSubrange(comma..., with:"again.")
}
greeting // Hello again.
```

注意，用于替换的字符串，可能与原字符串相邻字符形成新的字位簇


## 字符串索引

String.Index 是 String 所使用的的索引类型，本质是一个存储了从字符串开头的字节偏移量的不透明值。

一旦有了有效的索引，可以通过索引下标以 O(1) 的时间对字符串进行访问，通过已有索引来寻找下一个索引也更高效。

```swift
let s = "abcdef"
let second = s.index(after: s.startIndex)
s[second] //b

//步进 4 个字符
let sixth = s.index(second, offsetBy: 4)
s[sixth] // f

let safeIdx = s.index(s.startIndex, offsetBy: 400, limitedBy: s.endIndex)
safeIdx // nil

s[..<s.index(s.startIndex, offsetBy: 4)] // abcd
s.prefix(4) // abcd
```

```swift
let date = "2020-07-01"
date.split(separator: "-")[1] // 09
date.dropFirst(5).prefix(2) // 09
```

```swift
var hello = "Hello!"
if let idx = hello.firstIndex(of: "!") {
    hello.insert(contentsOf: ", world", at: idx)
}
hello // Hello, world!
```


## 子字符串

String 的 SubSequence 类型：Substring。

```swift
// 扩展一个接受含有多个分隔符的序列作为参数的 spilt 方法
extension Collection where Element: Equatable {
    func split<S: Sequence>(separators: S) -> [SubSequence]
        where Element == S.Element {
            return split { separators.contains($0) }
        }
}
```

* StringProtocol

String 和 Substring 都遵守 StringProtocol 协议，字符串几乎所有 API 都定义在这个协议里。

不建议长期持有子字符串，这是因为子字符串会一直持有整个原始字符串。

通常，在一个操作内部使用子字符串，而只在结束时创建新字符串，将赋值操作推迟到最后一刻，这样可以确保是最低开销。

如果想扩展 String，可以将这个扩展放在 StringProtocol，可以保持 String 和 Substring 的 API 统一性。


## String 与 NSString

任意的 String 实例，可以通过 as 操作桥接为 NSString，那些接受或者返回 NSString 的 Objective-C API 也会把类型自动转换为 String。

Swift 中 String 的编译器优化：引入 Foundation 后，NSString 成员可以在 String 的实例上进行访问。

由于 Swift String 在内存中的原生编码是 UTF-8，而 NSString 是 UTF-16，因此频繁的在 string 和 NSString 之间桥接会有额外性能开销。

举例：

NSAttributeString 的 API ```attributes(at: Int, effectiveRange: NSRangePointer?)``` 

接受的是一个整数索引（UTF-16 测量），非 String.Index

通过指针返回的 effectiveRange，是一个 NSRange 结构体，而非 Range<String.Index>

```swift
// 为字符串中的 “Click here” 添加一个链接

let text = "👉 Click here for more info."
let linkTarget = URL(string: "https://www.baidu.com")!
let formatter = NSMutableAttributedString(string: text)

//修改文本的部分属性
if let linkRange = formatted.string.range(of: "Click here") {
    let nsRange = NSRange(linkRange, in: formatted.string) // {3, 10}
    formatted.addAttribute(.link, value:linkTarget, range: nsRange)
}
```

```swift
if let queryRange = formatted.string.range(of: "here") {
    let nsRange = NSRange(queryRange, in: formatted.string)
    var attributesRange = NSRange()
    let attributes = formatted.attributes(at: nsRange.location, effectiveRange: &attributesRange)

    attributesRange // {3, 10}

    // 把 NSRange 转为 Range<String.Index>
    if let effectiveRange = Range(attributesRange, in: formatted.string) {
        formatted.string[effectiveRange] // Click here
    }
}
```

* CharacterSet

CharacterSet 是 Foundation 中的类型，实际上它表示一系列 Unicode 标量的数据结构体，和 Character 类型不兼容。

CharacterSet 提供了一些工厂初始化方法，.alphanumerics 、 .whitespacesAndNewlines。

```swift
// 因为 emoji 中，女人 + ZWJ + 消防车 = 女消防员
// ZWJ 零连接符:U+200D
let  favoriteEmoji = CharacterSet("👩‍🚒".unicodeScalars)
favoriteEmoji.contains("🚒") // true
```


## Unicode 属性

* Unicode.Scalar

Swift5 中，CharacterSet 的部分功能移植到了 Unicode.Scalar

```swift
("😀" as Unicode.Scalar).properties.isEmoji // true
("∬" as Unicode.Scalar).properties.isMath // true
```


## String 和 Character 的内部结构

* String

Swift5 里，原生字符串的内存是使用 UTF-8 格式表示的。因此遍历 UTF-8 视图会比遍历 UTF-16 或 Unicode标量 视图更快。

小于 16 个（32 位平台是 11 个）UTF-8 编码单元的小型字符串，Swift 不会为其创建专门的存储缓冲区，而是直接使用内连的方式存储。

* Character

```swift
public struct Character {
    internal var _str: String

    internal init(unchecked str: String) {
        self._str = str
        // ...
    }
}
```

通过源码可知，一个 Character 在内部被表示为长度为 1 的字符串。

Swift5 之后，Character 的优化通过 String 自身的优化实现小字符串优化。


## 字符串字面量

""是字符串字面量，通过 ExpressibleByStringLiteral 协议实现。

字符串字面量是 ExpressibleByStringLiteral，ExpressibleByExtendedGraphemeClusterLiteral 和 ExpressibleByUnicodeScalarLiteral 三个协议的一部分。其中，每个协议都约束了一个用它们各自表示的字面量 创建对象的 init方法。但除非你真的需要根据 Unicode标量还是字位族仔细调整初始化逻辑， 否则，只需要实现字符串的版本就好。

```swift
// 自定义类型 SafeHTML 支持字符串字面量创建。
extension String {
    var htmlEscaped: String {
        // 简单的 仅替换所有开闭尖括号
        return replacingOccurrences(of: "<", with: "&lt;").replacingOccurrences(of: ">", with: "&gt;")
    }
}

struct SafeHTML {
    private(set) var value: String

    init(unsafe html: String) {
        self.value = html.htmlEscaped
    }
}

extension SafeHTML: ExpressibleByStringLiteral {
    public init(stringLiteral value: StringLiteralType) {
        self.value = value
    }
}

let safe: SafeHTML = "<p>Angle brackets in literals are not escaped</p>"
// SafeHTML(value: "<p>Angle brackets in literals are not escaped</p>")
```


## 定制字符串描述

```swift
extension SafeHTML: CustomStringConvertible {
    var description: String {
        return value
    }
}

extension SafeHTML: CustomeDebugStringConvertible {
    var debugDescription: String {
        return "SafeHTML:\(value)"
    }
}
print(safe)
```

