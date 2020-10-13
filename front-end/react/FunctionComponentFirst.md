# React源码阅读

这是 React 源码阅读的第七篇文章，有以下几点事项需要注意以下：

+ 目前阅读的 <b>React 版本为 16.13.0</b>
+ 需要辅以练习代码，在结合理解进行阅读，这样对理解react源码的帮助有所提升。
+ 文中提到的源码  均以 dev 环境下的源码, 可自行 yarn 对应包查看。

以下是阅读 React 源码的 流程：

![readAll.png](../../pic/readAll.png)



## 知识点

该内容分为两个文章解析，分别讲述以下内容：

+ 第一章，讲述渲染函数组件过程中对hooks API的使用(本文)
+ 第二章，将ReactElement变成fiber对象并更新的过程中对节点类型的实现([详情](https://github.com/owenCoderLi/learn/blob/master/front-end/react/FunctionComponentSecond.md))



在第五章中提到，workLoop() 的过程中workInProgress.tag判断类型为 `FunctionComponent`，会进行相应的更新。

```javascript
function beginWork(current, workInProgress, renderExpirationTime) {
  // ...省略部分代码
  switch (workInProgress.tag) {
    case FunctionComponent: {
      var _Component = workInProgress.type;
      var unresolvedProps = workInProgress.pendingProps;
      var resolvedProps = workInProgress.elementType === _Component
        ? unresolvedProps
        : resolveDefaultProps(_Component, unresolvedProps);

      return updateFunctionComponent(
        current,
        workInProgress,
        _Component,
        resolvedProps,
        renderExpirationTime
      );
    }
  }
}
```

本文就分析一下，`FunctionComponent`是如何更新的。

### updateFunctionComponent()

```javascript
// 执行FunctionComponent的更新
// 省略dev代码跟部分代码，后续会讲到
function updateFunctionComponent(
  current,
  workInProgress,
  Component,
  nextProps,
  renderExpirationTime
) {
  var context;
  var unmaskedContext = getUnmaskedContext(
    workInProgress,
    Component,
    true
  );
  context = getMaskedContext(workInProgress, unmaskedContext);

  var nextChildren;
  prepareToReadContext(workInProgress, renderExpirationTime);
  
  // 在渲染过程中，对立面用到的hook函数做一些操作
  nextChildren = renderWithHooks(
    current,
    workInProgress,
    Component,
    nextProps,
    context,
    renderExpirationTime
  );

  // 如果不是第一次渲染，并且没有接收到更新的话
  if (current !== null && !didReceiveUpdate) {
    // 跳过hooks函数的更新
    bailoutHooks(current, workInProgress, renderExpirationTime);

    // 上文已分析，跳过该节点以及该节点上所有子节点的更新
    return bailoutOnAlreadyFinishedWork(
      current,
      workInProgress,
      renderExpirationTime
    );
  }

  // 表明当前组件在渲染的过程中有被更新到
  workInProgress.effectTag |= PerformedWork;
  
  // 将ReactElement变成fiber对象并更新，生成对应DOM的实例，并挂载到真正DOM节点上
  reconcileChildren(
    current,
    workInProgress,
    nextChildren,
    renderExpirationTime
  );
  return workInProgress.child;
}
```

在 `updateFunctionComponent`中，主要就是执行了两个函数：

+ renderWithHooks()
+ reconcileChildren()

执行完这两个方法后返回 workProgress.child，即正在执行更新的fiber对象的第一个子节点。

### renderWithHooks()

```javascript
// 渲染的过程中，对hooks函数做一些操作

function renderWithHooks(
  current,
  workInProgress,
  Component,
  props,
  secondArg,
  nextRenderExpirationTime
) {
  renderExpirationTime = nextRenderExpirationTime;
  // 当前正要渲染的fiber对象
  currentlyRenderingFiber$1 = workInProgress;

  workInProgress.memoizedState = null;
  workInProgress.updateQueue = null;
  workInProgress.expirationTime = NoWork;
  
  // 用于存放hook函数的对象
  if (current !== null && current.memoizedState !== null) {
    ReactCurrentDispatcher.current = HooksDispatcherOnUpdateInDEV;
  } else {
    ReactCurrentDispatcher.current = HooksDispatcherOnMountInDEV;
  }
    
  var children = Component(props, secondArg);
  if (workInProgress.expirationTime === renderExpirationTime) {
    var numberOfReRenders = 0;
    do {
      workInProgress.expirationTime = NoWork;
      // 重新渲染时的fiber节点数
      numberOfReRenders += 1;
      // 释放当前state
      currentHook = null;
      workInProgressHook = null;
      workInProgress.updateQueue = null;
      ReactCurrentDispatcher.current =  HooksDispatcherOnRerenderInDEV ;
      children = Component(props, secondArg);
    } while (workInProgress.expirationTime === renderExpirationTime);
  }
  ReactCurrentDispatcher.current = ContextOnlyDispatcher;
  var didRenderTooFewHooks =
    currentHook !== null &&
    currentHook.next !== null;
  
  // 重置
  renderExpirationTime = NoWork;
  currentlyRenderingFiber$1 = null;
  currentHook = null;
  workInProgressHook = null;  
  didScheduleRenderPhaseUpdate = false;
  
  if (!!didRenderTooFewHooks) {
    { throw Error( "Rendered fewer hooks than expected. This may be caused by an accidental early return statement." ); }
  }
  return children;
}
```

在FunctionComponent中是无法使用setState的，取而代之的是 useState(), userEffect()等API
在 `renderWithHooks`中处理hooks函数：

+ 根据current用来判断是否是组件的第一次渲染
+ 无论是 `HooksDispatcherOnUpdate`还是`HooksDispatcherOnMount`, 它们都是存放hooks函数的对象

```javascript
HooksDispatcherOnMount = {
  readContext: function (context, observedBits) {
    return readContext(context, observedBits);
  },
  useCallback: function (callback, deps) {
    currentHookNameInDev = 'useCallback';
    mountHookTypesDev();
    checkDepsAreArrayDev(deps);
    return mountCallback(callback, deps);
  },
  useEffect: function (create, deps) {
    currentHookNameInDev = 'useEffect';
    mountHookTypesDev();
    checkDepsAreArrayDev(deps);
    return mountEffect(create, deps);
  }
  // ...省略部分代码
};

HooksDispatcherOnUpdateInDEV = {
  readContext: function (context, observedBits) {
    return readContext(context, observedBits);
  },
  useCallback: function (callback, deps) {
    currentHookNameInDev = 'useCallback';
    updateHookTypesDev();
    return updateCallback(callback, deps);
  },
  useEffect: function (create, deps) {
    currentHookNameInDev = 'useEffect';
    updateHookTypesDev();
    return updateEffect(create, deps);
  }
  // ...省略部分代码
};
```

### reconcileChildren()

在该函数中,将ReactElement变成fiber对象并更新, 生成对应DOM实例, 并挂载到真正的DOM节点上

```javascript
function reconcileChildren(
  current, workInProgress,
  nextChildren, renderExpirationTime
) {
  if (current === null) {
    workInProgress.child = mountChildFibers(
      workInProgress, null,
      nextChildren, renderExpirationTime
    );
  } else {    
    workInProgress.child = reconcileChildFibers(
      workInProgress, current.child,
      nextChildren, renderExpirationTime
    );
  }
}
```

因为 `mountChildFibers`和`reconcileChildFibers`调用得是同一个函数

```javascript
// false表示是第一次渲染，true反之
var reconcileChildFibers = ChildReconciler(true);
var mountChildFibers = ChildReconciler(false);
```

### ChildReconciler(), reconcileChildFibers()

该方法前面全是 function的定义，最后返回 `reconcileChildFibers`

```javascript
function ChildReconciler(shouldTrackSideEffects) {
  // 省略部分代码
  function reconcileChildFibers(
    returnFiber,
    currentFirstChild,
    newChild,
    expirationTime
  );
  return reconcileChildFiber;
}
```

因为第一次渲染时无副作用，所以 `shouldTrackSideEffects`为false，多次渲染是有副作用的，所以 `shouldTrackSideEffects`为true。

```javascript
function reconcileChildFibers(
  returnFiber,
  currentFirstChild,
  newChild,
  expirationTime
) {
  var isUnkeyedTopLevelFragment = 
    typeof newChild === 'object' &&
    newChild !== null &&
    // 在开发中写<div>{ arr.map((a,b)=>xxx) }</div>，这种节点称为 REACT_FRAGMENT_TYPE
    newChild.type === REACT_FRAGMENT_TYPE &&
    newChild.key === null;
    
  // type 为REACT_FRAGMENT_TYPE是不需要任何更新的，直接渲染子节点即可
  if (isUnkeyedTopLevelFragment) {
    newChild = newChild.props.children;
  }

  var isObject = typeof newChild === 'object' && newChild !== null;
  // element节点
  if (isObject) {
    switch (newChild.$$typeof) {
      // ReactElement节点
      case REACT_ELEMENT_TYPE:
        return placeSingleChild(
          reconcileSingleElement(
            returnFiber,
            currentFirstChild,
            newChild,
            expirationTime
          )
        );
      case REACT_PORTAL_TYPE:
        return placeSingleChild(
          reconcileSinglePortal(
            returnFiber,
            currentFirstChild,
            newChild,
            expirationTime
          )
        );
    }
  }

  // 文本节点
  if (typeof newChild === 'string' || typeof newChild === 'number') {
    return placeSingleChild(
      reconcileSingleTextNode(
        returnFiber,
        currentFirstChild,
        '' + newChild,
        expirationTime
      )
    );
  }

  // 数组节点
  if (isArray$1(newChild)) {
    return reconcileChildrenArray(
      returnFiber,
      currentFirstChild,
      newChild,
      expirationTime
    );
  }

  if (getIteratorFn(newChild)) {
    return reconcileChildrenIterator(
      returnFiber,
      currentFirstChild,
      newChild,
      expirationTime
    );
  }

  // 如果未符合上述element节点的要求，则报错
  if (isObject) {
    throwOnInvalidObjectType(returnFiber, newChild);
  }

  if (typeof newChild === 'undefined' && !isUnkeyedTopLevelFragment) {
    switch (returnFiber.tag) {
      case ClassComponent: {
      }
      case FunctionComponent: {
        var Component = returnFiber.type;
        // 报错
      }
    }
  }
  // 如果旧节点还存在，但是更新的节点是null的话，需要删除旧节点的内容
  return deleteRemainingChildren(returnFiber, currentFirstChild);
}
```

(1). `isUnkeyedTopLevelFragment`

当在开发中写了如: `<div>{ arr.map(a, b) => xxx) }</div>`  这样的代码时，这种节点类型会被判定为 REACT_FRAGMENT_TYPE，React会直接渲染它的子节点：`newChild = newChild.props.children`。

(2) 如果element type是object的话，也就是ClassComponent 或 FunctionComponent 会有两种情况

+ 一个是 REACT_ELEMENT_TYPE： 常见的ReactElement节点。 执行`reconcileSingleElement`
+ 一个是REACT_PORTAL_TYPE： portal节点(应用于对话框，提示框)

(3) 如果是文本节点，会执行 `reconcileSingleTextNode`方法

(4) 如果执行到最后的`deleteRemainingChildren`，说明待更新的节点是null，需要删除原有旧节点的内容

可以看到 ChildReconciler中的`reconcileChildFibers`方法的作用就是根据新节点 newChild 的节点类型，来执行不同的操作节点函数。

在下一篇中，会解析 `reconcileSingleElement`, `reconcileSingleTextNode`, `reconcileChildrenArray`方法。
