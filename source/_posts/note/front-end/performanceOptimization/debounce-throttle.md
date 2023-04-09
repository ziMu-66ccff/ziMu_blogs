---
title: 防抖和节流
subtitle: 实现防抖和节流
link: debounce-throttle
catalog: true
date: 2023-02-17 18:28:13
tags:
  - 前端
  - 性能优化
  - 防抖节流
categories:
  - [笔记, 前端, 性能优化]
---

# 防抖

- 简要介绍
  防抖是指事件被触发后的一定时间后才执行回调，这期间如果事件被再次触发，则重新计时，常用于按钮的点击事件，避免按钮短时间内被多次点击，向服务器发送过多的请求
- 主要应用
  - 按钮提交场景：防⽌多次提交按钮，只执⾏最后提交的⼀次
  - 服务端验证场景：表单验证需要服务端配合，只执⾏⼀段连续的输⼊事件的最后⼀次。
- 完整代码

```javascript
function debounce(fn, dup === 100) {
    let timer;
    return function(...arg) {
        clearTimeout(timer);
        timer = setTimeout(() => {
            fn.call(this, ...arg)
        }, dup)
    }
}
```

# 节流

- 简要介绍
  节流是指在一定事件内，事件被多次触发只生效第一次， 常用于对`scroll`事件的处理
- 主要应用
  - 拖拽场景：固定时间内只执⾏⼀次，防⽌超⾼频次触发位置变动
  - 缩放场景：监控浏览器`resize`
  - 动画场景：避免短时间内多次触发动画引起性能问题
- 完整代码

```javascript
function throttle(fn, dup === 100) {
    let timer = null;
    return function(...arg) {
        if (timer === null) {
            fn.call(this, ...arg);
            timer = setTimeout(() => {
                timer = null;
            }, dup)
        }
    }
}
```
