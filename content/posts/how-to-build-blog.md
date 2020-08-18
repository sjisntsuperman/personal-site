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

最近在腾讯云好像有搞优惠，买服务器赠送.com的域名。不过上面看.com的域名需要23元一年。也还好。

### 第一步：购买域名

购买域名的话，我是直接在*namesilo.com*上面买的，价格不贵，我买的是.cc后缀，当时汇率大约7，也就20块钱左右。之前在freenom上注册过几次，都被说是机器人，不给注册，所以就放弃了。

### 第二步：搭建博客

因为我自己是用hugo搭建的，所以会讲hugo，之前用过hexo，搭建也很简单。不过这里，就只讲hugo好了。

### 第三步：选择一个博客主题

在网站上选择*https://themes.gohugo.io/*，现在用的主题是LoveIt。

```bash
git clone https://..
```

### 第四步：发布内容

hugo新建文件的话，一行代码。

```bash
hugo new posts/first-post.md
```

之后就在md文件里写作就可以了，非常方便。

预览也很简单。也是一行。

```bash
hugo server
```

输入命令后，在浏览器上打开*localhost:1313*即可看到自己的博客。

### 第五步：注册github

因为我这里用的是githubPages来搭建静态博客，所以只介绍githubPages这一种方法，另外还有gitee，ve，
这里就不作比较了，网上的说法是gitee在国内比较快，ve的话比较推荐，githubPages的话访问会比较慢，不过我用了一段时间，发现其实并不慢，满足我的需求，所以就没有去更换了。

### 第六步：dnspod

这里我之前是用腾讯的dnspod来更改dns节点的。直到后来发现cloudflare也有这个功能。

### 第七步：cloudflare

到了这一步，就是更改dns节点，以及挂上https了。cloudflare还可以看到访问流量。按照网上的教程，配置一下就实现了。

### 第八步：修改主题样式

有些博客主题难免会有样式不符合自己审美的情况，这个时候就要学习一点html的知识，去改变样式。其实也很简单。如果不想改的话，直接换主题就是了，我是后者，后来换成了现在这个主题，还凑合把。