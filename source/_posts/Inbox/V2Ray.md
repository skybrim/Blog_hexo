---
title: V2Ray
date: 2019/12/12
tags: V2Ray
comments: true
---

Project V 是一个工具集合，它可以帮助你打造专属的基础通信网络。Project V 的核心工具称为V2Ray，其主要负责网络协议和功能的实现，与其它 Project V 通信。V2Ray 可以单独运行，也可以和其它工具配合，以提供简便的操作流程。
<!--more-->

## 开始搭建

### 一、购买VPS

* 服务商

根据个人需求，找一些适合自己的VPS服务商。

### 二、部署服务器

* 服务器选择依据  
    > 速度  
    > 价格  
    > 流量  
    > 系统（这个一般来说没问题）  
    > 个人其他需求  

* V2Ray 在Linux系统中可用版本：  
    > Linux 2.6.23 及之后版本（x86 / amd64 / arm / arm64 / mips64 / mips）。
    > 包括但不限于 Debian 7 / 8、Ubuntu 12.04 / 14.04 及后续版本、CentOS 6 / 7、Arch Linux；

* 账号密码  
    部署完服务器后，记录一下自己的服务器的 IP Address 、 Username 、 Password

### 三、连接服务器

* MacOS系统
MacOS中，直接打开终端（Terminal），输入命令 ：

    ```bash
    ssh root@45.00.00.00
    拼写方式是 ssh+空格+Username(root)+@+IP
    加号不用输入，IP 和 Username 就是你刚刚复制的，一般 Username 是 root
    ```

    显示如下，说明可以正常连接服务器，输入yes，并回车。  
    ![1](https://raw.githubusercontent.com/skybrim/AllImages/master/20190314172310.png)

    接下来输入密码。选中密码，拷贝，选中终端，cmd+V，不显示密码或者**是正常情况，直接回车即可。  
    ![2](https://raw.githubusercontent.com/skybrim/AllImages/master/20190314150746.png)

    进入下面这个界面，说明成功连接上VPS。  
    ![3](https://raw.githubusercontent.com/skybrim/AllImages/master/20190314150916.png)

* Windows系统

    使用一些SSH工具即可。如：Putty。

### 四、服务器安装V2Ray

* 执行安装脚本

  ```bash
    bash <(curl -L -s https://install.direct/go.sh)
  ```

    如果之前系统选择正确的话，直接执行下面的脚本即可，官方提供。  
    此脚本会配置自动运行脚本。自动运行脚本会在系统重启之后，自动运行 V2Ray。  
    这个脚本会自动检测有没有安装过 V2Ray，如果没有，则进行完整的安装和配置；如果之前安装过 V2Ray，则只更新 V2Ray 二进制程序而不更新配置。  
    脚本安装完毕，会自动配置好一套，如果不需要搭建酸酸乳，直接使用即可。  
    复制并保存 PORT 、 UUID  
    ![4](https://raw.githubusercontent.com/skybrim/AllImages/master/20190314154947.png)

* 配置文件

    ```bash
    vim /etc/v2ray/config.json
    ```

    进入配置文件界面，按一下 i 键，开始编辑，下面给一套配置好的格式，照着编辑即可

    注意，必须是在英文输入法状态下编辑，json格式一定要正确，否则不会运行。

    ```json
    "inbounds": [{
        "port": 443,// vmess 协议服务器监听端口
        "protocol": "vmess",
        "settings": {
        "clients": [
            {
            "id": "b831381d-6324-4d53-ad4f-8cda48b30811", //UUID
            "level": 1,
            "alterId": 64
            }
        ]
        }
    },{
        "port": 444,// SS 协议服务端监听端口
        "protocol": "shadowsocks",
        "settings": {
            "method": "aes-128-gcm",
            "password": "password" //密码
        }
    }],
    "outbounds": [{
        "protocol": "freedom",
        "settings": {}
        },{
        "protocol": "blackhole",
        "settings": {},
        "tag": "blocked"
    }],
    "routing": {
        "rules": [{
            "type": "field",
            "ip": ["geoip:private"],
            "outboundTag": "blocked"
        }]
    }
    ```

* 开启服务
执行下面命令，服务端配置完成。

    ```bash
    service v2ray start
    ```

    每次更改完配置文件，都需要重新开启服务。  
    其他常用命令：

    ```bash
    //服务停止
    service v2ray stop
    //服务状态
    service v2ray status
    //服务重启
    service v2ray restart
    ```

### 五、客户端配置

* iOS

    Kitsunebi

    > Kitsunebi 是一个基于 V2Ray 核心的 iOS 应用。它可以创建基于 VMess 或者 Shadowsocks 的 VPN 连接。Kitsunebi 支持导入和导出与 V2Ray 兼容的 JSON 配置。

    i2Ray

    > i2Ray 是另一款基于 V2Ray 核心的iOS应用。界面简洁易用，适合新手用户使用。同时兼容Shadowrocket和Quantumult格式的规则导入。

    Shadowrocket

    > Shadowrocket 是一个通用的 iOS VPN 应用，它支持众多协议，如 Shadowsocks、VMess、SSR 等。

上面三款国区并未上线，大家可以去别的区购买，关键词 Gift Card。

* Android

    BifrostV

    >BifrostV 是一个基于 V2Ray 内核的 Android 应用，它支持 VMess、Shadowsocks、Socks 协议。
    >下载地址： [APK Pure](https://apkpure.com/bifrostv/com.github.dawndiy.bifrostv)

    V2RayNG

    >V2RayNG 是一个基于 V2Ray 内核的 Android 应用，它可以创建基于 VMess 的 VPN 连接。
    >下载地址：[GitHub](https://github.com/2dust/v2rayNG)

* Windows

    V2RayN

    >V2RayN 是一个基于 V2Ray 内核的 Windows 客户端。
    >下载 v2rayN-Core.zip：[GitHub](https://github.com/2dust/v2rayN/releases)

    shadowsocks-windows

    >Windows下的SSR客户端
    >下载地址：[GitHub](https://github.com/shadowsocks/shadowsocks-windows/releases)

* MacOS

    V2RayX

    >V2RayX 是一个基于 V2Ray 内核的 Mac OS X 客户端。用户可以通过界面生成配置文件，并且可以手动更新 V2Ray 内核。V2RayX 还可以配置系统代理。
    >下载地址：[GitHub](https://github.com/Cenmrev/V2RayX/releases)
