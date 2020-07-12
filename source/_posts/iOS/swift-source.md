---
title: swift-source
comments: true
date: 2020-07-07 14:23:18
tags: [iOS, swift]
---

编译 Swift 源码
<!--more-->

## 编译

system macOS 10.15.5
Xcode 11.5

如果下面的命令，出现了一些 python 的错误，这是因为改动了系统的 python 环境。

请切换回系统自带的 python，或者虚拟一个新的 python2.7 的环境

```bash
# 编译工具
brew install cmake ninja

# 下载源码
mkdir swift-source
cd swift-source
git clone https://github.com/apple/swift.git

# 切换到最新的 release tag （写文是 5.2.4）
cd swift
git checkout swift-5.2.4-RELEASE
./utils/update-checkout --tag swift-5.2.4-RELEASE --clone


# 编译的最后一步是生成文档，如果没有安装 Sphinx 会报错
# 根据个人的 python 配置情况安装一下 Sphinx
# 我使用的是 conda 虚拟 python 环境
conda install Sphinx

# 编译 
# 编译好的文件放在 swift-source/build/ 文件夹下
# 根据电脑不同，编译时间大概在 1~2 小时
# 参数
# -x 使用 xcode 编译，生成一个 xcodeproj 文件
# -r --release-debuginfo
# --debug-swift-stdlib 对标准库开启 Debug 模式
./utils/build-script -x -r --debug-swift-stdlib 
```

## 运行

1. 首先运行 cmark 工程
2. 再运行 swift 工程

## 更新源码

```bash
cd swift-source
./swift/utils/update-checkout
./swift/utils/build-script -x -r --debug-swift-stdlib 
```

## 查找源码

* 如果知道源码具体位置，直接打开对应的文件查看
* 全局搜索 ```public func xxx```