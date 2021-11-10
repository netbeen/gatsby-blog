---
title: Babel 学习
date: '2021-11-08'
tags: ['cdn', 'network']
category: etc
---

## 前言

你在阅读本文前需要了解的基础知识：计算机网络基础知识、[DNS 工作原理](/posts/dns-introduction/)

## Babel 是什么

Babel是一个工具集，主要用于将 `ES6` 版本的 JavaScript 代码转为 `ES5` 等向后兼容的代码，从而可以运行在低版本的环境中

```javascript
// 转码前
var fn = (num) => num + 2;

// 转码后
var fn = function fn(num) {
    return num + 2;
}
```

## 参考文献

- [Babel教程 - 姜瑞涛的官方网站](https://www.jiangruitao.com/babel/introduction/)
