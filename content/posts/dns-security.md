---
title: DNS 安全协议
date: '2021-11-06'
tags: ['dns', 'network', 'security']
category: etc
---

## 前言

你在阅读本文前需要了解的基础知识：计算机网络基础知识、[DNS 工作原理](/posts/dns-introduction/) 、[DNS 攻击原理](/posts/dns-attack/)

## DNS 安全协议为了什么？

由于 DNS 原始协议设计不包含任何安全细节，为了夺回被 DNS 攻击剥夺的 `Confidentiality` 和 `Integrity` ，逐渐形成了各种 DNS 安全协议，守护信息安全

## DNSSEC

DNSSEC `DNS Security Extensions` ，DNSSEC 是目前最大规模部署的 DNS 安全协议（目前已经在 `Root Nameservers` 和大部分 `TLD Nameservers` 完成部署），它通过在 `DNS Response` 里新增`RRSIG` 、 `DNSKEY` 、 `DS` 等字段，包含了数字签名，保证了包内数据的 `Integrity`，但是这个协议完全没有办法保护 `Confidentiality`

## DNSCrypt

`DNSCrypt` 完全推翻了 DNS 协议规范，重新设计，将数据进行加密和签名，这样就同时保证了 `Integrity` 和 `Confidentiality` ， 但是可能是因为改动过大，导致未被规范化成 RFC 标准，所以各方的支持和兼容的力度都不是很大 

## DNS-over-TLS

DoT `DNS-over-TLS` ，不同于原始 DNS 使用裸 `UDP` 作为底层协议，DoT 使用了 `TCP` + `TLS` ，借助于 `TLS` 的能力，天然就保证了 `Integrity` 和 `Confidentiality`

## DNS-over-HTTPS

DoH `DNS-over-HTTPS` ， DoH 就借力更加彻底了，直接使用 HTTPS 作为底层协议，这样借助于 `TCP` + `TLS`
+ `HTTPS` 三重 Buff 加成，也是天然就保证了 `Integrity` 和 `Confidentiality` 。相比 DoT ，DoH 因为多封装了一层流行协议 HTTP ，所以性能更差，但实现难度更低

### Firefox 启用 DoH

设置 -> 常规 -> 网络设置 -> 设置... -> 勾选 启用HTTPS的DNS 并选择 Cloudflare

### 如何验证 DoH 是否生效

访问 [https://1.1.1.1/help](https://1.1.1.1/help) ，会看到如下输出，其中第二行内容即 DoH 是否生效的检测结果

```text
Debug Information
Connected to 1.1.1.1        Yes
Using DNS over HTTPS (DoH)	Yes
Using DNS over TLS (DoT)    No
Using DNS over WARP         No
AS Name                     Cloudflare
AS Number                   13335
Cloudflare Data Center      LAX
```

## 总结

| 特性                   | DNSSEC   | DNSCrypt  |  DNS-over-TLS | DNS-over-HTTPS |
| ------ | --------- | ---------- |--- | ---|
| 保护 Confidentiality    |     | Y   | Y | Y |
| 保护 Integrity          | Y   | Y   | Y | Y |
| 有 RFC 协议规范          | Y   |     | Y | Y |
| 公共 DNS 支持（Cloudflare）   | Y   |     | Y | Y |
| 公共 DNS 支持（Google）   | Y   |     | Y | Y |
| 公共 DNS 支持（Open DNS）  | Y   |  Y   |  | Y |

## 参考文献

- [DNS security | Cloudflare](https://www.cloudflare.com/learning/dns/dns-security/)
