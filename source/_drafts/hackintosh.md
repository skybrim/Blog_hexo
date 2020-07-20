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

[OpenCore Desktop 教程](https://dortania.github.io/OpenCore-Install-Guide/)

OpenCore 0.5.9 + macOS Catalina 10.15.5

## 写在前面

文章只是个人使用 OpenCore 安装黑苹果的采坑记录，没有利益相关。

本人有 MacBook Pro，所以是在 macOS 中进行各种配置；windows的用户区别主要在于制作 U盘启动器，需要自己去 OpenCore 官网采坑。

**只推荐爱折腾的人尝试黑苹果。**

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

电脑系统的通过主板的启动器(BIOS/UEFI)，唤醒操作系统。

黑苹果相当于，自己配置一套 EFI，欺骗macOS并引导其工作。

主要就是两大步骤：

1. 制作 U盘安装工具
2. 配置 EFI 文件

## U盘安装工具

### 下载 OS X

* macOS

1. 直接去 Mac App Store 下载系统
   
2. 格式化 U 盘

    1. macOS 自带的 Disk Utility (实用工具-磁盘工具)
    2. 左上角 View-Show All Device
    3. 选择自己的U盘的最外层的目录
    4. Erase(抹除)-> Name: MyVolume; Format: Mac OS Extended(Journaled); Scheme: GUID Partition Map
    5. Terminal（终端）- ```sudo /Applications/Install\ macOS\ Catalina.app/Contents/Resources/createinstallmedia --volume /Volumes/MyVolume```
    
* Windows

[OpenCore Windows Creating the USB](https://dortania.github.io/OpenCore-Install-Guide/installer-guide/winblows-install.html)

## EFI

### 加载 EFI

macOS 下比较简单，直接使用 diskutil 命令就可以
```bash
diskutil list
sudo diskutil mount xxx
```

### Base OpenCore Files

[OpenCorePkg](https://github.com/acidanthera/OpenCorePkg/releases/)

### config.plist

### ACPI

## 调优