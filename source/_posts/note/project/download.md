---
title: 前端实现文件下载的几种方式
subtitle: 前端实现文件下载的几种方式
link: download
catalog: true
date: 2023-07-18 10:13:12
tags:
  - 项目
  - 文件下载
categories:
  - [笔记, 项目]
---

# 后端提供的下载文件的方式

1. 直接返回文件的网络地址（一般用在静态文件上，比如图片以及各种音视频资源等）
2. 返回文件流（一般用在动态文件上，比如根据前端选择，导出不同的统计结果 excel 等）

# 不同方式，前端的处理方案

#### 第一种

- 通过 a 标签的`download`属性和`click`函数

  ```html
  <a href="https://www.baidu.top.pdf" download="附件.pdf">下载文件</a>
  ```

- `window.location.href` (推荐)

  ```javaScript
  <script>
  function Download() {
    window.location.href = 'www.baidu.pdf'
  }
  </script>
  ```

- `window.open`(推荐)

  ```javaScript
  <script>
    function Download() {
      window.open('www.baidu.pdf')
    }
  </script>
  ```

#### 第二种 使用 blob 文件流下载

```javaScript
 <script>
    function Download() {
      axios({
        url: "www.baidu.pdf",
        method: 'GET',
        responseType: 'blob', // 这里就是转化为blob文件流
        headers: {
          token: 'sss'     // 可以携带token
        }
      }).then(res => {
        const href = URL.createObjectURL(res.data)
        const box = document.createElement('a')
        box.download = '附件.pdf'
        box.href = href
        box.click()
      })
    }
  </script>
```
