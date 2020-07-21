---
title: FFmpeg 初级开发内容
date: 2019/6/28
tags: ffmpeg
comments: true
---

ffmpeg primary
<!--more-->

## FFmpeg 日志的使用及目录操作

### 开启日志
* #include <libavutil/log.h>

* 设置日志级别  
avi\_log\_set\_level(AV\_LOG\_DEBUG)  
常见日志级别：  
AV\_LOG\_ERROR  
AV\_LOG\_WARNING  
AV\_LOG\_INFO  
AV\_LOG\_DEBUG  

* av\_log(NULL,AV\_LOG\_INFO,"...%s\n", op) 

### 操作文件

* #include <libavformat/avformat.h>  
指定库文件位置： pkg-config --libs libavformat  

* 删除  
avpriv\_io\_delete("1.txt");  

* 重命名   
avpriv\_io\_move("111.txt", "222.txt");  

### 操作目录

* #include <libavformat/avformat.h>  

* 打开目录  
avio\_open\_dir()

* 读  
avio\_read\_dir()  

* 关闭
avio\_close\_dir()  

* 重要结构体  
操作目录的上下文 AVIODirContext  
目录项 AVIODirEntry  

### 示例
```c
#include <libavutil/log.h>
#include <libavformat/avformat.h>

int main(int argc, char *argv[]) {
	av_log_set_level(AV_LOG_INFO);
	
	int ret;

	//操作目录
	AVIODirContext *ctx = NULL;
	AVIODirEntry *entry = NULL;
	ret = avio_open_dir(&ctx, "./", NULL);
	if (ret < 0) {
		av_log(NULL, AV_LOG_ERROR, "Cant open dir:%s\n", av_err2str(ret));
		return -1;
	}
	while(1) {
		ret = avio_read_dir(ctx, &entry);
		if (ret < 0) {
			av_log(NULL, AV_LOG_ERROR, "Cant read dir:%s.\n", av_err2str(ret));
			//报错后，跳转到__fail 标记，释放 ctx
			goto __fail;
		}
		if (!entry) {
			break;
		}
		av_log(NULL, AV_LOG_INFO, "%12"PRId64" %s \n", entry->size, entry->name);
		//释放 entry
		avio_free_directory_entry(&entry);
	}
__fail;
	avio_close_dir(&ctx);
	
	// move rename
	ret = avpriv_io_move("111.txt", "222.txt");
	if (ret < 0) {
		av_log(NULL, AV_LOG_ERROR, "Failed to rename.\n");
		return -1;
	}
	av_log(NULL, AV_LOG_INFO, "Success to rename.\n");

	// delete
	ret = avpriv_io_delete("./222.txt");
	if (ret < 0) {
		av_log(NULL, AV_LOG_ERROR, "Failed to delete.\n");
		return -1;
	}
	av_log(NULL, AV_LOG_INFO, "Success to delete.\n");

	return 0;
}
```

## FFmpeg 基本概念以及常用结构体

* 概念  
多媒体文件本质是一个容器  
在容器里有很多流（Stream/Track）  
每种流是由不同的编码器编码的  
从流中读取的数据，称为包  
在一个包中包含着一个或多个帧，包是压缩后的数据  

* 常用结构体  
AVFormatContext  
AVStream  
AVPacket  

* FFmpeg操作流数据的基本步骤  
解复用 --> 获取流 --> 读取数据包 --> 操作 --> 释放资源  

## 对复用/解复用及流操作的各种实践

* 打印音视频信息（meta）  


## FFmpeg 代码结构

![FFmpeg-代码结构](https://raw.githubusercontent.com/skybrim/AllImages/dev/FFmpeg%E4%BB%A3%E7%A0%81%E7%BB%93%E6%9E%84.jpg)
