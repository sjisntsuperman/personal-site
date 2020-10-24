---
title: "聊聊 module"
subtitle: ""
date: 2020-10-21T00:13:51+08:00
lastmod: 2020-10-21T00:13:51+08:00
author: ""
authorLink: ""
description: ""

tags: [
  ”web
]
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

将一个复杂的程序依据一定的规则(规范)封装成几个块(文件), 并进行组合在一起，块的内部数据与实现是私有的, 只是向外部暴露一些接口(方法)与外部其它模块通信。

## 进化史

1. 全局function
2. namespace
3. IIFE
4. 模块化

## 模块化和组件化带来的好处

1. 避免命名冲突(减少命名空间污染)
2. 更好的分离, 按需加载
3. 更高复用性
4. 高可维护性
5. 避免script过多而导致的请求数过多，从而提高性能
6. 难以维护

## 模块化规范（CommonJs/AMD/CMD/ES MODULE）

### CommonJs

每一个文件有自己的作用域，一个文件就是一个模块，一个作用域。在服务端是同步加载的，在浏览器端是要提前编译，之后加载的。

#### 特点

1. 不会引起全局污染
2. 第一次加载，取缓存结果，如果结果变更，则重新缓存
3. 同步

#### 范式

```js
const Koa = require('koa');
const app = new Koa();

module.exports=app;
```

#### 加载机制

**输入的是被输出的值的拷贝**。

### AMD

以requirejs为代表。异步加载。

范式：

```js
// 暴露
define(function(){})
// 引入
require(['./a.js','./b.js'],function(){})
```

### CMD

以seajs为代表，define提前，异步加载。

范式：

```js
// 暴露
define(function(require, exports, module){
  exports.xxx = value
  module.exports = value
})
// 引入
define(function (require) {
  var m1 = require('./module1')
  var m4 = require('./module4')
  m1.show()
  m4.show()
})
```

### ES MODULE

ES6 模块的设计思想是尽量的静态化，使得编译时就能确定模块的依赖关系，以及输入和输出的变量。CommonJS 和 AMD 模块，都只能在运行时确定这些东西。比如，CommonJS 模块就是对象，输入时必须查找对象属性。

范式：

```js
// 引入
import a from 'b';
// 暴露
export default a;
```

## es6 module 和 commonjs  的区别

1. commonjs输出的是值的拷贝，es6 module输出的是值的引用
2. commonjs是运行时加载，es6 module是编译时输出接口

## commonjs

### require, module.exports

  实则, require建造了一个闭包, 变量内部使用;

### require 创建了Module的函数供开发者使用

  ```js
  Module {
    id: '.', // 如果是 mainModule id 固定为 '.'，如果不是则为模块绝对路径
    exports: {}, // 模块最终 exports
    filename: '/absolute/path/to/entry.js', // 当前模块的绝对路径
    loaded: false, // 模块是否已加载完毕
    children: [], // 被该模块引用的模块
    parent: '', // 第一个引用该模块的模块
    paths: [ // 模块的搜索路径
      '/absolute/path/to/node_modules',
      '/absolute/path/node_modules',
      '/absolute/node_modules',
      '/node_modules'
    ]
  }
  ```
  
### require 和 Module.require

Module.require 就是 require的原生引用。 它们的效果是一样的。

### require.main

node 通过判断 require.main.filename == 当前运行的filename 来决定是否同步运行。

例如 require('./test') 不是同步。而node filename 是。

### require.cache

require.cache 会把值的引用缓存起来。即使循环引用，也会优先访问这个cache里面的值，以至于不会导致循环引用的崩溃情况。

不过，也会导致线上环境不能获取最新的文件。

## es6 module和commonjs 共存

实则, es6 module转化为commonjs, 成为require中的 __esModule 来存储。

require 引用 es6 中 export default 的模块的话， require(xx).default 即可。

## module.exports和exports

## 参考

1. <https://juejin.im/post/5c17ad756fb9a049ff4e0a62>
