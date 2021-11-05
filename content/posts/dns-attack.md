---
title: DNS 攻击原理
date: '2021-11-05'
tags: ['dns', 'network', 'security']
category: etc
---

## 前言

你在阅读本文前需要了解的基础知识：计算机网络基础知识、[DNS 工作原理](/posts/dns-introduction/)

## 历史原因

由于 DNS 协议制定的年代比较早， [RFC 1034](https://datatracker.ietf.org/doc/html/rfc1034) 早在1987年就发布了，后续作为 Internet 的基础设施，出于向前兼容的考虑，也没有办法做大幅度的改变。导致 DNS 协议带有如下先天问题，为后来的攻击者提供了便利：

- DNS 作为应用层协议，它的传输层使用的是 `UDP` 协议，是一种 **不可靠、无连接** 的协议
- DNS 应用层协议包内的数据都以 **明文** 传输

从信息安全的角度，攻击一般都是用来破坏信息的完整性 `Integrity` 或保密性 `Confidentiality` ，即篡改信息和偷看信息。目前，被使用比较多的 DNS 攻击手段也恰恰攻击这两个点

## DNS 监控

DNS 监控实现难度很低，由于 DNS 的应用层协议包内的数据是明文传递的，对任何报文途径的各个网络设置都是可见的，所以只需要在网络链路上的任意一个节点，部署数据包解析以及数据收集的软件即可实现 DNS 监控，这项技术破坏的是 `Confidentiality`

## DNS Poisoning

由于 DNS 的传输层协议是 `UDP` ，是一种不可靠、无连接的协议，`DNS Clint` 发起 `DNS Query` 后，就等待第一个返回的有效数据包，而抛弃了后续的数据包，这就给了 `DNS Poisoning` 可以施展的空间

需要实现 `DNS Poisoning` 可以在整个网络的链路任意一个节点，部署数据包解析软件，一旦接收到了 `DNS Query` 的数据包（或者进一步解析，命中更细化更具体的规则），就立即返回一个伪造的 `DNS Response` 给 `DNS Client` ，这样 `DNS Client` 就会信任这个数据包，使用了错误的 IP 地址，至此 `DNS Poisoning` 成功，这项技术破坏的是 `Integrity`

PS：如果实施 `DNS Poisoning` 的是一台 `DNS Server` ，那就更简单了，因为通常 `DNS Server` 都会将解析结果缓存下来，仅需修改缓存里的结果为伪造的IP地址即可，所以这样的技术也被称为 `DNS Cache Poisoning`

## DNS Hijacking

与 `DNS Poisoning` 有些类似，甚至笔者认为两者的边界有一些模糊，`DNS Hijacking` 的目标也是希望返回一个伪造的 IP 地址给 `DNS Client` ，只是实施 `DNS Hijacking` 的只能是 `DNS Server` 甚至是 `Authoritative Nameserver` 实施层级越高的 `DNS Hijacking` 影响范围越大。

此类攻击的实现原理是：当该 `DNS Server` 收到 `DNS Query` 后，如果命中了特定的规则，则返回一个伪造的 IP 地址回去， 至此 `DNS Hijacking` 成功

![DNS Hijacking 工作原理](https://www.cloudflare.com/img/learning/dns/dns-security/dns-hijacking.png)

## DNS 攻击的危害

- 遭遇破坏 `Confidentiality` 的攻击时，用户的访问信息会被窃取
- 遭遇破坏 `Integrity` 的攻击时，用户会无法访问到想去的站点（无法获取对应信息），甚至访问到一个高仿的钓鱼网站（如果在钓鱼网站上填写信息，会造成更大的信息安全损失）

PS：往往这两种攻击手段会叠加使用，杀伤力更大

## 参考文献

- [What is DNS cache poisoning? | DNS spoofing | Cloudflare](https://www.cloudflare.com/learning/dns/dns-cache-poisoning/)
