---
title: CDN 原理
date: '2021-11-08'
tags: ['cdn', 'network']
category: etc
---

## 前言

你在阅读本文前需要了解的基础知识：计算机网络基础知识、[DNS 工作原理](/posts/dns-introduction/)

## CDN 加速步骤

如果您想使用CDN加速指定网站的业务，则需要将对应的网站作为源站，为其添加加速域名。CDN通过加速域名将源站资源缓存到CDN加速节点，实现资源访问加速。成功添加加速域名后，系统会将相关域名配置下发至全网CDN加速节点，此时不会影响您的现网业务。

成功添加加速域名后，CDN会为您分配一个CNAME域名。您需要在域名解析服务商处将加速域名的DNS解析记录指向CNAME域名，访问请求才能转发到CDN节点上，实现CDN加速。

加速域名：
CNAME域名：
源站域名/IP：

![用户访问CDN资源时会被引导到物理位置较近的节点](https://cloudflare.com/img/learning/cdn/what-is-a-cdn/what-is-a-cdn.png)

## 实验

百闻不如一见，我们以淘宝网使用的 CDN `https://g.alicdn.com/` 进行分析，看看 CDN 是如何对资源进行加速的

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

### 香港节点测试过程

测试过程与第一轮一致，先测试 `g.alicdn.com` 的解析步骤，得到了与杭州一致的结果，位于北京， IP 为 `106.11.35.19` 的 `Nameserver` 回复： `g.alicdn.com.` 的域名被 `CNAME` 到了 `g.alicdn.com.danuoyi.alicdn.com.` 上。第一轮查询结束

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

第二轮查询开始，也是查询 `g.alicdn.com.danuoyi.alicdn.com.` ， 这一步的结果与杭州的测试结果不一致了。虽然北京的 `Nameserver` `ns1.alibabadns.com.` 也是返回了域名一样的 `Authoritative Nameserver` ，但是解析出的 IP 并不一致，它的 IP 为 `198.11.185.190` ，位于美国堪萨斯州。这个美国的
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

## 参考文献

- [What is a CDN? | How do CDNs work? | Cloudflare](https://www.cloudflare.com/zh-tw/learning/cdn/what-is-a-cdn/)
