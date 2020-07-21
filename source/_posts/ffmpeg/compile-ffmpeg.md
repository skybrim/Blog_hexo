---
title: 编译 FFmpeg 源码
date: 2019/6/28
tags: ffmpeg
comments: true
---

ffmpeg compile
<!--more-->

* 下载源码

```bash
git clone https://git.ffmpeg.org/ffmpeg.git
```

* 依赖库

安装依赖库

```bash
brew install pkg-config
brew install yasm
brew install fdk-aac
brew install speex
brew install x264
brew install x265
brew install sdl2
```

* 切换到 release 分支

个人偏好，在稳定版本下学习

* 修改 config.asm

```c
#define CONFIG_FFPLAY 1
```

* 设置编译参数

```bash
./configure --prefix=/usr/local/ffmpeg
                  --enable-gpl
                  --enable-nonfree
                  --enable-libfdk-aac
                  --enable-libx264
                  --enable-libx265
                  --enable-filter=delogo
                  --enable-debug
                  --disable-optimizations
                  --enable-libspeex
                  --enable-videotoolbox
                  --enable-shared
                  --enable-pthreads
                  --enable-version3
                  --enable-hardcoded-tables
                  --cc=clang
                  --host-cflags=
                  --host-ldflags=
```

* 编译

```bash
sudo make && make install
```

* 在 zsh 中添加路径

```bash
export PATH=$PATH:/usr/local/ffmpeg/bin
```
