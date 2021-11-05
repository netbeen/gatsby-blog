---
title: DNS 工作原理
date: '2021-11-05'
tags: ['dns', 'network']
category: etc
---

## 前言

你在阅读本文前需要了解的基础知识：计算机网络基础知识

## 什么是 DNS

DNS `Domain Name System` 是 Internet 里的黄页，最重要的职责是将 `domain name` 映射为 `IP address` ，以避免通信过程中，人类需要记住不那么好记的 `IP address` 

## 域名层次

以笔者的博客 URL `https://blog.netbeen.top/` 为例，表面上 `domain name` 是 `blog.netbeen.top` ，而实际的 `domain name` 是 `blog.netbeen.top.` ，最后的 `.` 表示 `root` ， 是整个 `domain name tree` 的根。 `domain name` 的级别如下：

```text
blog.netbeen.top.
三级域名.二级域名.顶级域名.根
```

## DNS Server 类型

所有的 DNS Server 都可以按照功能划分为以下四类：

### Recursive Resolver

`Recursive Resolver` 是 Client 发起 `DNS query` 的第一处理人，大家在 PC 中配置的 DNS 地址都是指向一个 `Recursive Resolver` ，它可以由 ISP 维护（比如中国电信、中国移动），也可以由第三方维护（ Cloudflare 、 Google ）

`Recursive Resolver` 接收来自 Client 的 `DNS query` 后，会屏蔽中间的实现细节（用迭代的方式自顶向下查询，查询到有效记录或者报错，通常还会将查询结果缓存，以提升后续查询的性能），返回 `DNS response` 给 Client

![Recursive Resolver 的工作方式](https://www.cloudflare.com/img/learning/dns/dns-server-types/recursive-resolver.png)

### Root Nameserver

`Root Nameserver` 是所有 `Recursive Resolver` 发起查询的第一站，`Root Nameserver` 并不会返回任何的最终结果，它只会根据 `DNS query` 的 TLD `Top-Level Domain` 信息，返回对应 `TLD Nameserver` 的地址，由 `Recursive Resolver` 再次发起向 `TLD Nameserver` 的查询。

全球一共有 13 个 `Root Nameserver` ，每个 `Root Nameserver` 都是一个服务器的集群，以处理全球的海量 `DNS query` 。这些 `Root Nameserver` 都由 ICANN `Internet Corporation for Assigned Names and Numbers` 负责管理

下面是使用 `dig` 命令查询全球 13 个 `Root Nameserver` 的结果，在没有缓存的情况下， `Recursive Resolver` 会同时向这 13 个 `Root Nameserver` 发起请求，并记录下返回最快的那个，后续的请求都会使用该 `Root Nameserver`

```shell
$ dig . NS                                                                              

; <<>> DiG 9.10.6 <<>> . NS
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 43273
;; flags: qr rd ra; QUERY: 1, ANSWER: 13, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1408
;; QUESTION SECTION:
;.				IN	NS

;; ANSWER SECTION:
.			3407	IN	NS	e.root-servers.net.
.			3407	IN	NS	h.root-servers.net.
.			3407	IN	NS	g.root-servers.net.
.			3407	IN	NS	c.root-servers.net.
.			3407	IN	NS	i.root-servers.net.
.			3407	IN	NS	m.root-servers.net.
.			3407	IN	NS	d.root-servers.net.
.			3407	IN	NS	k.root-servers.net.
.			3407	IN	NS	b.root-servers.net.
.			3407	IN	NS	j.root-servers.net.
.			3407	IN	NS	f.root-servers.net.
.			3407	IN	NS	a.root-servers.net.
.			3407	IN	NS	l.root-servers.net.

;; Query time: 56 msec
;; SERVER: 10.86.112.1#53(10.86.112.1)
;; WHEN: Fri Nov 05 13:24:06 CST 2021
;; MSG SIZE  rcvd: 239
```

假设我们的 `Recursive Resolver` 使用了 `e.root-servers.net.` 作为首选，向这个 `Root Nameserver` ，发起了如下的查询请求：

```shell
$ dig @e.root-servers.net.  blog.netbeen.top                                             

; <<>> DiG 9.10.6 <<>> @e.root-servers.net. blog.netbeen.top
; (2 servers found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 12793
;; flags: qr rd; QUERY: 1, ANSWER: 0, AUTHORITY: 8, ADDITIONAL: 9
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1472
;; QUESTION SECTION:
;blog.netbeen.top.		IN	A

;; AUTHORITY SECTION:
top.			172800	IN	NS	a.zdnscloud.com.
top.			172800	IN	NS	b.zdnscloud.com.
top.			172800	IN	NS	c.zdnscloud.com.
top.			172800	IN	NS	d.zdnscloud.com.
top.			172800	IN	NS	f.zdnscloud.com.
top.			172800	IN	NS	g.zdnscloud.com.
top.			172800	IN	NS	i.zdnscloud.com.
top.			172800	IN	NS	j.zdnscloud.com.

;; ADDITIONAL SECTION:
a.zdnscloud.com.	172800	IN	A	203.99.24.1
b.zdnscloud.com.	172800	IN	A	203.99.25.1
c.zdnscloud.com.	172800	IN	A	203.99.26.1
d.zdnscloud.com.	172800	IN	A	203.99.27.1
f.zdnscloud.com.	172800	IN	A	114.67.16.204
g.zdnscloud.com.	172800	IN	A	42.62.2.16
i.zdnscloud.com.	172800	IN	AAAA	2401:8d00:1::1
j.zdnscloud.com.	172800	IN	AAAA	2401:8d00:2::1

;; Query time: 197 msec
;; SERVER: 192.203.230.10#53(192.203.230.10)
;; WHEN: Fri Nov 05 13:38:18 CST 2021
;; MSG SIZE  rcvd: 338
```

可见，返回了 `a.zdnscloud.com.` 等 `.top` 域名的 `TLD Nameserver` 的 IP 地址， 这样  `Recursive Resolver` 就可以继续发起下一级 `DNS query` 了

### TLD Nameserver

`TLD Nameserver` 维护着整个 `TLD` 内的 域名 - IP 的映射关系，它是由 ICANN 的一个分支机构 IANA `Internet Assigned Numbers Authority` 负责管理。

继续上文的例子，如果 `Recursive Resolver` 选择了 `a.zdnscloud.com.` 作为 `.top` 域名的首选 `TLD Nameserver` ，那么会向它发出如下请求：

```shell
$ dig @a.zdnscloud.com.  blog.netbeen.top  
                                                   
; <<>> DiG 9.10.6 <<>> @a.zdnscloud.com. blog.netbeen.top
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 25218
;; flags: qr rd; QUERY: 1, ANSWER: 0, AUTHORITY: 2, ADDITIONAL: 1
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;blog.netbeen.top.		IN	A

;; AUTHORITY SECTION:
netbeen.top.		3600	IN	NS	dns30.hichina.com.
netbeen.top.		3600	IN	NS	dns29.hichina.com.

;; Query time: 459 msec
;; SERVER: 203.99.24.1#53(203.99.24.1)
;; WHEN: Fri Nov 05 13:58:44 CST 2021
;; MSG SIZE  rcvd: 96
```

由于笔者的域名 `netbeen.top` 是在阿里云购买的，所以 `TLD Nameserver` 返回的结果并没有直接返回结果，而是又返回了一个 `NS` 记录，指向了阿里云维护的 `Authoritative Nameserver`

### Authoritative Nameserver

此时 `Recursive Resolver` 继续向 `dns29.hichina.com.` 查询，得到如下的结果：

```shell
$ dig @dns29.hichina.com.  blog.netbeen.top                                              

; <<>> DiG 9.10.6 <<>> @dns29.hichina.com. blog.netbeen.top
; (9 servers found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 15688
;; flags: qr aa rd; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;blog.netbeen.top.		IN	A

;; ANSWER SECTION:
blog.netbeen.top.	600	IN	CNAME	cname.vercel-dns.com.

;; Query time: 57 msec
;; SERVER: 140.205.41.29#53(140.205.41.29)
;; WHEN: Fri Nov 05 14:07:54 CST 2021
;; MSG SIZE  rcvd: 79
```

由于笔者的博客是托管在 `Vercel` 上，所以 `Authoritative Nameserver` 没有直接返回 `A` 记录，返回了一个 `CNAME` 记录，指向了 `cname.vercel-dns.com.`

由于最终得到的是一个 `CNAME` 记录，无法直接访问，所以 `Recursive Resolver` 只能再次重新发起查询（从 `Root Nameserver` 开始再来一遍，过程我就不再赘述了），跳过中间的步骤，最终得到如下的查询结果：

```shell
$ dig cname.vercel-dns.com.                                                              

; <<>> DiG 9.10.6 <<>> cname.vercel-dns.com.
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 50871
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1408
;; QUESTION SECTION:
;cname.vercel-dns.com.		IN	A

;; ANSWER SECTION:
cname.vercel-dns.com.	15	IN	A	76.223.126.88

;; Query time: 65 msec
;; SERVER: 10.86.112.1#53(10.86.112.1)
;; WHEN: Fri Nov 05 14:11:32 CST 2021
;; MSG SIZE  rcvd: 65
```

最终本次查询完成 `76.223.126.88` 即为笔者博客的对应 Server 的IP地址 

## DNS 记录类型 

常见的 DNS 记录类型主要有如下几类：

- A `Address` ：地址记录，返回域名指向的IP地址
- NS `Name Server` ：返回下一级 `DNS Server` 地址，该记录只能设置为域名，不能设置为IP地址
- MX `Mail eXchange` ：返回邮件服务的服务器地址（由于早起邮件服务是很重要的Internet服务，所以单拎出来）
- CNAME `Canonical Name` ：返回别名，即当前查询的域名结果等同于另一个域名

## 参考文献

- [什么是 DNS_DNS的工作方式_DNS高速缓存的工作方式| Cloudflare 中国官网 | Cloudflare](https://www.cloudflare.com/zh-cn/learning/dns/what-is-dns/)
- [DNS server types | Cloudflare](https://www.cloudflare.com/learning/dns/dns-server-types/)
