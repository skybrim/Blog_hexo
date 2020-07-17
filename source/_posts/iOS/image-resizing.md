---
title: 图片渲染
date: 2019/1/1
tags: iOS
comments: true
---

image resizing
<!--more-->

## 场景 1：一般网络图片渲染 eg:YYWebImage

### 下载

* 从网络下载图片源数据，默认放入内存和磁盘缓存中
* 异步解码，解码后的数据放入内存缓存中
* 回调主线程渲染图片
* 内部维护磁盘和内存的cache，支持设置定时过期清理，内存cache的上限等

### 渲染

* 从内存中查找图片数据，如果有并且已经解码，直接返回数据，如果没有解码，异步解码缓存内存后返回
* 内存中未查找到图片数据，从磁盘查找，磁盘查找到后，加载图片源数据到内存，异步解码缓存内存后返回，如果没有去网络下载图片。走上面的流程。

### 分析

* 这样滴流程解决了UIImage imageNamed这种加载一定在主线程解码图片的问题，异步加载，避免了主线程阻塞。
* 通过缓存内存方式，避开了频繁的磁盘IO
* 通过缓存解码后的图片数据，避开了频繁解码的CPU消耗。

## 场景 2：超大图片渲染到较小的 view 上

使用的是 ImageIO 框架的 下采样（downsampling）  
生成对应图像的缩略图，使得图像符合显示区域的大小  
如果是网络图片，用 SDWebImage or YYWebImage，选择不解码，直接下载到本地，否则按照通常的网络图片加载策略，几张图片就会导致内存警告，并闪退。  

### Code

```Swift
func downsampleImage(at URL:NSURL, maxSize:Float) -> UIImage {
    let sourceOptions = [kCGImageSourceShouldCache:false] as CFDictionary
    let source = CGImageSourceCreateWithURL(URL as CFURL, sourceOptions)!
    let downsampleOptions = [kCGImageSourceCreateThumbnailFromImageAlways:true,
                             kCGImageSourceThumbnailMaxPixelSize:maxSize
                             kCGImageSourceShouldCacheImmediately:true,
                             kCGImageSourceCreateThumbnailWithTransform:true,
                             ] as CFDictionary
    let downsampledImage = CGImageSourceCreateThumbnailAtIndex(source, 0, downsampleOptions)!
    return UIImage(cgImage: downsampledImage)
}
```

### warning

* 设置kCGImageSourceShouldCache为false，避免缓存解码后的数据，64位设置上默认是开启缓存的

* 设置kCGImageSourceShouldCacheImmediately为true，避免在需要渲染的时候才做解码，默认选项是false

* 解码&渲染图片，异步加载

### 优势

* 内存占用小  
直接使用 UIImage(contentsOfFile: url.path) 加载图片，堆内存的使用是根据图片的像素来的。  
但是，实际上 iPhone 的屏幕并不能显示那么多像素。  
通过下采样，可以使图片渲染根据 iPhone 的屏幕分辨率来，即保证了清晰度，又不会过于占用内存。  

* 速度快  
渲染像素减少，渲染速度快  

* API 简洁  
Core Graphics 的 API 较复杂

## 场景 3：微信、微博长图详情显示

CATiledLayer。  
原理是分片渲染，滑动时通过指定目标位置，通过映射原图指定位置的部分图片数据解码渲染  
如果是网络图片，也要想 场景2 一样，不解码先下载到本地。  
