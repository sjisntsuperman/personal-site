---
title: "用github action部署hugo blog"
date: 2020-08-19T01:06:26+08:00
tags: []
featured_image: ""
description: ""
summary: ""
draft: true
---

> 最近看到了github出了个github action的CI工具，感觉挺方便的，于是想试用下。

## github action

首先创建.github目录，在目录下新建workflow，创建个文件`mail.yml`，我们在这里创建我们的workflow。

github action介绍的文章很多了。这里就不赘述了。

说下如何部署hugo的CICD流。

首先通过

1. xxx，用来安装hugo
2. xxx，用来把指定资源发布到指定分支

分别建立github-action。下面分步解析一下github action实现了什么功能。

## checkout

这一步主要是用来从指定分支上拉取资源。

## build

这一步主要是通过hugo的命令来部署静态资源。

## deploy

这一步主要是通过前面提到的action发布资源到指定分支上。
