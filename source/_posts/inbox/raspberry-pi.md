---
title: Raspbery Pi 初始化
date: 2019/12/12
tags: [raspberry-pi, inbox]
comments: true
---

<!--more-->

## 硬件

树莓派 3B+

16GB 的内存卡


## 下载树莓派镜像

[树莓派系统](https://shumeipai.nxez.com/download#os)


## 写入镜像

我使用的是 balenaEtcher 工具写入镜像

写入镜像后，重新插拔一下内存卡

1. 新建一个名为 ssh 的文件，无任何后缀
2. 新建一个 wpa_supplicant.conf 文件，内容如下

```bash
country=CN
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
 
network={
    ssid="WiFi-A"
    psk="12345678"
    key_mgmt=WPA-PSK
    priority=1
}
```

## 连接

```bash
ssh pi@raspberrypi.local
password: raspberry
```