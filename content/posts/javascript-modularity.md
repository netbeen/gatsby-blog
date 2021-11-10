---
title: JavaScript 模块化方案
date: '2021-11-10'
tags: ['javascript']
category: etc
---

## 前言

你在阅读本文前需要了解的基础知识：JavaScript 基础知识

## 历史

JavaScript 与其他大部分现代开发语言不同，由于早期的 javaScript 不是面向工程化思维设计的语言，而是仅用于配合 HTML 的动态修改、特效创作等用途，所以并没有和其他语言类似，在创建之初就设计了官方的模块化方案。近些年尤其是 Node.js 用量起飞后，JavaScript 早已脱离了原来的领域，成为了前端工程里的绝对主力，甚至渗透进了移动端、服务端领域，这样就对 JavaScript 的模块化方面提出了要求，于是产生了如下方案：

## CJS

CJS `CommonJS` 的目的是允许 JavaScript 在浏览器环境之外创建模块

- 时间线：在 2009-01 由 Kevin Dangoor 发起，在 2013-05 被Node.js官方放弃
- CJS 导入模块为 **同步** 导入
- CJS 导入后的模块是目标模块的 **拷贝**
- CJS 无法在浏览器里直接使用，主要使用在 Node.js 运行时里

```javascript
// CJS 代码示例
// importing 
const doSomething = require('./doSomething.js'); 

// exporting
module.exports = function doSomething(n) {
  // do something
}
```

## AMD

AMD `Asynchronous Module Definition` 的目的是异步加载模块，使得加载过程不会阻碍其他任务执行。由于 AMD 不是 JavaScript 原生支持的，所以使用 AMD 模块需要使用对应库函数，比如 `RequireJS`，实际上 AMD 就是 `RequireJS` 在推广过程中对模块定义的规范化的产出

- AMD 导入模块为 **异步** 导入
- AMD 主要使用在 浏览器 运行时里

```javascript
// AMD 代码示例 * 2
// AMD 推荐写法
define(['dep1', 'dep2'], function (dep1, dep2) {
    // Define the module value by returning a value.
    return function () {};
});

// RequireJS 2.0 也支持类似 CMD 的写法
define(function (require) {
    var dep1 = require('dep1'),
        dep2 = require('dep2');
    return function () {};
});
```

## CMD

CMD `Common Module Definition` 规范主要是 `Sea.js` 推广中形成的

- CMD 主要在浏览器中运行，也可以在 Node.js 运行时中运行
- CMD 导入模块为 **异步** 导入 （与 AMD 一致）
- 与 AMD 在写法上的不同：AMD 推崇依赖前置、提前执行，CMD推崇依赖就近、延迟执行，可以看 AMD 第二个代码举例了解一个大概
- CMD 目前已经很少被使用了，就不上代码举例了

## UMD

UMD `Universal Module Definition` 提出的目的是使模块可以同时被浏览器和Node.js使用，即 `CJS` 和 `AMD` ，同时还支持最古老的全局变量规范

- UMD 更像是一个兼容包，让同一个模块既可以在前端使用，也可以在后端使用
- 当使用 `Rollup` / `Webpack` 之类的打包器时，UMD 通常用作备用模块

```javascript
// UMD 代码示例
(function (root, factory) {
    if (typeof define === "function" && define.amd) {
        define(["jquery", "underscore"], factory);
    } else if (typeof exports === "object") {
        module.exports = factory(require("jquery"), require("underscore"));
    } else {
        root.Requester = factory(root.$, root._);
    }
}(this, function ($, _) {
    // this is where I defined my module implementation

    var Requester = { // ... };

    return Requester;
}));
```

## ESM

ESM `ES Modules` 最新最标准最熟悉的方案了吧？

- 时间线：于 2015-06 跟随者 ES6 的发布而提出
- ESM ，是一个标准化解决方案，可以同时使用在前端和后端
- 得益于 ES6 的静态模块结构，可以进行 `Tree Shaking`
- 除了 IE 等远古浏览器外，现代浏览器均已支持

```javascript
// ESM 代码示例
// 导入模块
import {foo, bar} from './myLib';

// 导出模块
export default function() { /* do some stuff */ };
export const function1() { /* do some stuff */ };
export const function2() { /* do some stuff */ };
```

```html
<script type="module">
  import {func1} from 'my-lib';

  func1();
</script>
```

## 总结

- ESM 语法最简单、最标准化，我认为是最好的模块化方案（待完全普及以后）
- UMD 是目前兼容性最好的方案，通常作为 ESM 的备胎
- CJS 是同步的，适合后端
- AMD 是异步的，适合前端

## 参考文献

- [What Are CJS, AMD, UMD, and ESM in Javascript?](https://irian.to/blogs/what-are-cjs-amd-umd-and-esm-in-javascript/)
