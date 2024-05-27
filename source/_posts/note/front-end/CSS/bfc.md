---
title: bfc-块级格式化上下文
link: bfc
catalog: true
date: 2023-11-25 18:03:11
tags:
  - 前端
  - CSS
  - BFC
categories:
  - [笔记, 前端, CSS]
---

# 什么是 BFC

BFC 就是一个 css 布局区域，这个区域内元素布局有一定的规则

# 哪些元素是 BFC（怎么让元素成为 BFC）

1. body 标签就是一个 bfc
2. inline-block 是 bfc
3. float 的元素 是 bfc
4. 除了 static, relative 的定位元素 是 bfc
5. overflow 设置为 hidden，auto 的元素 是 bfc

# BFC 的规则，BFC 的特点

1. BFC 区域内的普通的块级元素垂直方向由上到下排列
   我们平时写的 html 标签都在 body 标签里面，也就是都在 bfc 渲染区域内，所以要遵守这个规则，所以才会从上到下排列
2. BFC 计算高度的时候，会计算浮动元素和定位元素的高度。
   基于这一点，可以去除浮动带来的副作用（让父元素变成 bfc， 这样就会计算浮动元素的高度）
3. BFC 渲染区域内的上下相邻的普通的块级元素垂直方向会出现外边距合并
4. BFC 的样式不受外界影响
   基于这个可以解决外边距合并的问题，只需要让其中一个元素成为 BFC，因为成为 BFC 的元素的外边距是自己的属性，也就是 BFC 自身的属性
5. BFC 不会和浮动元素重叠
   基于这一点可以做两栏布局，和三栏布局
