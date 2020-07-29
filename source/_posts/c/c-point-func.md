---
title: 返回指针的函数与指向函数的指针
date: 2019/10/13
tags: c
comments: true
---

point 与 func
<!--more-->

## 返回指针的函数

指针也是C语言中的一种数据类型，因此一个函数的返回值肯定可以是指针类型的。  
返回指针的函数的一般形式为：类型名 * 函数名(参数列表)

```C
// 将字符串str中的小写字母变成大写字母，并返回改变后的字符串
// 注意的是：这里的参数要传字符串变量，不能传字符串常量
char * upper(char *str) {
    // 先保留最初的地址。因为等会str指向的位置会变来变去的。
    char *dest = str;
    // 如果还不是空字符
    while (*str != '\0') {
        // 如果是小写字母
        if (*str >= 'a' && *str <= 'z') {
            // 变为大写字母。小写和大写字母的ASCII值有个固定的差值
            *str -= 'a' - 'A';
        }
        // 遍历下一个字符
        str++;
    }
    // 返回字符串
    return dest;
}
```

## 指向函数的指针

* 函数名就代表着函数的地址

* 指向函数的指针的定义

定义的一般形式：**函数的返回值类型 (*指针变量名)(形式参数1, 形式参数2, ...);**  
形式参数的变量名可以省略，甚至整个形式参数列表都可以省略

```C
#include <stdio.h>
int sum(int a, int b) {
    return a + b;
}
int main(int argc, char *argv[]) {
    // 定义一个指针变量p，指向sum函数
    int (*p)(int a, int b) = sum;
    // 或者 int (*p)(int, int) = sum;
    // 或者 int (*p)() = sum;
    // 利用指针变量p调用函数
    int result = (*p)(1, 3);
    // 或者 int result = p(1, 3);
    printf("%d", result);
    return 0;
}
```

* 应用

```C
#include <stdio.h>

// 减法运算
int minus(int a, int b) {
    return a - b;
}

// 加法运算
int sum(int a, int b) {
    return a + b;
}

// 这个counting函数是用来做a和b之间的计算，至于做加法还是减法运算，由函数的第1个参数决定
void counting( int (*p)(int, int) , int a, int b) {
    int result = p(a, b);
    printf("计算结果为：%d\n", result);
}

int main(int argc, char *argv[]) {
    // 进行加法运算
    counting(sum, 6, 4);
    // 进行减法运算
    counting(minus, 6, 4);
    return 0;
}
```


