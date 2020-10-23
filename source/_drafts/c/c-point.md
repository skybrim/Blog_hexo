---
title: C语言指针
date: 2019/10/13
tags: c
comments: true
---

point
<!--more-->

## 直接引用

通过变量名引用变量，由系统自动完成变量名和其存储地址之间的转换，称为变量的"直接引用"方式  

```C
char a;
a = 10;
```

## 指针

首先将变量a的地址存放在另一个变量中，比如存放在变量b中，然后通过变量b来间接引用变量a，间接读写变量a的值。  
变量b就是个**指针变量**。  
用来存放**变量地址**的变量，就称为**指针变量**。

## 定义指针

```C
// 定义int类型的变量a
int a = 10;
// 定义一个指针变量p
int *p;
// 将变量a的地址赋值给指针变量p，所以指针变量p指向变量a
p = &a;
```

```C
// 定义int类型的变量a
int a = 10;
// 定义一个指针变量p
// 并将变量a的地址赋值给指针变量p，所以指针变量p指向变量a
int *p = &a;
```

## 指针运算符

* 给指针指向的变量赋值

```C
char a = 10;
printf("修改前，a的值：%d\n", a);
// 指针变量p指向变量a
char *p = &a;
// 通过指针变量p间接修改变量a的值
*p = 9;
printf("修改后，a的值：%d", a);
```

* 取出指针所指向变量的值

```C
char a = 10;
char *p;
p = &a;
char value = *p;
printf("取出a的值：%d", value);
```

## 应用

* swap

```C
void swap(char *v1, char *v2) {
    // 中间变量
    char temp;
    // 取出v1指向的变量的值
    temp = *v1;
    // 取出v2指向的变量的值，然后赋值给v1指向的变量
    *v1 = *v2;
    // 赋值给v2指向的变量
    *v2 = temp;
}
int main() {
    char a = 10, b = 9;
    printf("更换前：a=%d, b=%d\n", a, b);
    swap(&a, &b);
    printf("更换后：a=%d, b=%d", a, b);
    return 0;
}
```

* 返回多个值

```C
// 计算2个整型的和与差
int sumAndMinus(int v1, int v2, int *minus) {
    // 计算差，并赋值给指针指向的变量
    *minus = v1 - v2;
    // 计算和，并返回和
    return v1 + v2;
}
int main(int argc, char *argv[]) {
    // 定义2个int型变量
    int a = 6, b = 2;
    // 定义2个变量来分别接收和与差
    int sum, minus;
    // 调用函数
    sum = sumAndMinus(a, b, &minus);
    // 打印和
    printf("%d+%d=%d\n", a, b, sum);
    // 打印差
    printf("%d-%d=%d\n", a, b, minus);
    return 0;
}
```

## 补充

* 指针变量占用多少字节内存空间  

![指针变量占用的内存空间](https://cdn.jsdelivr.net/gh/skybrim/AllImages@dev/%E6%8C%87%E9%92%88%E5%8D%A0%E7%94%A8%E7%9A%84%E5%86%85%E5%AD%98%E7%A9%BA%E9%97%B4.png)

* 指针类型的作用

根据类型，决定读取内存区域