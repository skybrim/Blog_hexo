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
Xcode 11.4.1

```bash
# 编译工具
brew install cmake ninja

# 下载源码
mkdir swift-source
cd swift-source
git clone https://github.com/apple/swift.git
./swift/utils/update-checkout --clone

# 切换到最新的 release tag （写文是 5.2.4）
./swift/utils/update-checkout --tag swift-5.2.4-RELEASE

# 编译 
# -x 使用 xcode 编译
# -r --release-debuginfo
./swift/utils/build-script -x -r --debug-swift-stdlib 
```

## 更新源码

```bash
./swift/utils/update-checkout
./swift/utils/build-script --release-debuginfo --debug-swift-stdlib
```

## 单个文件编译

```bash
# --line-directive '' 用于在生成的文件中去掉不必要的说明；
# -o 输出目录
./swift/utils/gyb \
    --line-directive '' \
    -o ./mycode/Sequence.swift \
    ./swift/stdlib/public/core/Sequence.swift.gyb
```

## 查找源码

* 如果知道源码具体位置，直接打开对应的文件查看
* 全局搜索 ```** public func xxx```