---
title: "vue 响应式原理"
subtitle: ""
date: 2020-12-05T19:07:15+08:00
lastmod: 2020-12-05T19:07:15+08:00
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

featuredImage: "logo.png"
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

最近一直忙于面试和熟悉新公司的业务，一直没更新文章。惭愧，下面来说说Vue可能会考察到的面试题，就是Vue的响应式原理。之后会继续更新React的源码解析系列。

最近面试发现很多公司问vue比较多，react源码问得比较少。而我看react比较多，现在就讲一讲vue的响应式原理吧。

vue总体来说，相对于react而言，源码还是好理解很多的。

vue的响应式主要分两大块吧。

- 依赖收集
- 派发通知

其中由这些角色所构成，

- Dep
- Watcher

## 初始化

`new Vue()`的过程中，vue先是会调用`init`的方法，初始化props,state,LifeCycle等。

- init
  - initProps
  - initState
  - initLifeCycle
  - init...

过程中到了，initState或者是initProps的时候会defineReactive，

其中有proxyGetter和proxySetter，用来触发getter和setter。

好了，直接跳到依赖收集部分吧。也就是defineReactive部分，

首先vue是通过Object.defineProperties的方式来实现双向绑定。

如代码所示，这一部分就是defineReactive主要做的事情。至于具体 get 和 set 做了什么事情，后面会解释。注意这里只是绑定，什么时候触发这个呢？接下来看看。

```js
function defineReactive$$1 (
  obj,
  key,
  val,
  customSetter,
  shallow
) {
  var dep = new Dep();

  var property = Object.getOwnPropertyDescriptor(obj, key);
  if (property && property.configurable === false) {
    return
  }

  // cater for pre-defined getter/setters
  var getter = property && property.get;
  var setter = property && property.set;
  if ((!getter || setter) && arguments.length === 2) {
    val = obj[key];
  }

  var childOb = !shallow && observe(val);
  Object.defineProperty(obj, key, {
    enumerable: true,
    configurable: true,
    get: function reactiveGetter () {
      var value = getter ? getter.call(obj) : val;
      if (Dep.target) {
        dep.depend();
        if (childOb) {
          childOb.dep.depend();
          if (Array.isArray(value)) {
            dependArray(value);
          }
        }
      }
      return value
    },
    set: function reactiveSetter (newVal) {
      var value = getter ? getter.call(obj) : val;
      /* eslint-disable no-self-compare */
      if (newVal === value || (newVal !== newVal && value !== value)) {
        return
      }
      /* eslint-enable no-self-compare */
      if (process.env.NODE_ENV !== 'production' && customSetter) {
        customSetter();
      }
      // #7981: for accessor properties without setter
      if (getter && !setter) { return }
      if (setter) {
        setter.call(obj, newVal);
      } else {
        val = newVal;
      }
      childOb = !shallow && observe(newVal);
      dep.notify();
    }
  });
}
```

### Dep.target是什么

Dep除了是个依赖收集器以外，其实它还有个静态的属性，就是target，来代表全局唯一的一个正在调度过程中的watcher。

```js
// The current target watcher being evaluated.
// This is globally unique because only one watcher
// can be evaluated at a time.
Dep.target = null;
var targetStack = [];

function pushTarget (target) {
  targetStack.push(target);
  Dep.target = target;
}

function popTarget () {
  targetStack.pop();
  Dep.target = targetStack[targetStack.length - 1];
}
```

至于什么时候会赋值target到watcher上呢，看下面watcher的get函数，它会调用一个`pushTarget`的方法，来将当前实例化的watcher被Dep.target指向。

那么它有什么作用呢？在什么会用到它呢？

来看看之前贴的一段代码，

```js
get: function reactiveGetter () {
  var value = getter ? getter.call(obj) : val;
  if (Dep.target) {
    dep.depend();
    if (childOb) {
      childOb.dep.depend();
      if (Array.isArray(value)) {
        dependArray(value);
      }
    }
  }
  return value
},
```

可以看到，它保证了watcher实例化和dep.depend的先后顺序。保证响应式的稳定性。

## 依赖收集

在讲依赖收集之前，我们先想想，什么时候实例化watcher。

### 什么时候实例化Watcher

其实是在`mountComponent`阶段，然后实例化Watcher，之后会触发Watcher内部的get函数，然后触发getter函数，其实getter就是`updateComponent`函数，这个函数会触发依赖收集。

```js
var Watcher = function Watcher (
  vm,
  expOrFn,
  cb,
  options,
  isRenderWatcher
) {
  /*省略一堆绑定*/
  this.value = this.lazy
    ? undefined
    : this.get();
};

/**
 * Evaluate the getter, and re-collect dependencies.
 */
Watcher.prototype.get = function get () {
  pushTarget(this);
  var value;
  var vm = this.vm;
  try {
    // 这里的getter是updateComponent
    value = this.getter.call(vm, vm);
  } catch (e) {
    if (this.user) {
      handleError(e, vm, ("getter for watcher \"" + (this.expression) + "\""));
    } else {
      throw e
    }
  } finally {
    // "touch" every property so they are all tracked as
    // dependencies for deep watching
    if (this.deep) {
      traverse(value);
    }
    popTarget();
    this.cleanupDeps();
  }
  return value
};
```

事实上，每次调用`render`这个函数时，会触发实例化watcher实例。什么时候会调用这个render呢？

其实就是每个vue文件里如果有一个`template`，通常会配一个render来渲染VNode。进入render函数的时候就会实例化watcher。

```js
updateComponent = function () {
    vm._update(vm._render(), hydrating);
};
new Watcher(vm, updateComponent);
```

vm是Vue实例，vm._render会生成一个VNode，_update会触发patch，在patch的过程中进行依赖收集。

往下发现有下一个VNode会继续调用mount，然后实例化watcher。

当在VNode解析的过程中，如果发现插入了一个响应式的数据，会进入到数据的getter方法里，进行依赖收集，当VNode解析完成之后，通过appendChild挂载到父节点上。

依赖收集的过程，其实就是在触发对应数据getter的时候new Dep()，后运行Dep.depend()，进行依赖收集，将watcher推入Dep的subs观察者队列里面。其实依赖收集就是这样一回事，那么推入subs列表有什么用呢？

其实是把跟对应data相关的watcher推入到subs队列，等待下次更新的时候逐一通知这些watcher进行视图的更新。

### updateComponent 的时候做了什么事情

先说结论吧，

生成VNode，并通过VNode结构对模板对象进行依赖收集

- reactiveGetter (vue.runtime.esm.js?2b0e:1027)
- proxyGetter (vue.runtime.esm.js?2b0e:4628)
- get (vue.runtime.esm.js?2b0e:2072)
- render (HelloWorld.vue?06ee:6)
- Vue._render (vue.runtime.esm.js?2b0e:3548)
- updateComponent (vue.runtime.esm.js?2b0e:4066)
- get (vue.runtime.esm.js?2b0e:4479)
- Watcher (vue.runtime.esm.js?2b0e:4468)
- mountComponent (vue.runtime.esm.js?2b0e:4073)
- Vue.$mount (vue.runtime.esm.js?2b0e:8415)
- init (vue.runtime.esm.js?2b0e:3118)
- createComponent (vue.runtime.esm.js?2b0e:5978)
- createElm (vue.runtime.esm.js?2b0e:5925)
- createChildren (vue.runtime.esm.js?2b0e:6053)
- createElm (vue.runtime.esm.js?2b0e:5954)
- patch (vue.runtime.esm.js?2b0e:6477)
- Vue._update (vue.runtime.esm.js?2b0e:3945)
- updateComponent (vue.runtime.esm.js?2b0e:4066)
- get (vue.runtime.esm.js?2b0e:4479)
- Watcher (vue.runtime.esm.js?2b0e:4468)
- mountComponent (vue.runtime.esm.js?2b0e:4073)
- Vue.$mount (vue.runtime.esm.js?2b0e:8415)
- init (vue.runtime.esm.js?2b0e:3118)
- createComponent (vue.runtime.esm.js?2b0e:5978)
- createElm (vue.runtime.esm.js?2b0e:5925)
- patch (vue.runtime.esm.js?2b0e:6516)
- Vue._update (vue.runtime.esm.js?2b0e:3945)
- updateComponent (vue.runtime.esm.js?2b0e:4066)
- get (vue.runtime.esm.js?2b0e:4479)
- Watcher (vue.runtime.esm.js?2b0e:4468)
- mountComponent (vue.runtime.esm.js?2b0e:4073)
- Vue.$mount (vue.runtime.esm.js?2b0e:8415)
- (anonymous) (main.js?56d7:6)
- ./src/main.js (app.js:1148)
- __webpack_require__ (app.js:849)
- fn (app.js:151)
- 1 (app.js:1161)
- __webpack_require__ (app.js:849)
- checkDeferredModules (app.js:46)
- (anonymous) (app.js:925)
- (anonymous) (app.js:928)

```javascript
var render = function() {
  var _vm = this
  var _h = _vm.$createElement
  var _c = _vm._self._c || _h
  return _c("div", { staticClass: "hello" }, [
    _c("h1", [_vm._v(_vm._s(_vm.msg))]),
    _vm._m(0),
    _c("h3", [_vm._v("Installed CLI Plugins")]),
    _vm._m(1),
    _c("h3", [_vm._v("Essential Links")]),
    _vm._m(2),
    _c("h3", [_vm._v("Ecosystem")]),
    _vm._m(3)
  ])
}
var staticRenderFns = [
  function() {
    var _vm = this
    var _h = _vm.$createElement
    var _c = _vm._self._c || _h
    return _c("p", [
      _vm._v(
        " For a guide and recipes on how to configure / customize this project,"
      ),
      _c("br"),
      _vm._v(" check out the "),
      _c(
        "a",
        {
          attrs: {
            href: "https://cli.vuejs.org",
            target: "_blank",
            rel: "noopener"
          }
        },
        [_vm._v("vue-cli documentation")]
      ),
      _vm._v(". ")
    ])
  },
  function() {
    var _vm = this
    var _h = _vm.$createElement
    var _c = _vm._self._c || _h
    return _c("ul", [
      _c("li", [
        _c(
          "a",
          {
            attrs: {
              href:
                "https://github.com/vuejs/vue-cli/tree/dev/packages/%40vue/cli-plugin-babel",
              target: "_blank",
              rel: "noopener"
            }
          },
          [_vm._v("babel")]
        )
      ]),
      _c("li", [
        _c(
          "a",
          {
            attrs: {
              href:
                "https://github.com/vuejs/vue-cli/tree/dev/packages/%40vue/cli-plugin-eslint",
              target: "_blank",
              rel: "noopener"
            }
          },
          [_vm._v("eslint")]
        )
      ])
    ])
  }
]
render._withStripped = true

export { render, staticRenderFns }
```

如上代码所示，一个VNode编译出来的模板是那样的。其中`_vm`相当于一个Vue实例，`_c`，`_v`分别代表什么呢？

搜索代码可以发现，其实_c就是渲染div节点，_v则是渲染文本节点。

## 派发更新

setter的时候做了什么？

- 触发dep.notify()

这个函数做了什么事情呢？

其实他是遍历subs然后调用每个watcher的update，

update做了什么？就是更新属性值。

### 视图更新之后，setter进入getter

那个xx面试官，说这个是死循环。其实源码是这样写的，

setter其实只是更新值，只是起到更新值的作用，而真正渲染其实是发生在getter。

watcher内部有个函数调用，`get(){...}` 会触发更新视图。

最终调用的函数式，`updateComponent`进行视图的更新。

## 关系图

// todo

## 一些吐槽

前段时间一个面试（深圳某云公司），面试官写错树的结构也就算了，后面问如何调试线上代码？我在想，不是应该本地搞好再发布么，模拟线上环境不行么？说了几个点，他还是坚持要调试线上代码。我也是醉了。

保持输出不易，希望大家能体谅一下笔者的苦心，包容下笔者的吐槽，因为这段时间经历多场面试后确实会感到体力不支。有点疲软。

面试官的思路让我困惑，

- vue的响应式中，getter和setter会导致死循环
- 往线上代码插入脚本进行调试
