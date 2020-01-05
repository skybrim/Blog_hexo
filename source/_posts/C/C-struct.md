---
title: 结构体
date: 2019/10/13
tags: C
comments: true
---

struct
<!--more-->

## 定义形式

```C
struct　结构体名{
    类型名1　成员名1;
    类型名2　成员名2;
    ...
    类型名n　成员名n;
};

//eg:
struct Student {
    char *name; // 姓名
    int age; // 年龄
    float height; // 身高
};
```

## 定义一个结构体变量

* 先定义结构体类型，再定义变量

```C
struct Student {
    char *name;
    int age;
};

struct Student stu;
```

* 定义结构体类型的同时定义变量

```C
struct Student {
    char *name;
    int age
} stu;
```

* 直接定义结构体类型变量，省略类型名

```C
struct {
    char *name;
    int age;
} stu;
```

## 注意

* 不能递归定义

```C
//错误
struct Student {
    int age;
    struct Student stu;
};
```

* 结构体内可以包含别的结构体

```C
struct Date {
    int year;
    int month;
    int day;
};

struct Student {
    char *name;
    struct Date birthday;
};
```

* 定义结构体类型，只是说明了该类型的组成情况，并没有给它分配存储空间。只有当定义属于结构体类型的变量时，系统才会分配存储空间给该变量

* 结构体变量占用的内存空间是其成员所占内存之和，而且各成员在内存中按定义的顺序依次排列

## 初始化

* 只能在定义变量的同时进行初始化赋值，初始化赋值和变量的定义不能分开

```C
struct Student {
    char *name;
    int age;
};

struct Student stu = {"ABC", 18};
```

## 使用

* 一般对结构体变量的操作是以成员为单位进行的，引用的一般形式为：结构体变量名.成员名

```C
struct Student {
    char *name;
    int age;
};

struct Student stu;

// 访问stu的age成员，赋值
stu.age = 27;
```

* 如果某个成员也是结构体变量，可以连续使用成员运算符"."访问最低一级成员

```C
struct Date {
     int year;
     int month;
     int day;
};

struct Student {
    char *name;
    struct Date birthday;
};

struct Student stu;

stu.birthday.year = 1990;
stu.birthday.month = 8;
stu.birthday.day = 23;
```

* 相同类型的结构体变量之间，可以整体赋值

```C
struct Student {
    char *name;
    int age;
};

struct Student stu1 = {"ABC", 18};

struct Student stu2 = stu1;

printf("age is %d", stu2.age);
```

## 结构体数组

* 定义

```C
struct Student {
    char *name;
    int age;
};
struct Student stu[5];
```

```C
struct Student {
    char *name;
    int age;
} stu[5];
```

```C
struct {
    char *name;
    int age;
} stu[5];
```

* 初始化

```C
struct {
    char *name;
    int age;
} stu[2] = { {"a", 18}, {"b", 18} };
```

## 结构体作为函数参数

将结构体变量作为函数参数进行传递时，其实传递的是全部成员的值，也就是将实参中成员的值一一赋值给对应的形参成员。因此，形参的改变不会影响到实参。

```C
#include <stdio.h>

//定义一个结构体
struct Student {
    int age;
};

void test(struct Student stu) {
    printf("修改前形参：%d \n", stu.age); //< 30
    //修改实参的 age
    stu.age = 10;
    printf("修改后形参：%d \n", stu.age); //< 10
}

int main(int argc, const char *argv[]) {
    struct Student stu = { 18 };
    printf("修改前实参：%d \n", stu.age); //< 30
    //调用函数
    test(stu);
    printf("修改后实参：%d \n", stu.age); //< 30
    return 0;
}
```

## 指向结构体的指针

* 定义形式  
struct 结构体名称 *指针变量名

* 访问结构体成员的方式  
结构体变量名.成员名  
(*指针变量名).成员名  
指针变量名->成员名  

* code

```C
#include <stdio.h>

int main(int argc, char *argv[]) {
    //定义结构体类型
    struct Student {
        char *name;
        int age;
    };

    //定义结构体变量
    struct Student stu = {"abc", 18};

    //定义指向结构体的指针变量
    struct Student *p;

    //指针指向结构体变量 stu
    p = &stu;

    //访问结构体的成员
    //1
    printf("name=%s, age=%d \n", stu.name, stu.age);
    //2
    printf("name=%s, age=%d \n", (*p).name, (*p).age);
    //3
    printf("name=%s, age=%d \n", p->name, p->age);

    return 0;
}
```

