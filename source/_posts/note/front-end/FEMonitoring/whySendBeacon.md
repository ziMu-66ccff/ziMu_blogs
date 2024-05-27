---
title: 前端监控中为什么选择sendBeacon来发送数据
link: whySendSeacon
catalog: true
date: 2023-11-20 11:11:40
tags:
  - 前端
  - 前端监控
  - sendSeacon
categories:
  - [笔记, 前端, 前端监控]
---

# 前端监控中发送数据通常有什么样的问题

前端监控中通常尝试在卸载（unload）文档之前向 Web 服务器发送数据。

1. 过早的发送数据可能导致错过收集数据的机会。
2. 然而，对于开发者来说保证在文档卸载期间发送数据一直是一个困难。
   - 在`unload`时，无法保证数据被可靠的稳定的发送出去了，因为浏览器通常会忽略在 `unload` 事件处理器中产生的异步 `XMLHttpRequest。`
   - 发送数据可能会迫使浏览器延迟卸载文档，并使得下一个导航出现的更晚。

# 各种技术发送数据时的优缺点

## XMLHttpRequest 或 Fetch API：

### 优点：

1. 支持各种请求方法：` XMLHttpRequest` 和 `Fetch API` 支持多种请求方法，包括 `GET`、`POST` 等。

### 缺点：

1. 同步请求可能阻塞页面： 如果在页面卸载或跳转时同步发送请求，可能会阻塞页面的卸载或跳转，影响用户体验。
2. 不适用于页面关闭时的异步请求：浏览器通常会忽略在 `unload` 事件处理器中产生的异步 `XMLHttpRequest。`

## image 对象

### 优点：

1. 简单： 通过创建 Image 对象，可以简单地发送 GET 请求。

### 缺点：

1. 仅支持 GET 请求： 由于是通过图片的 src 属性发送请求，只能发送 GET 请求。
2. 不可靠： 无法确定请求是否成功，不能获得服务器的响应状态。

## sendBeacon

### 优点：

1. 异步方式发送请求： `sendBeacon` 是异步的，不会阻塞页面的卸载或者跳转，适用于页面关闭时仍需要发送统计数据的场景。
2. 可靠性： 由于是浏览器在后台默默处理，不会因为页面的关闭或者跳转而中断发送。

### 缺点：

1. 仅支持 `POST` 请求： `sendBeacon` 只支持使用 `POST` 方法发送数据，因此不适用于所有类型的请求。

# 前端监控中为什么选择 sendBeacon 来发送数据

1. 是异步发送数据，不会阻塞页面的卸载和跳转
2. 可靠性很高，不会因为页面的关闭 or 跳转 而 终端发送
3. 虽然只支持`post`请求，但是用来发送埋点的统计数据，却足够了，也极为合适

# sendBeacon 使用方法

`navigator.sendBeacon(url, data);`
`url`: 上报地址（接口地址）
`data`： 可以是`ArrayBufferView` ,`Blob`, `DOMString`, `Formdata`

根据官方规范，需要 `request header` 为 CORS-safelisted-request-header，在这里则需要保证 `Content-Type `为以下三种之一：

- `application/x-www-form-urlencoded`
- `multipart/form-data`
- `text/plain`

我们一般会用到 DOMString , Blob 和 Formdata 这三种对象作为数据发送到后端，下面以这三种方式为例进行说明。

```javascript
import queryString from 'query-string';

// 1. DOMString类型，该请求会自动设置请求头的 Content-Type 为 text/plain
const reportData = (url, data) => {
  navigator.sendBeacon(url, data);
};

// 2. 如果用 Blob 发送数据，这时需要我们手动设置 Blob 的 MIME type，
// 一般设置为 application/x-www-form-urlencoded。
// 然后可以用query-string库来做编吗将data编码成query-string格式
const reportData = (url, data) => {
  const blob = new Blob([
    queryString.stringify(data),
    {
      type: 'application/x-www-form-urlencoded',
    },
  ]);
  navigator.sendBeacon(url, blob);
};

// 3. 发送的是Formdata类型，
// 此时该请求会自动设置请求头的 Content-Type 为 multipart/form-data。
const reportData = (url, data) => {
  const formData = new FormData();
  Object.keys(data).forEach((key) => {
    let value = data[key];
    if (typeof value !== 'string') {
      // formData只能append string 或 Blob
      value = JSON.stringify(value);
    }
    formData.append(key, value);
  });
  navigator.sendBeacon(url, formData);
};
```
