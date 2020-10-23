---
title: C语言指针与字符串
date: 2019/10/13
tags: c
comments: true
---

point 与 string
<!--more-->

## 用指针遍历字符串的所有字符

```C
// 定义一个指针p
char *p;
// 定义一个数组s存放字符串
char s[] = "ab";
// 指针p指向字符串的首字符'm'
p = s; // 或者 p = &s[0];
for (; *p != '\0'; p++) {
    printf("%c \n", *p);
}
```

## 用指针直接指向字符串

* 直接用指针指向一个字符串

```C
#include <string.h>

int main(int argc, char *argv[]) {
    // 定义一个字符串，用指针s指向这个字符串
    char *s = "ab";
    // 使用strlen函数测量字符串长度
    int len = strlen(s);
    printf("字符串长度：%D", len);
    return 0;
}
```

* 字符串常用函数的声明

```C
size_t     strlen(const char *);
char    *strcpy(char *, const char *); // 字符串拷贝函数
char    *strcat(char *, const char *); // 字符串拼接函数
int     strcmp(const char *, const char *); // 字符串比较函数
```

可以看出，都是通过传指针进行处理的

## 指针处理字符串的注意

char a[] = "abc";定义的是一个字符串**变量**
char *p = "abc";定义的是一个字符串**常量**

```C
// 正确
// 定义一个字符串变量"lmj"
char a[] = "abc";
// 将字符串的首字符改为'L'
*a = 'A';
printf("%s", a); //< Abc

// **错误**
char *p2 = "abc";
*p2 = 'A'; // 报错
printf("%s", p2); 
```

