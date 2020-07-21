---
title: typedef
date: 2019/10/13
tags: C
comments: true
---

typedef
<!--more-->

## typedef 作用简介

我们可以使用typedef关键字为各种数据类型定义一个新名字

```C
#include <stdio.h>

typedef int Integer;
typedef unsigned int UInteger;
typedef float Float;

int main(int argc, char *argv[]) {
    Integer i = -10;
    UInteger ui = 11;

    Float f = 12.39f;

    printf("%d %d %.2f", i, ui, f);

    return 0;
}
```

也可以在别名的基础上再起一个别名  

```C
typedef int Integer;
typedef Integer MyInteger;
```

## typedef 与 指针

除开可以给基本数据类型起别名，typedef也可以给指针起别名

```C
#include <stdio.h>

typedef char *String;

int main(int argc, char *argv[]) {
    String str = "This is a string!";
    printf("%s", str);
    return 0;
}
```

## typedef 与 结构体

```C
struct MyPoint {
    float x;
    float y;
};

typedef struct MyPoint Point;

int main(int argc, char * argv[]) {
    Point p;
    p.x = 10.0f;
    p.y = 10.0f;
    return 0;
}
```

简化  

```C
typedef struct MyPoint {
    float x;
    float y;
} Point;
```

```C
typedef struct {
    float x;
    float y;
} Point;
```

## typedef 与 指向结构图的指针

typedef 可以给指针、结构体起别名，当然也可以给指向结构体的指针起别名

```C
#include <stdio.h>

typedef struct {
    float x;
    float y;
} Point;

typedef Point *PP;

int main(int argc, char *argv[]) {
    Point point = {10, 20};
    PP p = &point;
    printf("x=%f, y=%f", p->x, p->y);
    return 0;
}
```

## typedef 与 枚举类型

```C
enum Season {spring, summer, autumn, winter};
typedef enum Season Season;
int main(int argc, char *argv[]) {
    Season s = spring;
    return 0;
}
```

简化  

```C
typedef enum Season {spring, summer, autumn, winter} Season;
```

```C
typedef enum {spring, summer, autumn, winter} Season;
```

## typedef 与 指向函数的指针

```C
#include <stdio.h>

//定义函数
int sum(int a, int b) {
    int c = a + b;
    printf("%d + %d = %d", a, b, c);
    return c;
}

typedef int (*MySum)(int, int);

int main(int argc, char *argv[]) {
    MySum p = sum;
    (*p)(4, 5);
    return 0;
}
```

## typedef 与 #define 的区别

只有str1、str2、str3才是指向char类型的指针变量，str4只是个char类型的变量  
因为 #define 是替换字符串  

```C
typedef char *String1;

#define String2 char *

int main(int argc, const char * argv[]) {
    String1 str1, str2;
    
    String2 str3, str4;
    return 0;
}
```
