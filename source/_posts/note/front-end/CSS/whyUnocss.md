---
title: 为什么选择unocss
link: whyUnocss
catalog: true
date: 2023-11-25 17:20:45
tags:
  - 前端
  - CSS
  - 原子化css
categories:
  - [笔记, 前端, CSS]
---

# 什么是原子化 css

原子化 css 就是提供了很多`preset class`（预设类），就是提前写好了很多类名，这些类名都有自己的样式，可以直接用这些类名，就不需要写 css 了（提供的预设类是足够的，完全不需要写 css）

# 早期原子化 css 带来的问题（现在都解决了）

1. 不能按需引入预设类，导致项目里有很多用不上的预设类，影响构建速度
   **解决方案**：现在可以构建阶段扫描代码，能够按照代码中的实际使用情况生成工具类，解决了原子类使 CSS 产物膨胀问题。
2. 不能使用提供的预设类来自定义类，导致不能完全脱离 css 代码
   **解决方案**：针对原子类堆叠降低可读性的问题，提供了 `@apply` 语法支持在 CSS 中对多个原子类进行合并，与语义化 CSS 实现了很好的配合。
3. 没有插件提供，提升我们使用预设类的效率
   **解决方案**：现在都推出了 VSCode 插件，为编写原子类提供了充分的提示与自动补全。
