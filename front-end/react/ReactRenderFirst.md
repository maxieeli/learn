# React源码阅读

这是 React 源码阅读的第二篇文章，有以下几点事项需要注意以下：

+ 目前阅读的 <b>React 版本为 16.13.0</b>
+ 需要辅以练习代码，在结合理解进行阅读，这样对理解react源码的帮助有所提升。
+ 文中提到的源码  均以 dev 环境下的源码, 可自行 yarn 对应包查看。

以下是阅读 React 源码的 流程：

![readAll.png](https://i.loli.net/2020/03/10/DPeKbUhVsEWGYOC.png)



## render

想必写过React项目都有写过以下类似的代码

```javascript
ReactDOM.render(<App />, document.getElementById('root'))
```

这句代码告知了React应用想在容器中渲染出一个组件，这通常也是一个React应用的入口代码，接下来就来梳理整个render的过程。

```javascript
function render(element, container, callback) {
  if (!isValidContainer(container)) {
    { throw Error( "Target container is not a DOM element." ); }
  }
  return legacyRenderSubtreeIntoContainer(
    null,
    element,
    container,
    false,
    callback
  );
}
```

在这部分代码中需要注意的一点是在调用 `legacyRenderSubtreeIntoContainer` 函数时写死了第四个参数 forceHydrate 为 false。这个参数为true时表明是服务端渲染。本文分析的是客户端渲染，所以不在此提及该内容。

接下来进入 `legacyRenderSubtreeIntoContainer` 函数中，这部分源码需要分为两部分来讲，第一部分是没有 root之前首先需要创建一个root，第二部分是有root之后的渲染流程。

```javascript
function legacyRenderSubtreeIntoContainer(
  parentComponent,
  children,
  container,
  forceHydrate,
  callback
) {
  var root = container._reactRootContainer;
  if (!root) {
    root = container._reactRootContainer 
         = legacyCreateRootFromDOMContainer(container, forceHydrate);
  }
}
```

一开始进来函数的时候肯定是没有root的，因此需要去创建一个root，可以发现这个root对象同样也被挂载在了 `container._reactRootContainer` 上。也就是DOM容器上，可以在浏览器的控制台输入查看对象。

```javascript
document.querySelector('#root')._reactRootContainer
```

得出的结果可以看到 root 是 ReactRoot 构造函数构造出来的，并且内部有一个 `_internalRoot`对象，这个对象是接下来要重点介绍的 fiber 对象。

```javascript
function legacyCreateRootFromDOMContainer(container, forceHydrate) {
  var shouldHydrate = 
    forceHydrate || shouldHydrateDueToLegacyHeuristic(container);
  if (!shouldHydrate) {
    var warned = false;
    var rootSibling;
    while(rootSibling = container.lastChild) {
      container.removeChild(rootSibling);
    }
  }
  return createLegacyRoot(container, shouldHydrate ? {
    hydrate: true
  } : undefined);
}
```

首先还是和上文提到的forceHydrate属性相关的内容，这部分在客户端渲染下，shouldHydrate肯定为false。

接下来是将容器内部的节点全部移除，一般来说都是需要写一个容器的.

```
<div id='root'></div>
```

这样的形式肯定不需要去移除子节点了，这也侧面说明了一点就是容器内部不要含有任何的子节点。一是肯定会被移除掉，二是还要进行DOM操作，会涉及重绘回流等问题。

最后返回一个 `createLegacyRoot`函数。其最终也是创建一个 ReactRoot对象。

```javascript
function createLegacyRoot(container, options) {
  return new ReactDOMBlockingRoot(container, LegacyRoot, options);
}

function ReactDOMBlockingRoot(container, tag, options) {
  this._internalRoot = createRootImpl(container, tag, options);
}

function createRootImpl(container, tag, options) {
  // 省略部分代码
  var hydrate = options != null && options.hydrate === true;
  var hydrationCallbacks = 
    options != null && options.hydrationOptions || null;
  // 这个root指的是 FiberRoot
  var root = createContainer(container, tag, hydrate);
  return root;
}

function createContainer(
  containerInfo,
  tag,
  hydrate,
  hydrationCallbacks
) {
  return createFiberRoot(containerInfo, tag, hydrate);
}
```

在 `createRootImpl`中创建了一个FiberRoot对象，返回root给`ReactDOMBlockingRoot`函数，并且挂载到了 `_internalRoot` 上，和DOM树一样， <b>fiber也会构建一个树结构(每个DOM节点一定对应一个fiber对象), FiberRoot就是整个fiber树的根节点</b>。

接下来将学习到关于fiber的内容，这里提及一点，fiber 和 Fiber是两个不同的概念，前者代表着数据结构，后者代表着新的React架构。

```javascript
function createFiberRoot(
  containerInfo,
  tag,
  hydrate,
  hydrationCallbacks
) {
  var root = new FiberRootNode(containerInfo, tag, hydrate);
  var uninitializedFiber = createHostRootFiber(tag);
  root.current = uninitializedFiber;
  uninitializedFiber.stateNode = root;
  initializeUpdateQueue(uninitializedFiber);
  return root;
}
```

在 `createFiberRoot`函数内部，分别创建了两个root，一个root叫 FiberRoot，一个叫RootFiber，并且两者还是相互引用的。

这两个对象内部拥有十几个属性，在目前只有部分属性需要了解，后续也会讲到它们的用处。

对于FiberRoot对象来说，目前需要了解到两个属性，分别是 containerInfo 和 current，前者代表着容器信息，也就是 `document.querySelector('#root')`，后者指向 RootFiber。

对于RootFiber对象来说，需要了解的属性相对多点。

```javascript
function FiberNode(tag, pendingProps, key, mode) {
  this.stateNode = null;
  this.return = null;
  this.child = null;
  this.sibling = null;
  this.effectTag = NoEffect;
  this.alternate = null;
}
```

return、child、sibling这三个属性很重要，因为它们是构成 fiber 树的主体数据结构，fiber 树其实是一个单链表结构，return及child分别对应着父子节点，并且父节点只有一个child指向它的第一个子节点，即便是父节点有好多个子节点，那么多个子节点是如何连接起来的呢？答案是 sibling。

每个子节点都有一个sibling属性指向着下一个子节点，都有一个return属性指向着父节点。

```javascript
// 用代码表示fiber树结构
const App = () => (
  <div>
    <span></span>
    <span></span>
  </div>
)
ReactDOM.render(<App />, document.querySelector('#root'))
```

假设需要渲染出以上组件，则它们对应的 fiber 树长这样：

![renderRoot.png](https://i.loli.net/2020/03/05/FahIgVExksMzfYy.png)



从图中可以看到，每个组件或者DOM节点都会对应着一个 fiber对象，也可以在浏览器控制台，查看 fiber 树的整个结构。

```
const fiber = document.querySelector('#root')._reactRootContainer._internalRoot;
```

另外还有两个属性需要提及一下。一个是effectTag，一个是alternate。

在说effectTag之前，先了解下什么是effect，简单来说就是DOM的一些操作，比如增删改，那么effectTag就是用来记录effect的，但是这个记录是通过位运算来实现的。

如果想新增一个effect的话，可以这样写 `effectTag |= Update;` 如果想删除一个 effect 的话，可以这样写 `effectTag &= ~Update`。

另外一个alternate属性，其实在一个React应用中，通常来说有两个fiber树，一个是old tree，一个是 workInProgress tree。前者对应着已经渲染好的DOM树，后者是正在执行更新中的fiber tree。还能便于中断后恢复。两棵树的节点相互引用，便于共享一些内部的属性。减少内存开销，毕竟上一篇文章说过每个组件或DOM都会对应着一个 fiber 对象，应用很大的话组成的fiber树也会很大，如果两棵树都是各自把一些相同的属性创建一遍的话，会损失不少的内存空间以及性能。

当更新结束以后，workInProgress tree 会将 old tree 替换掉。这也是性能优化里的一种做法。



## 总结

以上就是本文的所有内容了，最后用图的形式总结一下该篇的内容。

![renderFirst.png](https://i.loli.net/2020/03/05/KZdoius5r2CyJax.png)

