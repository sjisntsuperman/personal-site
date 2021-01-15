---
title: "前端自测清单(前端八股文)"
subtitle: ""
date: 2020-11-13T17:58:41+08:00
lastmod: 2020-11-13T17:58:41+08:00
draft: false
author: "steinw"
authorLink: "steinw.cc"
description: ""

tags: [
  "web"
]
categories: []

hiddenFromHomePage: false
hiddenFromSearch: false

featuredImage: "featured-image.jpg"
featuredImagePreview: ""

toc:
  enable: true
math:
  enable: false
lightgallery: false
license: ""
---

<!--more-->
## 缘起

这篇文章主要列举一些自己想到的面试题目，让大家更加熟悉前端八股文。

先从性能优化开始吧。性能优化大体可以分为两个，

- 运行时优化
- 加载时优化

## 加载时优化

### 网络优化

- dns寻址过程
- tcp的三次握手和四次挥手，以及为何要三次和为何要四次
- https的握手过程，以及对称加密和非对称加密的区别，什么是中间人劫持，ca证书包括哪些内容
- http1.0，http1.1以及http2.0的区别，多路复用具体指的是什么，keep-alive具体如何体现
- cdn的原理，cdn什么情况下会回源，cdn的适用场景
- 浏览器缓存有哪几种，它们的区别是什么，什么时候发生缓存，如何决定缓存哪些文件
- 了解过websocket么，解释一下websocket的作用

### 渲染优化

- 关键渲染路径优化，什么是关键渲染路径，分别如何优化
- 优化体积，webpack的分包策略，如何配置优化，如何提高构建速度，tree-shaking是什么
- cssom 的优化，以及html解析过程中，遇到哪些tag会阻塞渲染
- 雅虎军规说，css尽量放到head里，js放到下方，那么移动端适配的flexiblejs为何要放到css上方呢
- 影响回流重绘的因素有哪些，如何避免回流，以及bfc是什么，bfc有什么特性，清除浮动的原理是什么

### 场景：如何优化首屏

除了上以及下面说到的，这里也是分两个层面，

- 加载时优化
- 运行时优化

#### 加载

- 首屏请求和非首屏请求拆分
- 图片都应该使用懒加载的形式加载
- 使用preload预加载技术，以及prefetch的dns预解析
- 与首屏无关的代码可以加async甚至是defer等待网页加载完成后运行

#### 运行

这里跟加载的异常耦合，另作分析吧。

## 运行时优化

- 虚拟长列表渲染
- 图片懒加载
- 使用事件委托
- react memo以及pureComponent
- 使用SSR
- 。。。

以及一些比较骚的操作，只能特定场景使用，

- serviceWorker劫持页面
- 利用worker

---

更新一波，性能优化之外的面试题，

## 底层

- V8是如何实现GC的
- JS的let，const，call stack，function context，global context。。。的区别
- this的指向，箭头函数中this和function里的this有什么区别
- 原型链是什么，继承呢，有几种继承方式，如何实现es6的class
- eventloop是什么，浏览器的eventloop和nodejs的eventloop有什么区别，nexttick是什么
- commonjs和AMD，CMD的区别，以及跟ES MODULE的区别
- 说说require.cache
- 了解过，洋葱模型没有，它是如何实现的
- 说说nodejs中的流，也就是stream
- 你用过ts，说说你常用的ts中的数据类型
- js的数据类型，weakMap，weakSet和Map以及Set的区别是什么
- 为何0.1+0.2 不等于0.3，如何解决这个问题
- js的类型转换
- 正则表达式
- 对象循环引用会发生什么问题
- 如何捕获异步的异常，你能说出几种方案

## CSS相关

- position有哪几种属性，它们的区别是什么
- 如何实现垂直居中，移动端的呢
- margin设置百分比，是依据谁的百分比，padding呢
- 怪异盒模型和一般盒模型有什么区别
- flex：1代表什么，flex-shrink和flex-grow有什么区别
- background-size origin基准点在哪里
- 移动端1px解决方案，以及为何会产生这个问题
- 移动端高清屏图片的解决方案
- 说说GPU加速

## 跨端

- RN 实现原理
- 小程序实现原理
- webview跟h5有什么区别
- RPC 是什么
- JSBridge 原理是什么
- 网页唤起app的原理是什么

## 服务端

- oauth2了解过没有，sso呢
- JWT 如何实现的

## 网络

除了之前提到的网络问题，当然还有很多，比如

- 为何使用1x1的gif进行请求埋点
- TCP 如何进行拥塞控制

## 安全

- csrf是什么，防范措施是什么
- xss如何防范

## 浏览器相关

- 跨域是如何产生的，如何解决
- 如何检查性能瓶颈
- 打开页面白屏，如何定位问题，或者打开页面CPU100%，如何定位问题
- jsonp是什么，为何能解决跨域，使用它可能会产生什么问题
- base64会产生什么问题
- event.target和event.currTarget有什么区别

## 框架相关

- react和vue的区别
- react的调度原理
- setstate为何异步
- key的作用是什么，为何说要使用唯一key，react的diff算法是如何实现的，vue的呢
- react的事件系统是如何实现的
- react hook是如何实现的
- react的通信方式，hoc的使用场景
- 听过闭包陷阱么，为何会出现这种现象，如何避免
- vue的响应式原理
- 为何vue3.x用的是proxy而不是object.defineProperties
- vue是如何实现对数据的监听的，对数组呢
- vue中的nexttick是如何实现的
- fiber是什么，简单说说时间切片如何实现，为何vue不需要时间切片
- webpack是如何实现的，HMR是如何实现的，可以写个简单的webpack么，webpack的执行流程是怎样的
- koa源码实现，洋葱模型原理，get/post等这些方法如何set入koa里的，ctx.body为何能直接改变response的body
- 你简历上写的了解过webpack源码，到哪种程度了（实话说没写koa简单。。

## 算法相关

- js大整数加法
- 双指针
- 经典排序
- 动态规划
- 贪心算法
- 回溯法
- DFS
- BFS
- 链表操作
- 线性求值
- 预处理，前缀和

## 项目相关

- 项目中遇到的最大问题是什么，如何解决的
- nodejs作为中间层的作用是什么

## 场景题（机试）

- 如何实现直播上的弹幕组件，要求不能重叠，仿照b站上的弹幕
- 如何实现动态表单，仿照antd上的form组件
- 实现一个promise（一般不会这样问）
- 实现一个限制请求数量的方法
- 如何实现一个大文件的上传
- 实现一个eventEmitter
- 实现一个new，call，bind，apply
- 实现一个throttle，debound
- 实现promise.then，finally，all
- 实现继承，寄生组合继承，instanceof
- 实现Generator，Aynsc

---

20.11.20 更新来了

- React 生命周期，（分三个阶段进行回答，挂载阶段，更新阶段以及卸载阶段）
- Vue 生命周期 以及其父子组件的生命周期调度顺序
- 如果让你用强缓存或者协商缓存来缓存资源的话，你会如何使用
- 作用域是什么，作用域链呢？（这题我想了下，不会利用语言去表达这个东西。。）

---

目前暂时想到这些，后来有想到的会补充上去。
