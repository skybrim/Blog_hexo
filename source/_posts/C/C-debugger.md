---
title: C语言调试器
date: 2019/10/13
tags: C
comments: true
---

debug
<!--more-->

## 流程

* 需要使用 -g 参数，编译出带调试信息的程序

* 调试信息包含：指令地址、对应源码及行号

* 指令完成之后，回调调试器


## 常用命令

![lldb/gdb-调试命令](https://raw.githubusercontent.com/skybrim/AllImages/dev/gdb%3Alldb-command.jpg)

## 开始调试

```bash
# 加载到调试器
lldb xxxx
# 退出
quit
```

