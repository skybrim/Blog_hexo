---
title: RaspberryPi NAS
date: 2019/12/12
tags: [raspberry-pi, inbox]
comments: true
---

记录一下，用树莓派搭建一个低功耗下载机，挂PT看视频  
硬件：RaspberryPi 3B+、 TF卡（安装树莓派系统）、 1T移动硬盘、 网线
<!--more-->

## 挂载磁盘

插上自己的移动硬盘

```bash
df -h  #查看磁盘
mkdir /share  #创建分享文件目录
chown pi:pi /share  #权限
mount /dev/sda1 /share  #挂载
```

## 自动挂载

开机或者重启的时候，自动挂载移动硬盘  

```bash
sudo vim /etc/fstab

proc            /proc      proc    defaults    0     0
/dev/mmcblk0p1  /boot      vfat    defaults    0     2
/dev/mmcblk0p2  /          ext4    defaults,noatime  0   1
/dev/sda1       /share     ext3    defaults    0     0  # 添加这一行信息
```

## 安装miniDLNA

个人硬件：RT-AC86U、RasbperryPi 3B+  
尝试过 samba 、 FTP 、 dlna 等协议，视频观看使用dlna协议最流畅，其他协议会出现卡顿、音画不同步等各种状况  

```bash
1、apt-get install minidlna #安装
2、vim /etc/minidlna.conf #编辑配置文件
3、/etc/init.d/minidlna restart #重启
4、service minidlna force-reload #重新加载配置
5、/etc/init.d/minidlna status #查看状态
```

## 安装samba

dlna协议不能编辑文件，安装samba，方便修改文件

```bash
sudo apt-get install samba samba-common-bin #安装
vim /etc/samba/smb.conf #修改其配置文件

[share]           #共享文件的名称， 将在网络上以此名称显示
path = /share           #共享文件的路径
valid users = pi        #允许访问的用户
browseable = yes        #允许浏览
public = yes            #共享开放
writable = yes          #可写
sudo /etc/init.d/samba restart #重启 samba
sudo smbpasswd -a pi    #添加samba共享用户
```
