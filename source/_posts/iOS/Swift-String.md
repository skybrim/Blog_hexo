---
title: swift 中 string
date: 2019/1/1
tags: iOS
comments: true
---

在此记录一下一些 string 的基本操作。  
持续更新...
<!--more-->

## String 索引

* 基础
使用 startIndex 属性可以获取一个 String 的第一个 Character 的索引。
使用 endIndex 属性可以获取最后一个 Character 的后一个位置的索引
endIndex 属性不能作为一个字符串的有效下标，直接使用 endIndex 会报错
如果 String 是空串，startIndex 和 endIndex 是相等的。

```swift
let str1 = "Hello"
print(str1[str1.startIndex]) //H
print(str1[str1.index(before: str1.endIndex)]) //o
print(str1[str1.index(after: str1.startIndex)]) //e
print(str1[str1.index(str1.startIndex, offsetBy: 1)]) //e
print(str1[str1.index(str1.endIndex, offsetBy: -1)]) //o
//使用 indices 属性会创建一个包含全部索引的 Range
for index in str1.indices {
    print("\(str1[index]) ", terminator: "") //H e l l o
}
```

* string.index 转 int

```
let distance = str.distance(from: str.startIndex to: index)
```

## String 插入

* 插入

```swift
var str2 = "Hello"
str2.insert("!", at: str2.endIndex)
print(str2) //Hello!
str2.insert(contentsOf:" world", at: str2.index(before: str2.endIndex))
print(str2) //Hello world!
```

* 删除

```swift
var str3 = "Hello, world!"
str3.remove(at: str3.index(before: str3.endIndex))
print(str3) //Hello world
let range = str3.index(str3.endIndex, offsetBy: -6)..<str3.endIndex
str3.removeSubrange(range)
print(str3) //Hello
```

## SubString

* 需要把结果转化为 String 以便长期存储

```swift
let str4 = "Hello, world!"
let index = str4.firstIndex(of: ",") ?? str4.endIndex
//substring1 是 subString 而不是 String
let substring1 = str4[..<index]
print(substring1) //Hello
let substring2 = str4.prefix(5)
print(substring2) //Hello
let substring3 = str4.suffix(6)
print(substring3) //world!
```

## 遍历

* character 遍历

```swift
let str5 = "Hello, world!"
//遍历出来的是 Character
for character in str5 {
    print("\(character) ", terminator: "") //H e l l o ,   w o r l d !
}
```

* string.index 遍历

```swift
//遍历出来的是 String.Index
for index in str5.indices {
    print("\(str5[index]) ", terminator: "") //H e l l o ,   w o r l d !
}
```

## 获取 string.index

* 从前往后

```swift
let str6 = "Hello, world!"
if let index = str6.firstIndex(of: ",") {
    print(str6[index])
}
```

* 从后往前

```swift
if let index = str6.lastIndex(of: ",") {
    print(str6[index])
}
```

## 字符串与字符串匹配

* string.index 转 int
从 haystack 中寻找 needle，并将起点 String.Index 转为 Int

```swift
var haystack = "Hello,world"
var needle = "world"
//range 中 lowerBound 是起始点，upperBound 是终点
if let index = haystack.range(of: needle)?.lowerBound {
    let distance = haystack.distance(from: haystack.startIndex, to: index)
}
```

## string 与高阶函数的应用

* 把 string 转为数组

```swift
let str7 = "Hello, world!"
let stringArray = str7.map{ String($0) }
print(stringArray) //["H", "e", "l", "l", "o", ",", " ", "w", "o", "r", "l", "d", "!"]
```

* 把字符串数组转为 String

```swift
let strings = ["H", "e", "l", "l", "o"]
let str8 = strings.joined()
print(str8)// Hello
//通过连接字符拼接
let str9 = strings.joined(separator: "-")
print(str9)//H-e-l-l-o
```

* 计算 string 中每个字符出现的频率

```swift
Dictionary("hello".map{ ($0, 1) }, uniquingKeysWith: +)
```

## 其他

* 判断前后缀

```swift
let strLast = "Hello, world!"
if strLast.hasPrefix("Hello") {
    print("strLast has prefix hello")
}
if strLast.hasSuffix("world!") {
    print("strLast has suffix world!")
}
```

* 反转字符串

```swift
print(String(strLast.reversed())) //!dlrow ,olleH
```

* [Character] 转 String

```swift
let chars: [Character] = ["H", "e", "l", "l", "o"]
print(String(chars))
```
