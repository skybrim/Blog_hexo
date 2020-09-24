---
title: css-box-model
comments: true
date: 2020-09-24 10:28:02
tags: [web, css]
---

css box model
<!--more-->

## block inline

```
display: inline; // 内联
display: block; // 块
display: inline-block; // 中间状态
```

一个元素使用 display: inline-block，实现我们需要的块级的部分效果：

设置width 和height 属性会生效；padding, margin, 以及border 会推开其他元素。

但是，它不会跳转到新行，如果显式添加 width 和 height 属性，它只会变得比其内容更大。


## 标准盒模型

* Content box: 这个区域是用来显示内容，大小可以通过设置 width 和 height.
* Padding box: 包围在内容区域外部的空白区域； 大小通过 padding 相关属性设置。
* Border box: 边框盒包裹内容和内边距。大小通过 border 相关属性设置。
* Margin box: 这是最外面的区域，是盒子和其他元素之间的空白区域。大小通过 margin 相关属性设置。

在标准模型中，如果你给盒设置 width 和 height，实际设置的是 content box。 padding 和 border 再加上设置的宽高一起决定整个盒子的大小。

margin 不计入实际大小 —— 当然，它会影响盒子在页面所占空间，但是影响的是盒子外部空间。盒子的范围到边框为止 —— 不会延伸到margin。

![](https://cdn.jsdelivr.net/gh/skybrim/AllImages@dev/css-box-mode.png)

## 替代盒模型

box-sizing: border-box;

使用这个模型，所有宽度都是可见宽度，所以内容宽度是该宽度减去边框和填充部分

## 外边距 margin

```
margin-top
margin-right
margin-bottom
margin-left
```

外边距是盒子周围一圈看不到的空间。它会把其他元素从盒子旁边推开。 

外边距属性值可以为正也可以为负。设置负值会导致和其他内容重叠。

无论使用标准模型还是替代模型，外边距总是在计算可见部分后额外添加。

如果你有两个外边距相接的元素，这些外边距将合并为一个外边距，即最大的单个外边距的大小。

在下面的例子中，我们有两个段落。
顶部段落的页 margin-bottom为50px。第二段的margin-top 为30px。
因为外边距折叠的概念，所以框之间的实际外边距是50px，而不是两个外边距的总和。

![](https://cdn.jsdelivr.net/gh/skybrim/AllImages@dev/css-fold-margin.png)

## 边框 border

```
border-top
border-right
border-bottom
border-left

border-width
border-style
border-color

border-top-width
border-top-style
border-top-color
border-right-width
border-right-style
border-right-color
border-bottom-width
border-bottom-style
border-bottom-color
border-left-width
border-left-style
border-left-color
```

边框是在边距和填充框之间绘制的。

如果您正在使用标准的盒模型，边框的大小将添加到框的宽度和高度。

如果您使用的是替代盒模型，那么边框的大小会使内容框更小，因为它会占用一些可用的宽度和高度。

## 内边距 padding

```
padding-top
padding-right
padding-bottom
padding-left
```

内边距位于边框和内容区域之间。

与外边距不同，您不能有负数量的内边距，所以值必须是0或正的值。

应用于元素的任何背景都将显示在内边距后面，内边距通常用于将内容推离边框。