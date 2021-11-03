---
title: HTTP 协议演进历史
date: '2020-02-15'
tags: ['http', 'http2', 'protocol', 'history']
category: etc
---

## 前言

你在阅读本文前需要了解的基础知识：计算机网络技术知识

## HTTP 雏形

1989-03-12，欧洲核子研究中心 `CERN` 的 Timothy John Berners-Lee 通过报告[《Information Management: A Proposal》](https://cds.cern.ch/record/369245/files/dd-89-001.pdf) 提出设想，为了解决`CERN`的大量信息无法有效管理的问题，设计了一个分布式的超文本系统

1990-12-20，Tim 开发了第一个浏览器`WorldWideWeb`，后改名为`Nexus`

1991-08-06，第一个网站 [http://info.cern.ch/](http://info.cern.ch/) 上线，至今依旧可以访问

## HTTP/0.9

最初版本的HTTP协议没有版本号，后来它的版本号被定位在 `0.9` 以区分后来的版本。该版本协议极其简单：request 由单行指令构成，以唯一可用方法GET开头

```text
GET /mypage.html
```

response 也极其简单的，只包含响应文档本身

```text
<HTML>
这是一个非常简单的HTML页面
</HTML>
```

## HTTP/1.0

- request 指定版本号（固定为HTTP/1.0）
- response 指定状态码
- request 和 response 增加 header ，通过 header 元数据 增强可扩展性
- 支持传输文本以外的文档

1991年~1995年，由于没有协议标准，导致各方的扩展出现不兼容的问题，1996年11月，[RFC 1945](https://datatracker.ietf.org/doc/html/rfc1945) 发布，制定了HTTP/1.0的协议标准

## HTTP/1.1

1997年1月，[RFC 2068](https://datatracker.ietf.org/doc/html/rfc2068) 发布，制定了HTTP/1.1的协议标准，后经两次修订：1999年6月的 [RFC 2616](https://datatracker.ietf.org/doc/html/rfc2616) 和 2014年6月的 [RFC 7230](https://datatracker.ietf.org/doc/html/rfc7230) 至 [RFC 7235](https://datatracker.ietf.org/doc/html/rfc7235)

- 支持TCP连接复用，避免了每个HTTP请求都需要重建TCP连接
- 更多的缓存策略，相较于HTTP/1.0就有的 `If-Modified-Since` 和 `Expires` ，增加 `Entity tag` 、 `If-Unmodified-Since` 、 `If-Match` 、 `If-None-Match`
- 支持response分块，通过 header 里的 `range` 只请求资源的一部分，Code = 206
- 增加 header 里的 `host` 字段，允许多个域名分享同一个ip，即虚拟主机技术
- 支持长连接 `PersistentConnection` 和请求的流水线 `Pipelining` 处理

## SPDY

取名源于 Speedy ，2009年由 Google 提出，2016年停止支持

- 支持多路复用 `Multiplexing`，多个 stream 共享一个 TCP 连接，解决 `HOL blocking` 问题（消除了浏览器的同域并发请求数上限），
- 支持设置请求优先级 `Request Prioritization` ，高级别内容如 html 优先响应
- 支持 header 压缩
- 强制要求 https
- 支持 `Server Push` ，例如我的网页有 `sytle.css` 的请求，在浏览器在收到 `sytle.css` 数据的同时，服务端会将`sytle.js` 的文件推送给客户端，当浏览器尝试获取 `sytle.js` 时就可以从缓存中获取到，不再发请求了

## HTTP/2.0

SPDY/3为蓝图起草，2013年IETF提出。与SPDY的差异主要如下

- 二进制传输
- 不强制使用HTTPS，保证向下兼容
- header 压缩算法变更，由 DEFLATE 变更为 HPACK

## 参考文献

- [Information Management: A Proposal](https://cds.cern.ch/record/369245/files/dd-89-001.pdf)
- [HTTP的发展 - HTTP | MDN](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Basics_of_HTTP/Evolution_of_HTTP)
