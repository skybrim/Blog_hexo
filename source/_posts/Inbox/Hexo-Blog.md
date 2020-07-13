---
title: hexo 搭建 blog
date: 2018/11/11
tags: [hexo, inbox]
comments: true
---

方案：Hexo + Next + GitHub Pages
<!--more-->

环境：macOS Mojave 10.14.4  时间：2019-05-13

<!--more-->

## 环境安装

* 安装git

```bash
brew install git
```

* 安装nvm

```bash
wget -qO-https://raw.githubusercontent.com/creationix/nvm/v0.34.0/install.sh |bash
```

* 安装Node.js

```bash
nvm install stable
```

* 安装hexo

```bash
npm install -g hexo-cli
```

* 安装next  

```bash
cd blog目录  
git clone https://github.com/iissnan/hexo-theme-next themes/next
```

## 配置

* 配置站点  
    打开 blog 目录下的 _config.yml 文件

* 基本设置

    >title: 标题  
    >subtitle: 副标题  
    >description:  
    >keywords:  
    >author: 作者  
    >language: zh-Hans  
    >timezone:  

* 主题设置

    >theme: next

* 部署设置

    >deploy:  
    >type: git  
    >repo: repo仓库地址  
    >branch: repo的分支  

* 配置主题
    打开 blog 目录,themes -> next -> _config.yml 文件
    >since: 2018 #copyright起始时间  
    >menu,根据个人需求，我使用了首页home和标签页tag  
    >Schemes,next主题的不同样式，我选择 Pisces  
    >avatar，个人头像设置，头像图片放在主题的 image 文件夹里面，/images/Avatar.png  
    >disqus，评论插件，需要去申请一个shortname  
    >google_analytics，谷歌网页分析，需要申请一个ID  
    >busuanzi_count,浏览量等统计插件，next当前这个版本有bug，需要自己去修改一下配置  

* 使用过的第三方插件：  
    >hexo-filter-mermaid-diagrams，配合markdown生成流程图  
    >hexo-blog-encrypt，加密插件  

### 写作

* tag
新建tag页面

    ```bash
    hexo new page tags
    ```

* 设置页面类型

    ```bash
    title: 标签
    date: 2014-12-22 12:39:04
    type: "tags"
    ```

* 生成文章

    ```bash
    hexo new post 文章标题
    ```

* 生成文件

    ```bash
    hexo g
    ```

* 部署
如果已经在站点配置文件设置好，可以直接部署到 github pages 上

    ```bash
    hexo d
    ```

    本地查看

    ```bash
    hexo s
    ```
