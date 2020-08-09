---
title: homebrew-source
comments: true
date: 2020-08-08 12:23:38
tags: [inbox, homebrew]
---

Homebrew 换源
<!--more-->

[中科大开源软件镜像服务](https://lug.ustc.edu.cn/wiki/lug/services/mirrors)

## 切换中科大的镜像

```bash
# 替换brew.git:
cd "$(brew --repo)"
git remote set-url origin https://mirrors.ustc.edu.cn/brew.git

# 替换homebrew-core.git:
cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
git remote set-url origin https://mirrors.ustc.edu.cn/homebrew-core.git

# Homebrew cask 软件仓库，提供 macOS 应用和大型二进制文件
# 替换homebrew-cask.git:
cd "$(brew --repo)"/Library/Taps/homebrew/homebrew-cask
git remote set-url origin https://mirrors.ustc.edu.cn/homebrew-cask.git

# 替换 Homebrew Bottles，这是Homebrew提供的二进制代码包
# zsh
export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.ustc.edu.cn/homebrew-bottles
```

## 切回官方

```bash
# 重置brew.git:
cd "$(brew --repo)"
git remote set-url origin https://github.com/Homebrew/brew.git

# 重置homebrew-core.git:
cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
git remote set-url origin https://github.com/Homebrew/homebrew-core.git

# 重置homebrew-cask.git:
cd "$(brew --repo)"/Library/Taps/homebrew/homebrew-cask
git remote set-url origin https://github.com/Homebrew/homebrew-cask
```





