---
title: Hackintosh
comments: true
date: 2020-06-22 11:59:12
tags: Hackintosh
---

OpenCore 0.5.9 引导

采坑记录
<!--more-->

## OpenCore

[OpenCore Desktop 教程](https://dortania.github.io/OpenCore-Desktop-Guide/)

OpenCore 0.5.9 + macOS Catalina 10.15.5

## 写在前面

本篇只是个人安装黑苹果的采坑记录

本人有 MacBook Pro，想给家里组装个台式机，因为没有游戏需求，又习惯 macOS 的开发环境，所以有了这次采坑记录。

建议强需求或者爱折腾的人士安装黑苹果，黑苹果并非主流。

## 配置

| 硬件 | 型号 | 备注 |
| ---- | ---- | ---- |
| CPU | Intel i5-9600k | 带核显可以开启 Sidecar |
| 主板 | MSI Z390-TOMAHAWK| |
| 显卡 | SAPPHIRE RX570 4G 白金 | 免驱 |
| 网卡 | BCM94352HMB | 旧设备，非免驱，需转接卡 |
| 硬盘 | WD SN750 | |

其他硬件，可以根据个人偏好选择

| 硬件 | 型号 | 备注 |
| ---- | ---- | ---- |
| 内存 | 十铨 火神 DDR4 3200 16GB x 2 | |
| 电源 | 振华 G550 |  |
| 机箱 | 追风者 p300 air | 更推荐 p400 air |
| 散热 | 雅浚 D3 | RGB YES! |
| 显示器 | 创维 28U1 | mac尽量上4k显示器 |

## 安装总结

OpenCore 安装黑苹果还是比较简单的，大致分为三步：

1. 制作 USB 启动器
2. 配置 EFI 文件
3. 调优

## USB启动器

### 下载 OS X

* macOS

1. 直接去 Mac App Store 下载系统
   
2. 格式化 U 盘

    使用自带的磁盘工具应用(实用工具-磁盘工具)
    

* Windows

[OpenCore Windows Creating the USB](https://dortania.github.io/OpenCore-Desktop-Guide/installer-guide/winblows-install.html)

### 格式化



## EFI

## 调优