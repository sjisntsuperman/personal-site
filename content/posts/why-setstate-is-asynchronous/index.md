---
title: "面试官系列之setState为何异步"
subtitle: ""
date: 2020-08-21T04:59:32+08:00
lastmod: 2020-08-21T04:59:32+08:00
author: ""
authorLink: ""
description: ""

tags: []
categories: []

hiddenFromHomePage: false
hiddenFromSearch: false

featuredImage: ""
featuredImagePreview: ""

toc:
  enable: true
math:
  enable: false
lightgallery: false
license: ""
---

## 前言

前段时间电话面试的时候面试官问了我这个问题，我当时听错了，以为问的是setState什么情况下是异步的，所以只答到了第二步，后面回想起来，好像是自己听错了，回到正题。  

为了回答这个问题，我们要搞清楚，

- setState真的是异步的吗
- setState在什么情况下是异步的
- 什么情况下能拿到最新的state

## setState真的是异步吗

首先先说结论，`setState不是异步的`。不过它看起来是异步的。为何这样说呢？

因为react在render的时候会进入一个事务，在这个事务下，会批量更新state，在批量更新的途中如果遇到setState更改相同的键值，后者会覆盖掉前者， 这些操作让setstate看起来是异步的。

## setState什么情况下是异步的

首先，`在合成事件和生命周期中是异步的，在原生事件和settimeout中是同步的`，
源码中，react通过`isBatchingUpdate`来判断setState是否为异步的。`isBatchingUpdate`默认是false的，
借用下别人的图，从图中可以很明显的看到，在合成事件和生命周期阶段，isBatchingUpdate更新为true，
进入批量更新的事务，同时setState在这个阶段中触发。因此说明了setState在这个阶段中是异步的。

（这里应该有图）

## 什么情况下拿到最新的state

`通过setState中第一个参数通过回调的形式能拿到最新的state`。为何能拿到呢？因为源码中对参数的类型做了判断，如果是函数的话，就会给最新的更改后的state，而如果不是的话，会进入批量更新的事务中。

（这里应该有部分判断的源码）