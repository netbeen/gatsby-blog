---
title: HTTP 缓存机制
date: '2021-11-07'
tags: ['http', 'network']
category: etc
---

## 前言

你在阅读本文前需要了解的基础知识：计算机网络技术知识

## 为什么需要 HTTP 缓存机制

缓存是一种保存资源副本并在下次请求时直接使用该副本的技术。当缓存发现请求的资源已经被存储，它会拦截请求，返回该资源的拷贝，而不会去源服务器重新下载。这样带来的好处有：缓解服务器端压力，提升性能(获取资源的耗时更短了)。

## 缓存策略的分类

根据浏览器是否会向服务器发起请求分类，可将缓存机制分为 `强缓存` 和 `协商缓存` ，其中 `强缓存` 命中后，浏览器不会向服务器发起请求，而 `协商缓存` 无论命中与否，都会向服务器发起请求。

### 强缓存

#### Expires

`Expires` header 字段是 `HTTP/1.0` 协议的产物，其值是一个 GMT 时间戳。其含义是当前资源的失效时间点，如果下次请求该资源的时间早于 `Expires` 的值，则命中缓存，反之亦然。

缺点：由于 `Expires` 表示的是一个绝对时间点，这就要求浏览器所设置的 GMT 时间需要和服务器一致才可生效。而在实际情况下，由于各种误差或者其他原因，浏览器和服务器的时间差异无法保证。在这种情况下 `Expires` 字段的效果就会与设计效果差距很大。

```text
expires: Mon, 19 Aug 2041 09:20:05 GMT
```

#### Cache-Control

`Cache-Control` header 字段是 `HTTP/1.1` 协议的产物，该字段是一些 `key-value` 的组合，可选的属性如下：

- `max-age` ：该资源可以被缓存多长时间，单位：秒
- `s-maxage` ：语义与 `max-age` 一致，只针对代理服务器缓存
- `public` ：该资源被任何缓存缓存
- `private` ：该资源只能被个人用户缓存，不可以被代理服务器缓存
- `no-cache` ：该资源不使用强缓存，使用协商缓存
- `no-store` ：该资源禁止缓存
- `must-revalidate` ：该资源过期前可以使用，过期后必须向服务器验证

`Cache-Control` 的优先级高于 `Expires`

```text
cache-control: max-age=2592000,s-maxage=3600
```

#### Pragma

`Pragma` 也是 `HTTP/1.0` 的产物，语义与 `Cache-Control` 的 `no-cache` 完全一致，目前使用量已经不多了（不推荐使用）

`Pragma` 的优先级高于 `Cache-Control`

### 协商缓存

协商缓存 是由服务器来确定资源是否可用。主要涉及 `Last-Modified/If-Modified-Since` 和 `ETag/If-None-Match` 两对字段，成对出现，即第一次请求的响应头带上某个字段， `Last-Modified` or `Etag` ，则后续请求也会带上对应的字段 `If-Modified-Since` or `If-None-Match` ，反之亦然。如果缓存命中，则服务器返回 `304` 状态码，反之，服务器返回对应的资源

#### Last-Modified/If-Modified-Since

`Last-Modified` 和 `If-Modified-Since` 的值都是一个 GMT 时间戳，指的是这个资源在服务器最后更新的时间，如果服务器接收到包含 `If-Modified-Since` 的请求时，会比对对应资源的最后更新时间：
- 如果一致，则返回 `304` ，
- 如果不一致，则返回资源

```text
Last-Modified: Mon, 19 Jul 2021 00:14:57 GMT
```

#### ETag/If-None-Match

`ETag` 和 `If-None-Match` 的值是一段哈希，来表示资源的内容是否被改变。当服务器接收到包含 `If-None-Match` 的请求时，会计算对应资源的哈希，
- 如果一致，则返回 `304` + `ETag` 
- 如果不一致，则返回资源 + `ETag`

`ETag/If-None-Match` 的优先级高于 `Last-Modified/If-Modified-Since`

```text
ETag: "60f4c401-e0a"
```

## 使用经验

- 对于静态资源（包括script/css），目前主流的方案会采用在 URL 中添加版本号的方案，所以单个 URL 基本不会改变，可以使用长时间的强缓存
- 对于动态资源，可以根据实际业务场景配置短时间的强缓存
- 对于与特定用户有关或者权限相关的敏感资源，需要设置 `Cache-Control` 的 `private` 属性

## 参考文献

- [HTTP 缓存 - HTTP | MDN](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Caching)
