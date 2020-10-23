---
title: C语言数据类型
date: 2019/10/13
tags: c
comments: true
---

数据类型
<!--more-->

## 不同编译器环境下基本数据类型的存储长度

![不同编译器环境下基本数据类型的存储长度](https://cdn.jsdelivr.net/gh/skybrim/AllImages@dev/%E4%B8%8D%E5%90%8C%E7%BC%96%E8%AF%91%E5%99%A8%E7%8E%AF%E5%A2%83%E4%B8%8B%E5%9F%BA%E6%9C%AC%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B%E7%9A%84%E5%AD%98%E5%82%A8%E9%95%BF%E5%BA%A6.png)

## 基本数据类型

* int
* float
* double
* char

>char的取值范围是：ASCII码字符 或者 -128~127的整数  
>char只能存储一个字符。汉字是两个字符，所以 char 不能存储单个汉字

```C
//下面两种方式等效，A 的 ASCII 值为 65
// 方式1
char c = 'A';
// 方式2
char c = 65;
```

## 指针类型

* void *

## 空类型

* void

## 构造类型

* array
* struct
* union
* enum

## 类型修饰符

常用语修饰 int，可以省略 int

* short  短型
* long  长型
* signed  有符号型
* unsigned  无符号型

>short跟int至少为16位(2字节)
>long至少为32位(4字节)
>short的长度不能大于int，int的长度不能大于long
>char一定为为8位(1字节)，毕竟char是我们编程能用的最小数据类型
>signed和unsigned的区别就是它们的最高位是否要当做符号位
>signed、unsigned也可以修饰char，long还可以修饰double

## 变量

```C
// 声明全局变量，默认赋值 0
int a;
int main(int argc, char *argv[]) {
    //声明局部变量。局部变量不赋值，会得到垃圾数据，必须赋值使用
    int b;
    printf("%d", b);
    return 0;
}
// 函数的实现（函数的定义）
int sum(int a, int b) {
    return a + b;
}
```
