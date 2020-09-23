---
title: css-selectors
comments: true
date: 2020-09-23 16:14:00
tags: [web, css]
---

css selectors 备忘
<!--more-->

```css
/* 类型选择器 */
h1 {}

/* 类选择器 */
.box {}

/* id选择器 */
#unique {}

/* 标签属性选择器 */
a[title] {} /* 存在title属性的<a> 元素 */
a[href="https://example.com"] {} /* 存在href属性并且属性值匹配"https://example.org"的<a> 元素 */
a[href*="example"] {} /* 存在href属性并且属性值包含"example"的<a> 元素 */
a[href*="insensitive" i] {} /* 包含 "insensitive" 的链接,不区分大小写 */
a[href*="cAsE" s] {} /* 包含 "cAsE" 的链接，区分大小写 */ 
a[href^="#"] {} /* 以 "#" 开头的页面本地链接 */
a[href$=".org"] {} /* 存在href属性并且属性值结尾是".org"的<a> 元素 */
a[class~="logo"] {} /* 存在class属性并且属性值包含以空格分隔的"logo"的<a>元素 */

/* 伪类选择器 */
/* 指定要选择的元素的特殊状态 */
/* https://developer.mozilla.org/zh-CN/docs/Web/CSS/Pseudo-classes */
button:hover {} /* 所有用户指针悬停的按钮 */

/* 伪元素选择器 */
/* 指定被选择元素的特定部分 */
/* https://developer.mozilla.org/zh-CN/docs/Web/CSS/Pseudo-elements */
p::first-line {} /* 每一个 <p> 元素的第一行。 */

/* 后代选择器 */
/* 当使用 空格 连接两个元素时，使得该选择器可以只匹配那些由第一个元素作为祖先元素的所有第二个元素(后代元素) */
div span {} /* 每一个 <div> 下的 <span> 元素 */

/* 子选择器 */
/* 使用 > 选择符分隔两个元素时,它只会匹配那些作为第一个元素的直接后代(子元素)的第二元素 */
div > span {}

/* 相邻兄弟选择器  */
/* 使用 + 介于两个选择器之间，当第二个元素紧跟在第一个元素之后，并且两个元素都是属于同一个父元素的子元素，则第二个元素将被选中。 */
img + p {} /* 图片后面紧跟着的段落将被选中 */

/* 兄弟选择器 */
/* 位置无须紧邻，只须同层级，A~B 选择A元素之后所有同层级B元素 */
p ~ span {} /* <p> 后面跟着的 <span> 被选中*/
```
