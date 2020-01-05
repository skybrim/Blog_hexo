---
title: Kindle电子书转PDF
date: 2019/12/12
tags: kindle
comments: true
---

没有 Kindle，也不是自购的 Kindle 电子书就不要进来看了。  
<!--more-->

## 初衷

想把自己购买的 Kindle 电子书转成 pdf ，在 iPad 的 Notability 上看，配合 apple pencil 作笔记会非常方便。

## 准备工具

* Kindle 电子书  
打开亚马逊的网站，我的账户 - 管理我的内容和设备 - 我的内容  
选择需要转换的电子书，点击 ... 按钮  
选择-通过电脑下载USB传输  
顺便去 我的设备 ，记录下自己的 kindle 的序列号

* Calibre  
brew cask install Calibre  

* DeDRM_tools  
https://github.com/apprenticeharper/DeDRM_tools/releases

## 转换

* 安装插件
打开 Calibre  
偏好设置 - 高级选项 - 插件  
选择 - 从文件加载插件  
选择刚才下载好的 DeDrm_tools

* 设置插件  
双击安装好的插件  
选择第一项 elnk Kindle ebooks  
点击 + 按钮，输入你的 Kindle 的序列号  
**注意，序列号需要去除空格**

* 添加书籍到 Calibre  
根据自己的需要，转换相应的格式
