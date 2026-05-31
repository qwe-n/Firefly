---
title: Cloudflare Worker 优选
published: 2026-04-25
updated: 2026-05-31
description: 通过 Cloudflare Worker 优选让你的网站解析到国内访问更快的 IP，提升访问速度
image: './images/cfwoker.png'
tags: [cloudflare, Worker, ip优选]
category: 'cloudflare'
draft: false
lang: 'zh-CN'
---

## 什么是优选

说白了就是**挑一个国内访问更快的 Cloudflare 节点。**

Cloudflare 默认分配给你的 IP，国内访问延迟可能很高，甚至直接打不开。通过优选，我们可以手动把域名解析到那些国内访问快的 Cloudflare IP 上，速度会明显提升。

要实现优选，核心就两件事：自己控制路由规则 + 自己控制 DNS 解析。

## 寻找优选域名

可以用社区大佬做的[这个网站](https://cf.090227.xyz/)来获取优选域名。

## Worker 优选方法

打开你的 Worker 项目，添加一条路由，路由格式填 `你的域名/*`：

![路由示例](./images/cf_2.png)

然后去域名 DNS 设置里加一条记录，类型选 CNAME。名称根据你路由里填的内容来定：比如你路由填的是 `blog.example.com`，这里名称就填 `blog`（前提是域名 `example.com` 托管在 Cloudflare 上）。如果路由填的域名跟你托管的域名一样，直接填 `@`。最后目标填从社区网站复制下来的优选域名就行。

![DNS记录示例](./images/cf_1.png)

> [!tip]
> 优选的前提是你有一个域名托管在 Cloudflare 上。
> 路由里记得在域名后面加 `/*`，不然优选只对根域名生效。
