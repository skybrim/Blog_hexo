---
title: C语言字符和字符串
date: 2019/10/13
tags: c
comments: true
---

char string
<!--more-->

## 字符 char

char的取值范围是：ASCII码字符 或者 -128~127的整数  
char只能存储一个字符。汉字是两个字符，所以 char 不能存储单个汉字  

* 输入输出

```C
//输出 putchar一次只能输出一个字符，而printf可以同时输出多个字符
putchar(65); // A
putchar('A'); // A
int a = 65;
putchar(a); // A
//输入
char c;
c = getchar();
```

## 字符串 char *

* 概况
C语言中没有String这种类型，用字符数组来存储字符串。  
字符串可以看做是一个特殊的字符数组，为了跟普通的字符数组区分开来，应该在字符串的尾部添加了一个结束标志'\0'  

* 初始化

```C
//初始化方式 1
char a[3] = {'a', 'b', '\0'};
//初始化方式 2
char b[3];
b[0] = 'a';
b[1] = 'b';
b[2] = '\0';
//初始化方式 3
char c[3] = "ab";
//初始化方式 4
char d[] = "ab";
//初始化方式 5
char e[20] = "ab";
```

3、4、5初始化，系统会自动在字符串尾部加上一个\0结束符  

* 输出

```C
//1
printf("%s", a);
//2 puts函数输出一个字符串后会自动换行
puts(a);
puts("abc");
```

* 输入

```C
//1
char a[10];
scanf("%s", a);
//2
char b[10];
gets(b);
```

* 字符串数组

```C
char names[2][10] = { {'J','a','y','\0'}, {'J','i','m','\0'} };
char names2[2][10] = { {"Jay"}, {"Jim"} };
char names3[2][10] = { "Jay", "Jim" };
```

* 字符串处理函数
  + strlen  
  测量字符串的字符个数，不包括 \0  

  ```C
  int size = strlen("ab"); // 长度为2
  char s1[] = "abc";
  int size1 = strlen(s1); // 长度为3
  char s2[] = {'a', 'b', '\0', 'c', 'd', 'e', '\0'};
  int size2 = strlen(s2); // 长度为2
  ```

  + strcpy  
  strcpy函数会将右边的"lmj"字符串拷贝到字符数组s中。从s的首地址开始，逐个字符拷贝，直到拷贝到\0为止。当然，在s的尾部肯定会保留一个\0。  
  假设右边的字符串中有好几个\0，strcpy函数只会拷贝第1个\0之前的内容，后面的内容不拷贝  

  ```C
  char s[10];
  strcpy(s, "abc");
  ```

  + strcat  
  strcat函数会将右边的"de"字符串拼接到s1的尾部，最后s1的内容就变成了"abcde"
  strcat函数会从s1的第1个\0字符开始连接字符串，s1的第1个\0字符会被右边的字符串覆盖，连接完毕后在s1的尾部保留一个\0  

  ```C
  char s1[30] = "abc";
  strcat(s1, "de");
  ```

  + strcmp  
  这个函数可以用来比较2个字符串的大小  
  两个字符串从左至右逐个字符比较（按照字符的ASCII码值的大小），直到字符不相同或者遇见'\0'为止。如果全部字符都相同，则返回值为0。如果不相同，则返回两个字符串中第一个不相同的字符ASCII码值的差。即字符串1大于字符串2时函数返回值为正，否则为负  

  ```C
  char s1[] = "abc";
  char s2[] = "abc";
  char s3[] = "aBc";
  char s4[] = "ccb";
  printf("%d, %d, %d", strcmp(s1, s2), strcmp(s1, s3), strcmp(s1, s4));
  // 0, 32, -2
  ```

