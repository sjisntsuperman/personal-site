---
title: "How to Build Blog"
date: 2020-06-26T04:56:24+08:00
featured_image: "images/built-blog/built-blog.jpg"
tags: ["record"]
description: "We'd love to hear from you"
summary: '记录一下自己搭建博客的点滴。'
---

---

> 一天，小明逛论坛的时候无意点开别人的blog，好漂亮，我也想整一个！之后，一篇踩坑文章指北就此诞生。

首先，推荐区*wordpress*上看自己喜欢的主题，然后根据官网的教程，搭建博客。

域名的话，如果不想花钱，可以到*freenom.com*上面购买。

如果想要形如 *.com*后缀的话，则需要花钱，推荐去*namesilo.com*上购买。

### 第一步：购买域名

购买域名的话，我是直接在*namesilo.com*上面买的，价格不贵，我买的是.cc后缀，当时汇率大约7，也就20块钱左右。之前在freenom上注册过几次，都被说是机器人，不给注册，所以就放弃了。

### 第二步：搭建博客

有折腾想法的朋友，推荐用hugo。网上教程也很多。没有的话，用wordpress一步到位就好，插件也很丰富。

### 第三步：选择一个博客主题

在网站上选择*https://themes.gohugo.io/*

### 第四步：发布内容

因为我是用hugo搭的， 命令很简单。就一行代码。

```bash
hugo new posts/first-post.md
```

预览也很简单。也是一行。

```bash
hugo server
```

输入命令后，在浏览器上打开*localhost:1313*即可看到自己的博客。

### 第五步：注册github

githubPages

### 第六步：dnspod

todo

### 第七步：cloudflare

todo

### 第八步：修改主题样式

todo