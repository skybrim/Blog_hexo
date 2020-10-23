---
title: 枚举
date: 2019/10/13
tags: c
comments: true
---

enum
<!--more-->

## 概念

枚举是C语言中的一种基本数据类型，并不是构造类型，它可以用于声明一组常数。

## 定义枚举类型

一般形式为：enum　枚举名　{枚举元素1,枚举元素2,……};

```C
enum Season {spring, summer, autumn, winter};
```

## 定义枚举变量

```C
enum Season {spring, summer, autumn, winter};
enum Season s;
```

```C
enum Season {spring, summer, autumn, winter} s;
```

```C
enum {spring, summer, autumn, winter} s;
```

## 使用注意

C语言编译器会将枚举元素(spring、summer等)作为整型常量处理，称为枚举常量。  
枚举元素的值取决于定义时各枚举元素排列的先后顺序。默认情况下，第一个枚举元素的值为0，第二个为1，依次顺序加1。  
也可以在定义枚举类型时改变枚举元素的值，没有指定值的枚举元素，其值为前一元素加1。  

```C
enum Season {spring, summer=3, autumn, winter};
```

## 基本操作

* 赋值

```C
enum Season {spring, summer, autumn, winter} s;
s = spring;
s = 3 // 等价 s = winter;
```

* 遍历

```C
enum Season {spring, summer, autumn, winter} s;
for (s = spring; s <= winter; s ++) {
    printf("枚举元素：%d \n", s);
}
```