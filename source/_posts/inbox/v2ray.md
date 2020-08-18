---
title: Project V
date: 2018/12/12
tags: [proxy, inbox]
comments: true
---

Project V 
<!--more-->

## 方案

TCP + TLS 分流器


## 服务端

### VPS

根据个人需求，找一些适合自己的VPS服务商。

### 部署

* Debian 10

* 自行设置 ssh 相关，或者直接使用服务商的后台

### 域名

根据自身条件，购买一个域名（国内需备案），并在域名商后台添加一个 A 记录指向自己的 vps 地址，dns 服务可能需要一点时间才能生效

下文使用 domain.me 替代

### 连接

* MacOS系统
    
MacOS中，直接打开终端（Terminal），输入命令 ：

```bash
ssh username@xxx.xxx.x.x
```

* Windows系统

使用一些SSH工具即可。如：Putty。

### vim

中间有一些需要使用 vim 编辑，简单说几个命令

i 进入编辑模式

esc 进入普通模式 

:wq 编辑好后，在普通模式下，保存退出

### v2ray

* 执行安装脚本

```bash
bash <(curl -L -s https://install.direct/go.sh)
```

![4](https://cdn.jsdelivr.net/gh/skybrim/AllImages@dev/20190314154947.png)

* 配置文件

```bash
vim /etc/v2ray/config.json
```

```json
{
    "inbounds": [
        {
            "protocol": "vmess",
            "listen": "127.0.0.1",
            "port": 40001, /* 自定，记得删除注释*/
            "settings": {
                "clients": [
                    {
                        "id": "f2435e5c-9ad9-4367-836a-8341117d0a5f" /* 自定，记得删除注释*/
                    }
                ]
            },
            "streamSettings": {
                "network": "tcp"
            }
        }
    ],
    "outbounds": [
        {
            "protocol": "freedom"
        }
    ]
}
```

### TLS 分流器

* 安装脚本

```bash
bash <(curl -L -s https://raw.githubusercontent.com/liberal-boy/tls-shunt-proxy/master/dist/install.sh)
```

* 配置文件

```bash
vim /etc/tls-shunt-proxy/config.yaml
```

```
listen: 0.0.0.0:443
vhosts:

# 将 domain.me 改为你的域名
- name: domain.me
    tlsoffloading: true
    managedcert: true
    alpn: h2,http/1.1
    
    # 如果不需要兼容 tls12, 可改为 tls13
    protocols: tls12,tls13
    http:
    handler: fileServer

    # /var/www/html 是静态网站目录
    args: /var/www/html
    default:
    handler: proxyPass
    args: 127.0.0.1:40001
```

### 启动服务

```bash
systemctl restart tls-shunt-proxy
systemctl restart v2ray
```


## 客户端

### 配置

```json
{
    "inbounds": [
        {
            "port": 1080,
            "listen": "127.0.0.1",
            "protocol": "socks",
            "sniffing": {
                "enabled": true,
                "destOverride": ["http", "tls"]
            },
            "settings": {
                "auth": "noauth"  //socks的认证设置，noauth 代表不认证，由于 socks 通常在客户端使用，所以这里不认证
            }
        }
    ],
    "outbounds": [
        {
            "protocol": "vmess",
            "settings": {
                "vnext": [
                    {
                        "address": "domain.me",
                        "port": 443,
                        "users": [
                            {
                                "id": "f2435e5c-9ad9-4367-836a-8341117d0a5f",
                                "security": "none"
                            }
                        ]
                    }
                ]
            },
            "streamSettings": {
                "network": "tcp",
                "security": "tls"
            }
        }
    ]
}    
```

### GUI 客户端

* iOS

    国区并未上线，去别的区购买，关键词 Gift Card。

    - Kitsunebi

    - i2Ray

    - Shadowrocket

* Android

    - [BifrostV](https://apkpure.com/bifrostv/com.github.dawndiy.bifrostv)

    - [V2RayNG](https://github.com/2dust/v2rayNG)

* Windows

    - [V2RayN](https://github.com/2dust/v2rayN/releases)

    - [shadowsocks-windows](https://github.com/shadowsocks/shadowsocks-windows/releases)

* MacOS

    - [V2RayX](https://github.com/Cenmrev/V2RayX/releases)
