---
title: 图片懒加载
subtitle: 如何优雅的实现图片懒加载
link: lazyLoading
catalog: true
date: 2023-02-17 16:52:37
tags:
  - 前端
  - 性能优化
  - 懒加载
categories:
  - [笔记, 前端, 性能优化]
---

# 实现图片懒加载

## 传统的不够优雅的实现方式

- 完整代码

```javascript
const imgs = document.querySelectorAll('img');

window.addEventListener('scroll', () => {
  imgs.forEach((img) => {
    if (img.getBoundingClientRect().top < window.innerHeight) {
      const dataSrc = img.getAttribute('data-src');
      img.setAttribute('src', dataSrc);
    }
  });
  console.log('事件被触发');
});
```

- 基本原理
  监听`scroll`事件，`scroll`事件一旦被触发，就在`callback`里面通过`xxx.getBoundingClientRect().top`拿到元素距离可视区顶部的距离，并通过`window.innerHeight`取得可视区的高度，将两个高度作比较，一旦前者小于后者，则意味着图片进入了可视区，就将自定义属性`data-src`存储的图片的`url`赋值给`src`属性，加载图片。
- 带来的问题
  `scroll`事件会被触发 n 多次，很容易造成卡顿，影响用户体验。
- 解决方案
  利用**节流**，但是这种解决方案并不优雅，而且也不能完完全全的解决问题。

## 利用 intersectionOberver API 优雅的实现懒加载

- 完整代码

```javascript
const imgs = document.querySelectorAll('img');

const observer = new IntersectionObserver((entries) => {
  entries.forEach((entry) => {
    if (entry.isIntersecting) {
      const img = entry.target;
      dataSrc = img.getAttribute('data-src');
      img.setAttribute('src', dataSrc);
      observer.unobserve(img);
    }
  });

  console.log('事件被触发');
});

imgs.forEach((img) => {
  observer.observe(img);
});
```

- intersectionObserver API 简单介绍

  - `intersectionObserver`是一个构造函数，可以给其传递一个参数`callback`, 会返回一个观察者实列`observer`
  - 调用实例身上的`observe`方法，并给其传一个元素的引用，就可以开始观察这个元素。相对的，调用实列身上的`unobserve`方法就可以取消观察这个元素。
  - 当被观察的元素进入 or 离开可视区的时候，`callback`就会被触发。
  - `callback`的参数为一个数组，数组里面存储的是被观察的元素中进入 or 离开可视区的元素的一些信息。比如参数数组中的元素的`isIntersecting`属性存储的就是该元素是进入可视区还是离开可视区，
    `target`属性存储的则是元素的引用。

- 利用 intersectionObserver API 实现懒加载的原理

1. `document.querySelectorAll(img)`获取对所有`img`标签的引用，存储进`imgs`
2. new 一个 intersectionObserver 实列 `observer`， 给其传递一个`callback`作为参数， 在`callback`中进行接下来的操作。
3. 拿到`callback`的参数`entries` _（为一个保存了被观察的元素中进入 or 离开可视区的元素的信息的数组_。遍历这个参数`entries`， 取出参数中的元素`entry`。
4. 判断`entry.isIntersecting`是否为`true`， 为`true`则说明是进入了可视区，则进行接下来的操作，为`false`则说明是离开可视区，则啥都不做。
5. 利用`entry.target`取出进去可视区的元素, 将其赋值给`img`， 然后取出`img`身上的`data-src`属性存储的`url`赋值给`src`属性，实现懒加载，并且调用`observer.unobserve`来取消对这个图片元素的观察（这样一旦元素被加载，就取消了对它的观察，就不会再触发 callback，避免不必要的性能消耗）
6. 遍历`imgs`调用`observer.observe(img)`来观察所有的`img`标签
