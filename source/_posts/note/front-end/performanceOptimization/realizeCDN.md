---
title: 理解CDN
subtitle: 理解CDN及其原理
link: realizeCDN
catalog: true
date: 2023-02-17 14:15:34
tags:
  - 前端
  - 性能优化
  - CDN
categories:
  - [笔记, 前端, 性能优化]
---

# 理解 CDN 及其原理

## CDN 是什么

CDN(Content Delivery Network)内容分发网络， 是一种靠互联网连接的电脑网络系统，它根据用户所在的区域，选择离用户最近的服务器来将相关文件发送给用户，以此提供更快速，更稳定的网络内容分发服务。

## CDN 由什么组成

- 分发服务系统
  分发服务系统最基本的工作单位是 Cache 设备，cache 设备负责直接响应用户的请求，将缓存在本地的内容快速的提供给用户，并且负责与源站点进行同步，将更新的内容，本地没有的内容，缓存到本地
- 负载均衡系统， 主要负责对所有发起请求的用户进行一个访问调度，主要分为全局负载均衡系统和本地负载均衡系统。
  - 全局负载均衡系统，主要根据发起请求的用户所在的区域，选择一个最靠近用户的 cache。
  - 本地负载均衡系统，主要是负责节点内部的设备负载均衡。
- 运营管理系统
  主要是负责业务层面与外界系统的交互。

## CDN 的作用

- 性能方面
  - 用户收到的内容来自最近的服务器，延迟更低，内容加载更快。
  - 部分资源的请求分配给了 CDN，服务器的负载减小。
- 安全方面， 可以有效的防止黑客攻击

## DNS 域名解析的过程

比如在浏览器输入www.test.com的解析过程如下

- 检查浏览器缓存
- 检查操作系统缓存，比如 hosts 文件
- 检查路由器缓存
- 如果以上都没有找到缓存，则会向网络服务商的 LDNS 服务器查询
- 如果 LDNS 服务器没有查询到，就会向根域名服务器进行查询
- 根服务器会返回一个顶级域名服务器，比如 `.com`, `.org`的地址，本例是返回`.com`的地址
- 接着向顶级域名服务器查询，顶级域名服务器会返回一个次级域名服务器的地址，本文中为`.test`的地址
- 接着向次级域名服务器查询，次级域名服务器会返回通过域名查询到的目标 ip， 本文为`www.test.com`的地址
- Local DNS Server 会缓存结果，并返回给用户， 缓存在系统中

## CDN 工作原理

- 用户没有用 CDN 缓存资源时

1. 浏览器会通过 DNS 域名解析（上面的 DNS 域名解析）, 得到域名对应的 ip 地址
2. 浏览器会根据得到的 ip 地址，向域名的服务主机发送请求
3. 服务器向浏览器响应数据。

- 用户使用了 CDN 缓存资源时

1. 浏览器的 DNS 域名解析会发现这个 url 对应的时 CDN 的 DNS 服务器，则会将域名解析权交给 CDN 专用的 DNS 服务器。
2. DNS 服务器会将 CDN 的全局负载均衡设备的 ip 返回给用户。
3. 用户会通过 ip 向全局负载均衡设备发送请求
4. 全局负载均衡设备会根据用户的 ip 地址，url，选择一台用户区域的区域负载均衡设备，让用户向这个设备发送请求
5. 区域负载均衡设备会选择一个适合的缓存服务器来提供服务，将这个服务器的 ip 地址返回给全局均衡设备。
6. 全局均衡设备会将这个 ip 返回这个用户
7. 用户会通过这个 ip 地址来向缓存服务器发起请求
8. 缓存服务器响应用户的请求，将内容发送至用户终端。

![CDN](https://i.328888.xyz/2023/02/17/pcIMz.png)

如果缓存服务器没有用户想要的内容，则会向它的上一级缓存服务器请求内容，直到获取到想要的内容，如果还是没有，则会回到自己的服务器去请求内容。
