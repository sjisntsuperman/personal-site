---
title: "React 源码解读系列一 -- ReactDom.render做了些什么"
subtitle: ""
date: 2020-11-12T15:59:50+08:00
lastmod: 2020-11-12T15:59:50+08:00
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

featuredImage: "react.jpg"
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

用React挺久了。。废话不多说，直接开搞吧。

### 环境

- "react": "^17.0.1"
- "react-dom": "^17.0.1"

### 带着问题看源码

这个系列主要是为了解决面试中常见的问题，列举一下，

- 初次渲染流程
- key的作用是什么
- setState的流程是什么
- Fiber架构是什么
- hook的原理是什么

在这篇文章中，我们主要讲了初次渲染流程，尝试带着下面几个问题看源码，

- 何时挂载DOM节点
- 异步渲染模式
- 每个阶段做了什么事情

## 正文

这里我们直接用`create-react-app`来生成代码，然后进行调试。

```jsx
/* App.js */
import React from 'react';
import ReactDOM from 'react-dom';
import './index.css';
import App from './App';
// import reportWebVitals from './reportWebVitals';

ReactDOM.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>,
  document.getElementById('root')
);
```

App.js里的代码是这样，

```jsx
import logo from './logo.svg';
import './App.css';

function App() {
  return (
    <div className="App">
      <header className="App-header">
        <img src={logo} className="App-logo" alt="logo" />
        <p>
          Edit <code>src/App.js</code> and save to reload.
        </p>
        <a
          className="App-link"
          href="https://reactjs.org"
          target="_blank"
          rel="noopener noreferrer"
        >
          Learn React
        </a>
      </header>
    </div>
  );
}

export default App;
```

## 调用栈

这是从webpack解析到初次渲染的调用栈。

- appendChildToContainer (react-dom.development.js:10266)
- insertOrAppendPlacementNodeIntoContainer (react-dom.development.js:21197)
- insertOrAppendPlacementNodeIntoContainer (react-dom.development.js:21203)
- insertOrAppendPlacementNodeIntoContainer (react-dom.development.js:21203)
- commitPlacement (react-dom.development.js:21181)
- commitMutationEffects (react-dom.development.js:23361)
- callCallback (react-dom.development.js:3945)
- invokeGuardedCallbackDev (react-dom.development.js:3994)
- invokeGuardedCallback (react-dom.development.js:4056)
- commitRootImpl (react-dom.development.js:23121)
- unstable_runWithPriority (scheduler.development.js:646)
- runWithPriority$1 (react-dom.development.js:11276)
- commitRoot (react-dom.development.js:22990)
- performSyncWorkOnRoot (react-dom.development.js:22329)
- scheduleUpdateOnFiber (react-dom.development.js:21881)
- updateContainer (react-dom.development.js:25482)
- (anonymous) (react-dom.development.js:26021)
- unbatchedUpdates (react-dom.development.js:22431)
- legacyRenderSubtreeIntoContainer (react-dom.development.js:26020)
- render (react-dom.development.js:26103)
- (anonymous) (index.js:7)
- ./src/index.js (index.js:18)
- __webpack_require__ (bootstrap:856)
- fn (bootstrap:150)
- 1 (reportWebVitals.js:14)
- __webpack_require__ (bootstrap:856)
- checkDeferredModules (bootstrap:45)
- webpackJsonpCallback (bootstrap:32)
- (anonymous) (main.chunk.js:1)

从调用栈明显能看到调度的流程。下面我们从每个调用栈中看看初次渲染做了什么事情。

## 解析

### webpack 将 文件转译成 React.Element

### legacyRenderSubtreeIntoContainer

因为是初次渲染，进入unbatchedUpdates，尽快渲染。

### updateContainer

这一阶段会requestUpdate，请求更新，

之后createUpdate，然后进入更新，主要是更新节点的状态。也就是setState的批量更新会在这里进行。

之后会单独做个文章分析，其实就是创建更新后，利用优先队列进行更新节点的属性。

### scheduleUpdateOnFiber

进行更新调度。这个阶段，包含下面几个调度分先后顺序（这个不是调用栈）进行

- renderRootSync
- workLoopSync
- performUnitOfWork
- beginWork$1
- beginWork
- updateHostRoot
- reconcileChildren
- reconcileChildFibers
- completeUnitOfWork
- completeWork
- updateHostContainer
- appendAllChildren

其中，beginWork会根据组件类型来分别处理不同组件。主要有这三种，

- HostComponent
- ClassComponent
- FunctionComponenet

workLoopSync 大循环结束后进入`completeUnitOfWork`，之后到`completeWork`。

每个 workLoop 会循环生成 nextUnitOfWork，然后后序遍历树的时候会进入`appendAllChildren`的阶段。

最后到`appendAllChildren`阶段，会循环判断是否为叶子节点，不是的话放入Fiber树的Child字段中，是的话会通过`parentInstance.appendChild(child);`的方法挂载到Fiber树中的DOM结构中。

最终会以Fiber树的形式存入 workInProgress 树中的 stateNode字段中，在commit阶段渲染到页面上。

#### stateNode

这里可以理解成为 用于渲染的DOM对象。下方有它的结构，可以看看。

### commitRoot

提交更新阶段。这一阶段会根据计算出来的优先级进行渲染，因为是初次渲染，所以优先级为99，即最高。

下面跳过几个函数，到`commitRootImpl`

### commitRootImpl

这一阶段处理一些副作用，主要通过`firstEffect`和`nextEffect`来控制当前渲染的Fiber树。这一阶段里，会存在着两颗Fiber树。其中`nextEffect`是全局变量。

这个阶段主要有以下几个问题，

- nextEffect是什么
- firstEffect是什么

这个阶段主要触发了下面几个处理effects，

- commitBeforeMutationEffects
- commitMutationEffects（这个阶段进行挂载节点）
- commitLayoutEffects

其中在`commitBeforeMutationEffects`之前会通过`nextEffect = firstEffect` 来赋值，然后在`commitLayoutEffects`中使用后，将`nextEffect = null`。

具体在下面这两个方法中会用到，可以看出主要是来处理副作用的，

- commitLifeCycles
- commitAttachRef

初次渲染完成，回顾下几个问题，

- current树 和 workInProgress树
- React的异步渲染机制

### React的异步渲染机制

其实React是同步渲染机制，即使官方说Fiber出来后要搞个异步渲染，可是通过代码发现至今没有完成。

### current和 workInProgress

current树很好理解，就是当前调度过程中进行处理的Fiber树，然后通过处理转换成需要渲染在页面上的workInProgress树。

### Fiber数据结构

```js
function FiberNode(tag, pendingProps, key, mode) {
  // Instance
  this.tag = tag;
  this.key = key;
  this.elementType = null;
  this.type = null;
  this.stateNode = null; // Fiber

  this.return = null;
  this.child = null;
  this.sibling = null;
  this.index = 0;
  this.ref = null;
  this.pendingProps = pendingProps;
  this.memoizedProps = null;
  this.updateQueue = null;
  this.memoizedState = null;
  this.dependencies = null;
  this.mode = mode; // Effects

  this.flags = NoFlags;
  this.nextEffect = null;
  this.firstEffect = null;
  this.lastEffect = null;
  this.lanes = NoLanes;
  this.childLanes = NoLanes;
  this.alternate = null;

  {
    // Note: The following is done to avoid a v8 performance cliff.
    //
    // Initializing the fields below to smis and later updating them with
    // double values will cause Fibers to end up having separate shapes.
    // This behavior/bug has something to do with Object.preventExtension().
    // Fortunately this only impacts DEV builds.
    // Unfortunately it makes React unusably slow for some applications.
    // To work around this, initialize the fields below with doubles.
    //
    // Learn more about this here:
    // https://github.com/facebook/react/issues/14365
    // https://bugs.chromium.org/p/v8/issues/detail?id=8538
    this.actualDuration = Number.NaN;
    this.actualStartTime = Number.NaN;
    this.selfBaseDuration = Number.NaN;
    this.treeBaseDuration = Number.NaN; // It's okay to replace the initial doubles with smis after initialization.
    // This won't trigger the performance cliff mentioned above,
    // and it simplifies other profiler code (including DevTools).

    this.actualDuration = 0;
    this.actualStartTime = -1;
    this.selfBaseDuration = 0;
    this.treeBaseDuration = 0;
  }
```

其中stateNode字段，是存储子节点的Fiber树。在面试过程中，有被问到过Fiber树和虚拟DOM树的区别。下面来看一下虚拟DOM树。

```js
function FiberRootNode(containerInfo, tag, hydrate) {
  this.tag = tag;
  this.containerInfo = containerInfo;
  this.pendingChildren = null;
  this.current = null;
  this.pingCache = null;
  this.finishedWork = null;
  this.timeoutHandle = noTimeout;
  this.context = null;
  this.pendingContext = null;
  this.hydrate = hydrate;
  this.callbackNode = null;
  this.callbackPriority = NoLanePriority;
  this.eventTimes = createLaneMap(NoLanes);
  this.expirationTimes = createLaneMap(NoTimestamp);
  this.pendingLanes = NoLanes;
  this.suspendedLanes = NoLanes;
  this.pingedLanes = NoLanes;
  this.expiredLanes = NoLanes;
  this.mutableReadLanes = NoLanes;
  this.finishedLanes = NoLanes;
  this.entangledLanes = NoLanes;
  this.entanglements = createLaneMap(NoLanes);

  {
    this.mutableSourceEagerHydrationData = null;
  }

  {
    this.interactionThreadID = tracing.unstable_getThreadID();
    this.memoizedInteractions = new Set();
    this.pendingInteractionMap = new Map();
  }

  {
    switch (tag) {
      case BlockingRoot:
        this._debugRootType = 'createBlockingRoot()';
        break;

      case ConcurrentRoot:
        this._debugRootType = 'createRoot()';
        break;

      case LegacyRoot:
        this._debugRootType = 'createLegacyRoot()';
        break;
    }
  }
}
```

可以看到，虚拟DOM树相比较调度过程中的Fiber树，多了一些tag，hydrate等字段，来模拟DOM树的结构。

---

普通函数组件的初次渲染就到这里了，之后会继续写关于其他问题的文章。

推荐阅读：

1. <https://blog.logrocket.com/deep-dive-into-react-fiber-internals/>
