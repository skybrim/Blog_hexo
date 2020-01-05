---
title: Raspbery Pi 初始化
date: 2019/12/12
tags: RaspberryPi
comments: true
---

RaspberryPi 折腾记录
<!--more-->

## 写入树莓派镜像

>下载树莓派最新的系统镜像
>[http://downloads.raspberrypi.org/raspbian_latest](http://downloads.raspberrypi.org/raspbian_latest)
>读卡器连TF卡到电脑，使用 Etcher 烧录镜像，显示校验成功
>创建一个文件，ssh，没有后缀，放到 tf 卡中
>插入树莓派，树莓派网线连上路由器，开机

## 基础

* ssh连接树莓派

```bash
ssh pi@raspberrypi.local
password: raspberry
```

* 更新

```bash
sudo apt-get update
```

* 安装 vim

```bash
sudo apt-get install vim
```

* 设置 wifi

```bash
vim /etc/wpa_supplicant/wpa_supplicant.conf
network={
    ssid="wifi_name"
    psk="wifi_pwd"
    priority=level数字
}
```

* 更改国家

出现 5g wifi 无法连接的问题，需要更改树莓派的国家地区

```bash
sudo raspi-config
```

* 重启

```bash
sudo reboot
```
