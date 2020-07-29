---
title: C语言编译器
date: 2019/10/13
tags: c
comments: true
---

编译
<!--more-->

## 编译命令

```bash
# gcc linux下的编译器，clang mac下的编译器
# -g 输出文件中的调试信息
# -O 对输出文件做指令优化 -O1 不优化 -O2 优化
# -o 输出文件名
# -I 指定头文件位置
# -L 指定库文件位置 
# -l 指定使用哪个库
gcc/clang -g -O2 -o test test.c -I... -L... -l
```

## 编译过程

* 预编译

* 编译

```bash
clang -c xxx.c
```

* 链接，动态链接/静态链接  
静态链接，将其他系统库、第三方库链接到一起  
动态链接，运行时，加载其他库  

```bash
clang -g -o xxx yyy.o -L . -lmylib
```

