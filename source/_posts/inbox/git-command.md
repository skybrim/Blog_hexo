---
title: git-command
comments: true
date: 2020-05-20 10:22:25
tags: [git, inbox]
---

git 命令备忘

持续更新 ...
<!--more-->

## 状态模型

![](https://cdn.jsdelivr.net/gh/skybrim/AllImages@dev/git_0.png)

## 仓库结构

![](https://cdn.jsdelivr.net/gh/skybrim/AllImages@dev/git_1.png)


## 基础

```bash

git commit # 提交

git commit --amend # 追加提交

git checkout <your-branch-name> # 切换到指定分支

git checkout -b <your-branch-name> # 创建分支并切换到该分支

git merge <your-branch-name> # 合并指定分支到当前分支

git rebase <your-branch-name> # 合并指定分支到当前分支，创造更线性的提交历史

```

## 高级

```bash

git checkout <commit-SHA-1> # 分离 HEAD，HEAD 指向某个提交

# 相对引用 ^ 分离 HEAD
git checkout master^ # HEAD 切换到 master 的父节点
git checkout master^^ # HEAD 切换到 master 的父节点的父节点
git checkout HEAD^ # HEAD 指向 HEAD 的父节点

# 相对引用 ~ 分离 HEAD
git checkout HEAD~3 # HEAD 向后移动 3 步

# 强制移动分支到指定提交
git branch -f master HEAD~3 # 强制 master 分支向后移动 3 步

# 撤销变更
git reset # 向后移动分支
git revert # 创建了一个新的提交，内容是撤销当前提交，这个操作针对远程分支

```

## 移动提交记录

利用 cherry-pick 和 rebase -i 命令，来修改分支的提交记录

```bash

# 将指定的提交，复制到当前分支
git cherry-pick <commit-SHA-1> <commit-SHA-1> ... 

# 在编辑器里修改提交
git rebase -i HEAD~5

```

## 杂项

```bash
git tag <tag-name> <commit-SHA-1>
```

## 参考

[Learn Git](https://learngitbranching.js.org/?locale=zh_CN)

[一文讲透 Git 底层数据结构和原理](https://zhuanlan.zhihu.com/p/142289703?utm_source=wechat_session&utm_medium=social&utm_oi=52825543409664&from=groupmessage&isappinstalled=0&wechatShare=1&s_s_i=QWra%2BCirMCY6QhghF0le9aHvj5P3HmW%2FdS4rvaZuIqY%3D&s_r=1)