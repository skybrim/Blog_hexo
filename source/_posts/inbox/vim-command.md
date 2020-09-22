---
title: VIM 基本命令备忘
date: 2018/12/12
tags: [vim, inbox]
comments: true
---

vim command
<!--more-->

## 退出

```bash
:wq #保存并退出
:q  #退出
:q! #放弃修改强制退出
```

## 插入

```bash
i #当前位置插入
a #当前位置后插入
o #在当前行下面插入新的一行
I #在当前行行首插入 
A #在当前行行尾插入
O #在当前行上面插入新的一行
```

## 常用命令

```bash
:vs #竖分屏 vertical split
:sp #横分屏 split
:% s/from/to/g #全局替换
v #可视化选择，选中当前位置
V #选中当前行
control+v #块状选择
```

##  插入模式

* 删除

```bash
# 插入模式下
ctrl+h #删除一个字符
ctrl+w #删除一个单词
ctrl+u #删除一个行
```

* 切换模式

```bash
ctrl+c #回到命令模式，可能会中断一些插件
ctrl+[ #回到命令模式
gi #快速回到最后一次进入编辑位置，并进入插入模式
```

## 快速移动

```bash
hjkl
w/W #移动到下一个 word/WORD 开头
e/E #移动到下一个 word/WORD 尾部
b/B #移动到上一个 word/WORD 开头
# WORD 指用空格分隔的单词
```

```bash
f+(char) #快速向后跳转到指定字符
F+(char) #快速先前跳转到指定字符
```

```bash
0 #移动到行首
$ #移动到行尾
```

```bash
() #句子之间移动
[] #段落之间移动
```

```bash
gg #文件开头
G  #文件的结尾
```

```bash
ctrl+u #向上翻页
ctrl+d #向后翻页
zz     #把当前行放到中间
```

## 增删改查

* 增

```bash
a/i/o/A/I/O
```

* 删

```bash
# 插入模式 
backspace 
ctrl+h
# normal模式
x 		    #向后删除一个字符，数字+命令
X			#向前删除一个字符
dd			#删除一行，数字+命令
d0			#删除到行首
d$			#删除到行尾
d+(数字)+d  #删除指定数的行
dt+(char)   #删除到指定字符
daw 	    #删除单词以及周围空格
diw		    #删除单词，不删除空格
world worldw world world
```

* 修改

```bash
r(replace)   #替换一个字符
R			 #向后连续替换字符
s(subsitute) #删除一个字符，并进入插入模式
S			 #删除整行，进入插入模式
c(change)    #ct+(char)删除到指定字符，并进入插入模式
cw			 #删除一个单词，并进入插入模式
C			 #和 S 一样
```

* 查

```bash
/+内容  #向后搜索匹配
?+内容  #向前搜索匹配
n       #下一个
N		#上一个
```

## 搜索替换

```bash
:[range]s[ubstitute]/{pattern}/{string}/[flags]
# range 范围，:10,20 表示 10-20 行；% 表示全部
# pattern 替换的文本
# string 替换后的文本
# flags 替换标志. g 全局；c 确认；n 报告匹配到的次数，不替换。
# 支持正则表达式

:% s/abc/def/g
:10,20 s/def/abc/g
```

## 多文件操作

* Buffer

```bash
:ls					   #vim中列出所有 buffer
:e+(FileName) 		   #打开文件 
:b+(FileName)/(number) #跳转buffer
```

* window

```bash
ctrl+w+s / :sp    #水平分隔窗口
ctrl+w+v / :vs	  #竖直分隔窗口
ctrl+w+w/h/j/k/l  #跳转窗口
ctrl+w+H/J/K/L    #移动窗口
ctrl+w+q/c		  #关闭窗口
```  

## Text Object

```bash
[number]<command>[text object]
# number 次数
# command 命令，d(elete),c(hange),y(ank),v(isual)
# text object，文本对象，w 单词，s 句子，p 段落

# eg
c+i+(
v+a+"
# i表示 inner，内部
ci( #删除小括号内容，不含小括号，并进入插入模式)
vi" #选中引号内容
diw #删除单词
# a表示 around，包围
ca( #选中小括号及括号内的内容
```

## 复制粘贴

* normal

```bash
y    #复制
p    #粘贴
d    #剪切
yy   #复制一行
```

* insert

```bash
# 复制到系统剪切板
"+y
    
# 粘贴系统剪切板内容
"+p          
```


## macro

```bash
q + 寄存器名字 		# 开始录制
执行操作
q              		# 退出录制
@ + 寄存器名称 		# 使用宏
v              		# 选中需要修改的地方
:normal @+寄存器名  # 批量操作
```

```bash
# 批量操作
v/V 			#选中需要批量的内容
:normal + 操作  #执行操作
```

## 补全

单词补全
ctrl + n/p

文件路径补全
ctrl + x + f

上下选择
ctrl + n/p

