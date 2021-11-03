---
title: 前端跨域请求原理
date: '2020-02-16'
tags: ['http', 'cors', 'jsonp']
category: etc
---

## 前言

你在阅读本文前需要了解的基础知识：ajax基础知识

## 同源策略

同源策略，由`Netscape`提出，著名的安全策略。 为了应对CSRF `Cross-site request forgery` ，现在所有支持JavaScript 的浏览器都会使用这个策略。所谓同源是指，域名，协议，端口相同

## JSONP

JSONP = JSON with Padding

### 原理

`<script>` 标签的src属性并不被同源策略所约束，所以可以获取任何服务器上脚本并执行

### 步骤

- 创建一个回调函数
- 用`addScriptTag`等函数，使用script的方式触发调用
- 远程服务中，把JSON 数据作为回调函数的参数，调用回调函数的形式返回，完成回调

### 缺点

- 仅支持 GET ，不支持其他 Method
- 不利于错误处理：调用失败的时候不会返回各种HTTP状态码
- 安全性差：Server 端必须严格限制 `Referer`

## CORS

CORS = Cross Origin Resource Sharing

### 分类

同时满足下面两个条件时，浏览器会直接发送请求，在同一个请求中做跨域权限的验证。（称之为简单请求）

Method是下列之一：
- GET
- HEAD
- POST

请求头中的Content-Type请求头的值是下列之一：

- application/x-www-form-urlencoded
- multipart/form-data
- text/plain

不同时满足上述两个条件的称为非简单请求

### 步骤

#### 简单请求

- 浏览器会直接发跨域请求，并在`Header`中携带`Origin`字段，表明这是一个跨域的请求
- 服务器端接到请求后，会根据自己的跨域规则，通过`Access-Control-Allow-Origin`和`Access-Control-Allow-Methods`响应头，来返回验证结果
- 如果验证成功，则会直接返回访问的资源内容

#### 非简单请求

- 浏览器会先发送 `Preflighted requests`（预先验证请求），`Preflighted requests` 是一个 `OPTION` 请求，用于询问要被跨域访问的服务器，是否允许当前域名下的页面发送跨域的请求。 `OPTIONS` 请求头部中会包含以下头部：`Origin` 、`Access-Control-Request-Method` 、 `Access-Control-Request-Headers`
- 服务器收到OPTIONS请求后，设置Access-Control-Allow-Origin、Access-Control-Allow-Method、Access-Control-Allow-Headers头部与浏览器沟通来判断是否允许这个请求
- 如果Preflighted requests验证通过，浏览器才会发送真正的跨域请求

#### 带认证的请求

- 默认情况下，跨源请求不提供凭据(cookie、HTTP认证及客户端SSL证明等)。通过将`withCredentials`属性设置为true，可以指定某个请求应该发送凭据。xhr.withCredentials = true;
- 如果服务器接收带凭据的请求，会用下面的HTTP头部来响应。`Access-Control-Allow-Credentials: true`
服务器还可以在`Preflight`响应中发送这个HTTP头部，表示允许源发送带凭据的请求


### 优点

克服了JSONP的缺点

- 支持更多的Method
- 更有利于错误处理，支持http错误码
- 更安全

