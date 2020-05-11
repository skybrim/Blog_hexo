---
title: swift 中 string
date: 2019/1/1
tags: iOS
comments: true
---

在此记录一下一些 string 的基本操作。  
持续更新...
<!--more-->

在 Swift 中，字符串不同于其他语言（包括 Objective-C），它是值类型而非引用类型，它是多个字符构成的序列（并非数组）。

## 增删改查

* 创建
```swift
let str = "1233"
let num = Int(str)
let number = 123
let string = String(num)
```

* 查
swift 中字符串并非数组，所以不能通过 int 索引来查找

使用 startIndex 属性可以获取一个 String 的第一个 Character 的索引。

使用 endIndex 属性可以获取最后一个 Character 的 **后一个位置的索引** ，所以 endIndex 属性不能作为一个字符串的有效下标，直接使用 endIndex 会报错。

如果 String 是空串，startIndex 和 endIndex 是相等的。

```swift
let str = "Hello"

//// 访问字符串中的单个字符，时间复杂度为O(1)
print(str[str.startIndex]) //H
print(str[str.index(before: str1.endIndex)]) //o
print(str[str.index(after: str1.startIndex)]) //e
print(str[str.index(str1.startIndex, offsetBy: 1)]) //e
print(str[str.index(str1.endIndex, offsetBy: -1)]) //o

//使用 indices 属性会创建一个包含全部索引的 Range
for index in str.indices {
    print("\(str[index]) ", terminator: "") //H e l l o
}

// String.index 转 int
let distance = str.distance(from: str.startIndex, to: index)

// 从前往后查找索引
if let index = str.firstIndex(of: ",") {
    print(str6[index])
}

// 从后往前查找索引
if let index = str.lastIndex(of: ",") {
    print(str6[index])
}
```

* 插入

```swift
var str = "Hello"

//向后添加
str.append("!")
print(str) //Hello!

// 中间插入
str.insert(contentsOf:" world", at: str.index(before: str.endIndex))
print(str) //Hello world!
```

* 删除

```swift
var str = "Hello, world!"

// 删除指定位置字符
str.remove(at: str.index(before: str.endIndex))
print(str) //Hello world

// 批量删除
let range = str.index(str.endIndex, offsetBy: -6)..<str.endIndex
str.removeSubrange(range)
print(str) //Hello
```

## SubString

SubString 不是 String 类型。需要把结果转化为 String 以便长期存储。

```swift
let str = "Hello, world!"
let index = str.firstIndex(of: ",") ?? str.endIndex
//substring1 是 subString 而不是 String
let substring1 = str[..<index]
print(substring1) //Hello
let substring2 = str.prefix(5)
print(substring2) //Hello
let substring3 = str.suffix(6)
print(substring3) //world!
```

## 字符串遍历

* character 遍历

```swift
let str = "Hello, world!"
//遍历出来的是 Character
for character in str {
    print("\(character) ", terminator: "") //H e l l o ,   w o r l d !
}
```

* string.index 遍历

```swift
//遍历出来的是 String.Index
for index in str.indices {
    print("\(str[index]) ", terminator: "") //H e l l o ,   w o r l d !
}
```

## 字符串与字符串匹配

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
let str = "Hello, world!"
let stringArray = str.map{ String($0) }
print(stringArray) //["H", "e", "l", "l", "o", ",", " ", "w", "o", "r", "l", "d", "!"]
```

* 把字符串数组转为 String

```swift
let strings = ["H", "e", "l", "l", "o"]
let str1 = strings.joined()
print(str1)// Hello

//通过连接字符拼接
let str2 = strings.joined(separator: "-")
print(str2)//H-e-l-l-o
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

```swift
// 检测字符串是否是由数字构成
func isStrNum(str: String) -> Bool {
  return Int(str) != nil
}

// 将字符串按字母排序(不考虑大小写)
func sortStr(str: String) -> String {
  return String(str.sorted())
}

// 判断字符是否为字母
char.isLetter

// 判断字符是否为数字
char.isNumber

// 得到字符的 ASCII 数值
char.asciiValue
```
