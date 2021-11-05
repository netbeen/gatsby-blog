---
title: Gatsby 初体验
date: '2021-11-03'
tags: ['gatsby', 'vercel', 'ssg']
category: etc
---

## 前言

你在阅读本文前需要了解的基础知识：Node.js 基础知识、React 基础知识、WEB 开发基础知识。

## Gatsby 是什么？

[Gatsby](https://gatsbyjs.com/) 是一个使用 React 作为基础框架，使用 GraphQL 作为数据层的 SSG `Static Site Generation`，它的同级竞品还有 [Next.JS](https://nextjs.org/) 、 [Hugo](https://gohugo.io/) 、 [Jekyll](https://jekyllrb.com/) 、[Nuxt.JS](https://nuxtjs.org/) （这些都暂时还没有使用过，留坑待填）， Gatsby 使用的技术栈对于 React 系的前端十分友好，上手很容易。

### 引申阅读：现代 WEB 渲染方式的演进 CSR -> SSR -> SSG

所谓现代，即抛开了远古时期的纯静态页面、中古时期的 JSP 之辈、近代时期的 jQuery 等非 MVVM 渲染方式

CSR `Client Side Rendering` : 整个页面都是由浏览器渲染的，script 加载完成后通过 ajax 获取数据，渲染DOM

优点： 
- 功能强大：几乎可以胜任任何 WEB 业务场景

缺点： 
- 不利于 SEO：传统爬虫完全无效，虽然后续的出现了可以运行 js 的爬虫，但是由于复杂度高，效果依旧不理想
- 首屏性能差：首屏渲染依赖 script 下载&执行，对于弱网环境和弱终端不友好，FCP `First Contentful Paint` 较长
- 运维负担：CSR 需要一个 Server 配合，需要处理安全补丁、非法入侵等运维事项

SSR `Server Side Rendering` : 首屏由 Server 渲染的，script 加载完成后接管路由，处理后续的页面跳转、数据拉取等工作

优点：
- 功能强大：同 CSR 优点1
- 对 SEO 友好：Server 直接返回带内容的 html，方便爬虫获取数据
- 首屏性能好：Server 直接返回带内容的 html ，浏览器直接渲染，省去了浏览器渲染的时间和开销

缺点：
- 运维负担：同 CSR 缺点3
- 实现复杂
- Server 性能瓶颈：由于 Server 在处理业务逻辑的同时，还需要处理页面渲染，业务高峰时容易形成性能瓶颈

PS: `SSR` 对于数据变化缓慢甚至不变化的站点，比如文档类站点，存在很大的性能浪费，因为服务端渲染出来页面，很有可能是相同的，所以引申出 `SSG` 的概念

SSG: 所有页面在 CI `Continuous integration` 阶段已经渲染完成，存储在 CDN `Content Delivery Network` 上，Server 通过路由规则反向代理，返回 URL 对应的页面

优点：
- 对 SEO 友好：同 CSR 优点2
- 首屏性能极好：Server 直接将预渲染的页面返回，相对于 SSR 性能更快
- 无运维负担：Server 不承担业务逻辑，无DB，安全性好，成本低
- 全球访问：页面部署在 CDN 上，借助于 CDN 的特性，提升性能 & 稳定性

缺点：
- 实现复杂
- 适用范围受限：不适用于内容快速变化的业务场景如微博、论坛；特别适用于博客、文档站

## 生成第一个 Gatsby 站点

本节内容源于 [Gatsby Docs](https://www.gatsbyjs.com/docs/) ，如有疑问可以访问官方文档查看更多细节

```shell
$ npm install -g gatsby-cli # 安装 gatsby-cli
$ gatsby new YOUR_BLOG_NAME https://github.com/gatsbyjs/gatsby-starter-blog # 使用名为 gatsby-starter-blog 的 Starter 初始化工程
$ cd YOUR_BLOG_NAME/
$ npm run develop # 本地开发 & 预览
$ npm run build # 生成静态页面
```

`gatsby-cli` 是 Gatsby 官方提供的命令行工具，提供了 `交互式` 和 `使用Starter` 两种方式创建 Gatsby 应用，starter 是快速创建 Gatsby 的模板工程，根据维护者不同，分为 [官方 Starter](https://github.com/gatsbyjs?q=starter) 和 社区 Starter，本例使用官方 Starter ，执行 `npm run build` 后，会在 `public/` 文件夹下生成本站点的所有静态页面

## 将 Gatsby 站点部署到 Server 上

将静态页面部署到 Server 上，传统步骤如下

- 购买一台云服务器
- 安装 `nginx` ，设置好反向代理配置
- 将静态站点部署到服务器上
- （如果要做的更好）还要绑定域名，再需要申请 `SSL证书` ，以支持 `https` 方式访问

不过现在有更好的选择：可以使用支持`静态页面托管`的云平台（目前用户量较大的是 [Gatsby Cloud](https://www.gatsbyjs.com/products/cloud/) 和 [Vercel](https://vercel.com/) ），更方便，更实惠，性能更好。与 `Vercel` 相比，目前 `Gatsby Cloud` ，功能和扩展性较弱，所以我选择以 `Vercel` 作为托管平台

| 特性                                   | Vercel   | Gatsby Cloud   | 
| ------------------------------------------- | --------- | ---------- |
| 底层架构                | 多平台   | AWS        |
| CI与Github集成              | 支持        | 支持       |
| 任意版本回滚         | 支持     |      |
| 兼容主流SSG                 | 支持  |  |
| 团队协作                 | 支持  |  |
| 自定义域名                 | 支持  | 支持 |
| 自动HTTPS                 | 支持  | 支持 |
| 免费套餐 CI 时限                 | 6000 min/月  | 无限 |
| 免费套餐流量限制                 | 100 GB/月  | 100 GB/月 |
| Serverless Function                 |  支持 | 支持 |
| 边缘计算                 | 支持  |  |

部署应用到 `Vercel` 操作也很简单：
- 登录 [https://vercel.com/](https://vercel.com/)
- New Project
- Import Git Repository
- 选择 Gatsby 站点工程，等待 CI 结束
- 此时站点已经可以用过 Vercel 的三级域名 *.vercel.app 访问
- 设置新站点 dns 解析，添加 cname 记录指向 Vercel 源，等待自动化申请 SSL 证书
- DONE

至此，你的站点已经被部署到 Internet 上了。使用 `SSG` + `页面托管平台` 的方案，非常适合个人主页、博客等，省心省力，YYDS

## 参考文献

- [Gatsby Docs](https://www.gatsbyjs.com/docs/)
- [Vercel vs Gatsby Cloud - Bejamas](https://bejamas.io/compare/vercel-vs-gatsby-cloud/)
