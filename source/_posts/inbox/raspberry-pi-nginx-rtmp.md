---
title: 树莓派 + nginx + rtmp
date: 2019/12/12
tags: [raspberrypi, inbox]
comments: true
---

RaspberryPi 折腾记录
<!--more-->

## 安装 nginx + rtmp

由于需要安装 rtmp，所以选择使用源码编译的方式进行安装

* 升级

```bash
sudo apt-get update
```

* 安装gcc g++依赖库

```bash
sudo apt-get install build-essential
sudo apt-get install libtool
```

* 安装pcre依赖库

```bash
sudo apt-get install libpcre3 libpcre3-dev
```

* 安装zlib依赖库

```bash
sudo apt-get install zlib1g-dev
```

* 安装SSL依赖库

```bash
sudo apt-get install openssl libssl-dev
```

* 下载 nginx

```bash
wget http://nginx.org/download/nginx-1.17.4.tar.gz
tar -zxvf nginx-1.17.4.tar.gz
```

* 下载 rtmp

```bash
git clone https://github.com/arut/nginx-rtmp-module.git
```

* 安装

```bash
cd nginx-1.17.4
./configure --prefix=/usr/local/nginx --add-module=/root/nginx-rtmp-module --with-http_ssl_module --with-cc-opt="-Wno-error"
make
sudo make install
```

* 配置

```bash
cd /usr/local/nginx/conf/
sudo vim nginx.conf

# 配置 rtmp server
rtmp {
    server{
        # 制定服务端口
        listen 1935;
        chunk_size 4000;
        # 制定流应用
        application live
        {
            live on;
            record off;
            allow play all;
        }
    }
}
```

* 启动

```bash
sudo ln -s /usr/local/nginx/sbin/nginx /usr/local/sbin/
nginx
```

* 验证

```bash
netstat -antup | grep 1935
```

## 推流

使用 ffmpeg 推流

* 安装 ffmpeg

```bash
sudo apt-get install ffmpeg
```

* 推流

```bash
ffmpeg -i http://pcyf.qing.mgtv.com/nn_live/nn_x64/cz05NThjNDc5ZjE2ZWNjMjcxNDU1MzQwNmExMWY1NWI5ZSZlcz0xNTcwMzgzOTExJnV1aWQ9MWZmYjQ5ZTdmNGRhZTJmZjE1YjYyY2Y3ZDNkOWE1NzAtNzQxOWZiYmImdj0yJmFzPTAmY2RuZXhfaWQ9eWZfcGNfbGl2ZQ,,/KLGMPP360.flv?timezone=8 -c:a copy -c:v copy -f flv rtmp://localhost:1935/live/room
```

## 播放

使用 ffplay 播放

```bash
ffplay rtmp://RaspberryPi_IP:1935/live/room
```
