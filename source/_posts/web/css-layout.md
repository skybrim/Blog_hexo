---
title: css 布局
comments: true
date: 2020-09-24 15:04:08
tags: [web, css]
---


简单备忘[css布局](https://developer.mozilla.org/zh-CN/docs/Learn/CSS/CSS_layout/Introduction)

<!--more-->

## display

### block

```
display: block; // 块
```

### inline

```
display: inline; // 内联
```

### inline-block

```
display: inline-block; // 中间状态
```
一个元素使用 display: inline-block，实现我们需要的块级的部分效果：

设置width 和height 属性会生效；padding, margin, 以及border 会推开其他元素。

但是，它不会跳转到新行，如果显式添加 width 和 height 属性，它只会变得比其内容更大。

### Flexbox

```
display: flex;
```
Flexbox用于设计横向或纵向的布局

在想要进行flex布局的父元素上应用display: flex ，所有直接子元素都将会按照flex进行布局

### Grid

```
display: grid;
```
Grid布局则被设计用于同时在两个维度上把元素按行和列排列整齐。

### table

```
display: table;
```
被用于兼容一些不支持Flexbox和Grid的浏览器。

## 浮动

```
float: left;

// left — 将元素浮动到左侧。
// right — 将元素浮动到右侧。
// none — 默认值, 不浮动。
// inherit — 继承父元素的浮动属性。
```
把一个元素“浮动”(float)起来，会改变该元素本身和在正常布局流（normal flow）中跟随它的其他元素的行为。

这一元素会浮动到左侧或右侧，并且从正常布局流(normal flow)中移除，这时候其他的周围内容就会在这个被设置浮动(float)的元素周围环绕。

## 定位

### 静态定位

将元素放在文档布局流的默认位置

### 相对定位

```
position: relative;
```
相对定位(relative positioning)让你能够把一个正常布局流(normal flow)中的元素从它的默认位置按坐标进行相对移动

### 绝对定位

```
position: absolute;
```
将元素移出正常布局流(normal flow)，以坐标的形式相对于它的容器定位到web页面的任何位置，以创建复杂的布局。

### 固定定位

```
position: fixed;
```
固定定位(fixed positioning)同绝对定位(absolute positioning)一样，将元素从文档流(document flow)当中移出了。

但是，定位的坐标不会应用于"容器"边框来计算元素的位置，而是会应用于视口(viewport)边框。

利用这一特性，我们可以轻松搞出一个固定位置的菜单，而不受底下的页面滚动的影响。

### 粘性定位

```
position: sticky;
```
将默认的静态定位(static positioning)和固定定位(fixed positioning)相混合。

当一个元素被指定了position: sticky时，它会在正常布局流中滚动，直到它出现在了我们给它设定的相对于容器的位置，这时候它就会停止随滚动移动，就像它被应用了position: fixed一样。