---
title: CDN 原理
date: '2021-11-08'
tags: ['cdn', 'network']
category: etc
---

## 前言

你在阅读本文前需要了解的基础知识：计算机网络基础知识、[DNS 工作原理](/posts/dns-introduction/)

## 为什么需要 CDN 加速

CDN `Content Delivery Network` 是为了解决浏览器与服务器的地理位置过于遥远，或者服务器负载过高导致的浏览器获取资源速度变慢的问题而产生的解决方案。本质上是通过一个部署在全球各地的庞大的服务器集群，将用户的请求导流至最近且负载合理的节点上，实现加速的效果。

## CDN 加速步骤

这里需要澄清几个概念：
- 源站域名/IP `Origin Server Domain / IP` ：需要被加速的业务资源的域名或者 IP 地址
- 加速域名：业务资源被加速后，用户最后使用的域名
- CNAME域名：CDN 平台提供的域名，会通过全局负载均衡器，引导至地理位置近且负载合理的 CDN 边缘节点
- CDN 边缘节点 `CDN Edge/Cache Server` ：离终端用户最近的 CDN 节点

用户通过 `加速域名` 访问资源，`加速域名` 会被CNAME至 `CNAME 域名` ，`CNAME 域名` 会经过 `全局负载均衡器` 访问到最接近的 `CDN 边缘节点`，`CDN 边缘节点` 会通过专线从 `CDN 中心节点` 同步数据，`CDN 中心节点` 会从 `源站` 同步数据

```text
# 使用 CDN 加速前的访问链路
浏览器 - 源站

# 使用 CDN 加速后的访问链路
浏览器 - 加速域名 - CNAME 域名 - CDN 负载均衡器 - CDN 边缘节点 - CDN 中心节点 - 源站
```

假如要为当前站点 `https://blog.netbeen.top` 开启 CDN 加速，通常需要如下几步：
- 确定源站域名为 `blog.netbeen.top.`
- 申请一个新的域名，用于用户端的 CDN 加速访问，假设为 `accelerating-blog.netbeen.top.`
- 向 CDN 平台（假设是阿里云）购买 CDN 服务，指定源站域名为 `blog.netbeen.top.` ，阿里云会生成一个 `CNAME 域名` ： `blog.netbeen.top.SOMETHING.alicdn.com.` ，这个 `CNAME 域名` 会屏蔽 CDN 平台内部的复杂实现，将所有指向 CNAME 域名的流量导流至 `地理位置近` 且 `负载合理` 的 `边缘节点` 上
- 将加速域名 `accelerating-blog.netbeen.top.` 的 DNS 解析用 `CNAME` 记录指向 `CNAME 域名` ： `blog.netbeen.top.SOMETHING.alicdn.com.`
- 将加速域名 `accelerating-blog.netbeen.top.` 告知用户，后续使用加速域名访问本博客

![用户访问被加速的资源时会被引导到物理位置较近的边缘CDN节点](https://cloudflare.com/img/learning/cdn/what-is-a-cdn/what-is-a-cdn.png)

## 实验

百闻不如一见，我们使用 `杭州` 和 `香港` 两个地地域的终端，对淘宝网使用的 CDN `https://g.alicdn.com/` 进行实战分析，看看 CDN 是如何对资源进行加速的

### 杭州节点测试过程

先看一下 `g.alicdn.com` 的解析步骤，经过略显无聊的 `Root Nameserver` 、`.com` `TLD Nameserver` 查询后，返回了一个 `NS` 记录 `ns1.alibabadns.com.` 对应的 IP 为 `106.11.35.19` ， 此 IP 的物理位置在北京。而这个位于北京的 `Nameserver` 回复： `g.alicdn.com.` 的域名被 `CNAME` 到了 `g.alicdn.com.danuoyi.alicdn.com.` 上。第一轮查询结束。

```shell
$ dig +trace g.alicdn.com                                                                 

; <<>> DiG 9.10.6 <<>> +trace g.alicdn.com
;; global options: +cmd
.			656	IN	NS	k.root-servers.net.
# 隐藏部分无关数据
;; Received 239 bytes from 10.86.112.1#53(10.86.112.1) in 48 ms

com.			172800	IN	NS	f.gtld-servers.net.
# 隐藏部分无关数据
;; Received 1172 bytes from 193.0.14.129#53(k.root-servers.net) in 53 ms

alicdn.com.		172800	IN	NS	ns1.alibabadns.com.
# 隐藏部分无关数据
;; Received 821 bytes from 192.35.51.30#53(f.gtld-servers.net) in 303 ms

g.alicdn.com.		600	IN	CNAME	g.alicdn.com.danuoyi.alicdn.com.
;; Received 86 bytes from 106.11.35.19#53(ns1.alibabadns.com) in 44 ms
```

在第一轮里，`g.alicdn.com` 就是一个 `加速域名` ，`g.alicdn.com.danuoyi.alicdn.com.` 就是一个`CNAME 域名` ，该 `加速域名` 被 DNS 通过 `CNAME` 记录指向了 `CNAME 域名`


第二轮查询开始，查询 `g.alicdn.com.danuoyi.alicdn.com.` 的 IP 地址，这次北京的 `Nameserver` `ns1.alibabadns.com.` 返回了一个 `Authoritative Nameserver` `danuoyinewns4.gds.alicdn.com.` ， IP 地址为 `121.43.18.33` ，位于杭州。这个杭州的 `Authoritative Nameserver` 回复了本轮查询最终的6个 `A` 记录，5个位于`湖北武汉` ，1个位于`山东济南`，查询结束

```shell
$ dig +trace g.alicdn.com.danuoyi.alicdn.com.                                   

; <<>> DiG 9.10.6 <<>> +trace g.alicdn.com.danuoyi.alicdn.com.
;; global options: +cmd
.			1468	IN	NS	k.root-servers.net.
# 隐藏部分无关数据
;; Received 239 bytes from 10.86.112.1#53(10.86.112.1) in 56 ms

com.			172800	IN	NS	a.gtld-servers.net.
# 隐藏部分无关数据
;; Received 1191 bytes from 193.0.14.129#53(k.root-servers.net) in 53 ms

alicdn.com.		172800	IN	NS	ns1.alibabadns.com.
# 隐藏部分无关数据
;; Received 840 bytes from 2001:503:a83e::2:30#53(a.gtld-servers.net) in 302 ms

danuoyi.alicdn.com.	86400	IN	NS	danuoyinewns4.gds.alicdn.com.
# 隐藏部分无关数据
;; Received 228 bytes from 106.11.35.18#53(ns1.alibabadns.com) in 40 ms

g.alicdn.com.danuoyi.alicdn.com. 60 IN	A	119.96.88.99
g.alicdn.com.danuoyi.alicdn.com. 60 IN	A	119.96.89.122
g.alicdn.com.danuoyi.alicdn.com. 60 IN	A	106.225.218.251
g.alicdn.com.danuoyi.alicdn.com. 60 IN	A	119.96.75.252
g.alicdn.com.danuoyi.alicdn.com. 60 IN	A	119.96.75.251
g.alicdn.com.danuoyi.alicdn.com. 60 IN	A	119.96.89.123
# 隐藏部分无关数据
;; Received 284 bytes from 121.43.18.33#53(danuoyinewns4.gds.alicdn.com) in 17 ms
```

在第二轮里，最后获取到的6个 `A` 记录，都是阿里云 CDN 的 `边缘节点`

### 香港节点测试过程

测试过程与第一轮一致，先测试 `g.alicdn.com` 的解析步骤，得到了与杭州一致的结果，位于北京， IP 为 `106.11.35.19` 的 `Nameserver` 回复： `加速域名` `g.alicdn.com.` 的域名被 `CNAME` 到了  `CNAME 域名` `g.alicdn.com.danuoyi.alicdn.com.` 上。第一轮查询结束

```shell
$ dig +trace g.alicdn.com

; <<>> DiG 9.16.1-Ubuntu <<>> +trace g.alicdn.com
;; global options: +cmd
.			158027	IN	NS	f.root-servers.net.
# 隐藏部分无关数据
;; Received 262 bytes from 127.0.0.53#53(127.0.0.53) in 4 ms

com.			172800	IN	NS	f.gtld-servers.net.
# 隐藏部分无关数据
;; Received 1172 bytes from 192.5.5.241#53(f.root-servers.net) in 0 ms

alicdn.com.		172800	IN	NS	ns1.alibabadns.com.
# 隐藏部分无关数据
;; Received 821 bytes from 192.35.51.30#53(f.gtld-servers.net) in 52 ms

g.alicdn.com.		600	IN	CNAME	g.alicdn.com.danuoyi.alicdn.com.
;; Received 86 bytes from 106.11.35.19#53(ns1.alibabadns.com) in 44 ms
```

第二轮查询开始，也是查询 `g.alicdn.com.danuoyi.alicdn.com.` ， 这一步的结果与杭州的测试结果不一致了。虽然位于北京的 `Nameserver` `ns1.alibabadns.com.` 也是返回了域名一样的 `Authoritative Nameserver` ，但是解析出的 IP 并不一致，它的 IP 为 `198.11.185.190` ，位于美国堪萨斯州。这个美国的
 `Authoritative Nameserver` 回复了两个 `A` 记录，都位于新加坡，查询结束

```shell
$ dig +trace g.alicdn.com.danuoyi.alicdn.com.

; <<>> DiG 9.16.1-Ubuntu <<>> +trace g.alicdn.com.danuoyi.alicdn.com.
;; global options: +cmd
.			4480	IN	NS	a.root-servers.net.
# 隐藏部分无关数据
;; Received 262 bytes from 127.0.0.53#53(127.0.0.53) in 4 ms

com.			172800	IN	NS	k.gtld-servers.net.
# 隐藏部分无关数据
;; Received 1191 bytes from 198.41.0.4#53(a.root-servers.net) in 48 ms

alicdn.com.		172800	IN	NS	ns1.alibabadns.com.
# 隐藏部分无关数据
;; Received 840 bytes from 192.52.178.30#53(k.gtld-servers.net) in 184 ms

danuoyi.alicdn.com.	86400	IN	NS	danuoyinewns4.gds.alicdn.com.
# 隐藏部分无关数据
;; Received 228 bytes from 198.11.138.254#53(ns1.alibabadns.com) in 160 ms

g.alicdn.com.danuoyi.alicdn.com. 60 IN	A	47.246.1.253
g.alicdn.com.danuoyi.alicdn.com. 60 IN	A	47.246.1.254
# 隐藏部分无关数据
;; Received 232 bytes from 198.11.185.190#53(danuoyinewns4.gds.alicdn.com) in 0 ms
```

第二轮里，返回的两个 `A` 记录是阿里云 `CDN` 位于新加坡的边缘节点。由于位于北京的 `Nameserver` `ns1.alibabadns.com.` 针对两个来自不同地域的 `DNS Query` 返回了不一样的结果，可以推断，`ns1.alibabadns.com.` 在阿里云 CDN 的架构中，是作为 `全局负载均衡器` 使用，拥有根据 `DNS Query` 来源 IP 确定浏览器的地理位置，并且检查符合条件的边缘节点的 Load，最终将地理位置近且负载合理的边缘节点返回的能力。

## CDN 回源

### CDN 多级缓存架构
在 CDN 的实际架构为一棵树，不止有上文提及的 `中心节点` 和 `边缘节点` 两级，还会根据实际需求增设中间层级，形成一个多级缓存的结构，L0 缓存 ~ Ln 缓存，其中 L0 缓存就是 `中心节点` 缓存，Ln 缓存就是 `边缘节点` 缓存。

### CDN 回源的原因及影响

原因：用户访问资源时，永远会被引流到CDN的 `边缘节点`， 如果 `边缘节点` Ln 的缓存没有命中该请求，就会向 上一级缓存 Ln-1 节点请求，依次类推，直至请求至 L0 `中心节点`，如果 `中心节点` 缓存也没有命中，则会想 `源站` 请求，这个过程就是 `CDN 回源`

影响：过多的 CDN 回源会导致源站 QPS 增加，可能会导致稳定性问题。

### 量化回源请求的方法

- 回源请求数量比：回源的请求数量 / 总请求数量
- 回源流量比：回源的请求流量 / 总请求流量

### CDN 回源过多的原因

- 源站动态资源较多，无法缓存
- 缓存策略配置不合理，导致 CDN 节点缓存时间较短或无法缓存
- 资源访问量较低，导致被节点移出缓存

## 参考文献

- [What is a CDN? | How do CDNs work? | Cloudflare](https://www.cloudflare.com/zh-tw/learning/cdn/what-is-a-cdn/)
