---
title: FFmpeg 常用命令
date: 2019/6/28
tags: ffmpeg
comments: true
---

ffmpeg command
<!--more-->

## 基本信息查询

![基本信息查询](https://cdn.jsdelivr.net/gh/skybrim/AllImages@dev/%E5%9F%BA%E6%9C%AC%E4%BF%A1%E6%81%AF%E6%9F%A5%E8%AF%A2%E5%91%BD%E4%BB%A4.jpg)

## 录制

* 查看支持的采集设备编号列表

```bash
ffmpeg -f avfoundation -list_devices true -i ""
```

* 视频录制

```bash
# -f : 指定使用 AVfoundation 采集数据
# -i : 指定从哪里采集数据，文件索引号  0 摄像头 1 屏幕
# -r : 指定帧率
# out.yuv : 保存的文件名，yuv 无压缩的数据
ffmpeg -f avfoundation -i 1 -r 30 out.yuv
```

* 视频播放

```bash
# -s : size，视频分辨率
# -pix_fmt : 像素格式，默认 yuv420p
ffplay -s 4096x2304 -pix_fmt uyvy422 out.yuv
```

* 音频录制

```bash
# -i : 指定采集设备，冒号前面是视频采集设置，冒号后面是音频采集设备
ffmpeg -f avfoundation -i :0 out.wav
```

* 音频播放

```bash
ffplay out.wav
```

## 分解与复用

* 转换 复用

```bash
## 直接转格式 demuxer + muxer
# -vcodec copy : 视频编码处理方式，copy 指不改变
# -acodec copy : 音频编码处理方式，copy 指不改变
ffmpeg -i input.mp4 -vcodec copy -acodec copy out.flv
```

* 分解

```bash
## 抽取视频
# -an : 不要音频
ffmpeg -i input.mp4 -an -vcodec copy out.h264

## 抽取音频
# -vn :  不要音频
ffmpeg -i input.mp4 -acodec copy -vn out.aac
```

## 处理原始数据

* 抽取视频原始数据 yuv

```bash
# -c:v rawvideo : 视频编码。rawvideo 原始视频
# -pix_fmt : 像素格式
ffmpeg -i input.mp4 -an -c:v rawvideo -pix_fmt yuv420p out.yuv
```

* 抽取音频原始数据 pcm

```bash
# -ar : 音频采样率 44.1k
# -ac 2 : channel，2 是双声道
# -f s16le : 数据存储格式； s : 有符号； 16 : 16位表示； le 小
ffmpeg -i input.mp4 -vn -ar 44100 -ac2 -f s16le out.pcm

## 播放原始音频
# 指定原始音频播放参数
ffplay -ar 44100 -ac 2 -f s16le out.pcm
```

## 剪裁与合并

* 裁剪

```bash
# -ss : 视频裁剪起始点
# -t : 裁剪时长，单位秒
ffmpeg -i input.mp4 -ss 00:00:00 -t 10 out.ts
```

* 合并

```bash
# inputs.txt : 想要合并的文件的文件名列表，内容格式为 file 'filename.xxx'
ffmpeg -f concat -i inputs.txt out.flv
# 拼接 MP4
```

## 图片与视频互转

* 图片转视频

```bash
ffmpeg -i image-%3d.jpeg out.mp4
```

* 视频转图片

```bash
# -r : 指定转换图片帧率。 1 每秒转出1张
# -f : 指定格式 image2
ffmpeg -i input.flv -r 1 -f image2 image-%3d.jpeg
```

## 直播相关

* 推流

```bash
# -re : Read input at native frame rate. Mainly used to simulate a grab device, or live input stream (e.g. when reading from a file). Should not be used with actual grab devices or live input streams (where it can cause packet loss). By default ffmpeg attempts to read the input(s) as fast as possible. This option will slow down the reading of the input(s) to the native frame rate of the input(s). It is useful for real-time output (e.g. live streaming).
# -c : 音视频编解码
# -f : 格式
# 推向的服务器地址
ffmpeg -re -i out.mp4 -c copy -f flv rtmpt://server/live/streamName
```

* 拉流

```bash
ffmpeg -i rtmp://server/live/streamName -c copy dump.flv
```

## 滤镜

* 视频裁剪

```bash
# -vf : vf,使用视频滤镜；crop 滤镜名；crop=in_w-200:in_h-200，滤镜参数，长宽减200；默认原点是中心
# crop 格式 : crop=xxxx_w:xxxx_h:x:y
# -c:v : 视频编码器，libx264
# -c:a : 音频编码器，copy
ffmpeg -i input.mp4 -vf crop=input_w-200:input_h-200 -c:v libx264 -c:a copy out.mp4
```

* 添加水印

```bash

```
