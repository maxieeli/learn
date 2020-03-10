# React源码阅读

这是 React 源码阅读的第六篇文章，有以下几点事项需要注意以下：

+ 目前阅读的 <strong>React 版本为 16.13.0</strong>
+ 需要辅以练习代码，在结合理解进行阅读，这样对理解react源码的帮助有所提升。
+ 文中提到的源码  均以 dev 环境下的源码, 可自行 yarn 对应包查看。

以下是阅读 React 源码的 流程：

![readAll.png](https://i.loli.net/2020/03/10/DPeKbUhVsEWGYOC.png)



## 知识点

文章分为三个部分，分别讲述以下内容：

+ 类实例(class instance) 未被创建的情况
+ 类实例存在，但是current为null时，即第一次渲染时的情况
+ 当执行完上述两种情况时




在第五章中提到，workLoop() 的过程中workInProgress.tag判断类型为 `ClassComponent`，会进行相应的更新。

```
function beginWork(current, workInProgress, renderExpirationTime) {
  // ...省略部分代码
  switch (workInProgress.tag) {
	case ClassComponent: {
      var _Component2 = workInProgress.type;
      var _unresolvedProps = workInProgress.pendingProps;
      var _resolvedProps = workInProgress.elementType === _Component2 
        ? _unresolvedProps
        : resolveDefaultProps(_Component2, _unresolvedProps);
      return updateClassComponent(
        current,
        workInProgress,
        _Component2,
        _resolvedProps,
        renderExpirationTime
      );
    }
  }
}
```

本文就分析一下，`ClassComponent`是如何更新的。



## 首先先看类实例(class instance) 未被创建的情况。


### updateClassComponent()

```
function updateClassComponent(
  current,
  workInProgress,
  Component,
  nextProps,
  renderExpirationTime
) {
  var hasContext;
  if (isContextProvider(Component)) {
    hasContext = true;
    pushContextProvider(workInProgress);
  } else {
    hasContext = false;
  }
  prepareToReadContext(workInProgress, renderExpirationTime);

  // 此处的stateNode指的是ClassComponent对应的Class实例
  // FunctionComponent没有实例，所有stateNode为null
  var instance = workInProgress.stateNode;
  var shouldUpdate;
 
  // 当未创建实例时
  if (instance === null) {

    // 渲染了但是没有实例的情况，比如报错时
    if (current !== null) {
      current.alternate = null;
      workInProgress.alternate = null; 
      workInProgress.effectTag |= Placement;
    }

    // 构建 class 实例
    constructClassInstance(workInProgress, Component, nextProps);

    // 在未 render 的 class 实例上调用挂载生命周期
    mountClassInstance(
      workInProgress,
      Component,
      nextProps,
      renderExpirationTime
    );
    shouldUpdate = true;
  }
  
  // 第一次渲染
  else if (current === null) {
    // 后续讲到
  } else {
    // 后续讲到
  }
  // 后续讲到
  return nextUnitOfWork;
}
```

### constructClassInstance()

作用：构建Class Instance

```
function constructClassInstance(workInProgress, ctor, props) {
  var isLegacyContextConsumer = false;
  var unmaskedContext = emptyContextObject;
  var context = emptyContextObject;
  var contextType = ctor.contextType;
  if (typeof contextType === 'object' && contextType !== null) {
    context = readContext(contextType);
  } else {
    unmaskedContext = getUnmaskedContext(workInProgress, ctor, true);
    var contextTypes = ctor.contextTypes;
    isLegacyContextConsumer =
      contextTypes !== null &&
      contextTypes !== undefined;
    context = isLegacyContextConsumer
      ? getMaskedContext(workInProgress, unmaskedContext)
      : emptyContextObject;
  }

  // ctor即workInProgress.type, 也就是定义classComponent的类
  var instance = new ctor(props, context);

  // instance,state 即开发层面的this.state
  // 连等赋值写法
  var state = workInProgress.memoizedState = 
    instance.state !== null &&
    instance.state !== undefined ? instance.state : null;
  
  // 初始化class实例，即初始化workInProgress和instance
  adoptClassInstance(workInProgress, instance);

  if (isLegacyContextConsumer) {
    cacheContext(workInProgress, unmaskedContext, context);
  }
  return instance;
}
```

### adoptClassInstance()

作用：初始化 class 实例, 即初始化 workInProgress和instance
(1). 将classComponent初始化的时候那倒的update对象赋值给instance.updater
(2). 将新的ClassComponent实例赋值给workInProgress.stateNode
(3). 执行set方法，将workInProgress赋值给`instance._reactInternalFiber`，这样就能通过 instance 即 this 找到了 workInProgress

```
function adoptClassInstance(workInProgress, instance) {
  instance.updater = classComponentUpdater;
  workInProgress.stateNode = instance;
  set(instance, workInProgress);
  { instance._reactInternalInstance = fakeInternalInstance; }
}
```

### mountClassInstance()

作用：在未render的class实例上调用挂载生命周期

```
function mountClassInstance(
  workInProgress,
  ctor,
  newProps,
  renderExpirationTime
) {

  // 更新 props/state
  var instance = workInProgress.stateNode;
  instance.props = newProps;
  instance.state = workInProgress.memoizedState;
  instance.refs = emptyRefsObject;
  initializeUpdateQueue(workInProgress);

  var contextType = ctor.contextType;
  if (typeof contextType === 'object' && contextType !== null) {
    instance.context = readContext(contextType);
  } else {
    var unmaskedContext = getUnmaskedContext(
      workInProgress,
      ctor,
      true
    );
    instance.context = getMaskedContext(
      workInProgress,
      unmaskedContext
    );
  }

  // 执行更新 update 队列
  processUpdateQueue(
    workInProgress,
    newProps,
    instance,
    renderExpirationTime
  );

  // 因为state更新了,所以instance的state也要更新
  instance.state = workInProgress.memoizedState;


  // React16的新生命周期函数
  var getDerivedStateFromProps = ctor.getDerivedStateFromProps;
  if (typeof getDerivedStateFromProps === 'function') {
    applyDerivedStateFromProps(
      workInProgress,
      ctor,
      getDerivedStateFromProps,
      newProps
    );
    instance.state = workInProgress.memoizedState;
  }


  // 判断是否要调用 componentWillMount
  // 第一次渲染的话,是要调用componentWillMount
  if (
    typeof ctor.getDerivedStateFromProps !== 'function' &&
    typeof instance.getSnapshotBeforeUpdate !== 'function' &&
    (typeof instance.UNSAFE_componentWillMount === 'function' ||
    typeof instance.componentWillMount === 'function')
  ) {
    callComponentWillMount(workInProgress, instance); 

    // 在 componentWillMount 中是有可能执行 setState的
    // 所以 React 也要及时更新state并更新到instance上
    processUpdateQueue(
      workInProgress,
      newProps,
      instance,
      renderExpirationTime
    );
    instance.state = workInProgress.memoizedState;
  }

  // 等到真正渲染到DOM上去的时候,再去调用componentDidMount
  if (typeof instance.componentDidMount === 'function') {
    workInProgress.effectTag |= Update;
  }
}
```

mountClassInstance() 的逻辑如下：

+ 初始化props和state
+ 如果有更新队列的话,执行`processUpdateQueue()` 并更新state,从开发角度看应该是根据 `constructor`里面的内容, 将新的update推入updateQueue, 并更新一次props和state。
+ 如果开发代码中执行了`getDerivedStateFromProps()`的话，则调用对应的`applyDerivedStateFromProps`，更新state
+ 如果开发代码中有执行`componentWillMount()` 的话，则调用对应的`callComponentWillMount()`，并且如果该方法里面有调用 setState 的话,导致updateQueue里面有更新时，执行`processUpdateQueue`，更新props和state
+ 最后等到要真正渲染到DOM上去的时候，再去调用`componentDidMount()`



## 接下来看类实例(class instance)存在,但current为null时,即第一次渲染的情况。

### updateClassComponent()

```
function updateClassComponent(
  current, workInProgress,
  Component, nextProps,
  renderExpirationTime
) {
  // 此处的stateNode指的是ClassComponent对应的Class实例
  // FunctionComponent没有实例，所有stateNode为null
  var instance = workInProgress.stateNode;
  var shouldUpdate;
  
  // 第一次渲染
  } else if (current === null) {
    // 调用生命周期(componentWillMount,componentDidMount),返回 shouldUpdate
    shouldUpdate = resumeMountClassInstance(
      workInProgress,
      Component,
      nextProps,
      renderExpirationTime
    );
  } else {
    // 当已经创建实例并且不是第一次渲染
    shouldUpdate = updateClassInstance(
      current,
      workInProgress,
      Component,
      nextProps,
      renderExpirationTime
    );
  }

  var nextUnitOfWork = finishClassComponent(
    current,
    workInProgress,
    Component,
    shouldUpdate,
    hasContext,
    renderExpirationTime
  );
  return nextUnitOfWork;
}
```

### resumeMountClassInstance()

作用：复用 `ClassComponent`实例，更新props和state，调用生命周期函数`componentWillMount()`和`componentDidMount()`，最终返回`shouldUpdate`

```
function resumeMountClassInstance(
  workInProgress,
  ctor,
  newProps,
  renderExpirationTime
) {
  // 获取 ClassComponent实例
  var instance = workInProgress.stateNode;
  // 获取已有的props
  var oldProps = workInProgress.memoizedProps;
  // 初始化类实例的props
  instance.props = oldProps;
  // context相关代码，省略
  // ...

  var getDerivedStateFromProps = ctor.getDerivedStateFromProps;
  // 从开发角度上看,只要有getDerivedStateFromProps或getSnapshotBeforeUpdate 
  // 其中一个生命周期API,变量hasNewLifecycles为true
  var hasNewLifecycles =
    typeof getDerivedStateFromProps === 'function' ||
    typeof instance.getSnapshotBeforeUpdate === 'function'; 

  // 如果没有用新的生命周期的方法，则执行componentWillReceiveProps()
  // 也就是说，如果有getDerivedStateFromProps()或getSnapshotBeforeUpdate()，就不调用componentWillReceiveProps方法了
  if (
    !hasNewLifecycles &&
    (typeof instance.UNSAFE_componentWillReceiveProps === 'function' ||
      typeof instance.componentWillReceiveProps === 'function')
  ) {
    if (oldProps !== newProps || oldContext !== nextContext) {
      callComponentWillReceiveProps(
        workInProgress,
        instance,
        newProps,
        nextContext
      );
    }
  }

  // 设置 hasForceUpdate为false
  resetHasForceUpdateBeforeProcessing();
  var oldState = workInProgress.memoizedState;
  var newState = instance.state = oldState;
  processUpdateQueue(
    workInProgress,
    newProps,
    instance,
    renderExpirationTime
  );
  newState = workInProgress.memoizedState;

  // 如果新老props和state没有差别,并且没有forceUpdate的情况,那么组件就不更新
  if (
    oldProps === newProps &&
    oldState === newState &&
    !hasContextChanged() &&
    !checkHasForceUpdateAfterProcessing()
  ) {
    // 由于current为null, 即第一次渲染,需要调用componentDidMount
    if (typeof instance.componentDidMount === 'function') {
      workInProgress.effectTag |= Update;
    }
    // 就是 shouldUpdate为false
    return false;
  }

  //有调用getDerivedStateFromProps()的话，则执行对应的applyDerivedStateFromProps
  //这边能执行，说明componentWillReceiveProps()就不执行
  if (typeof getDerivedStateFromProps === 'function') {
    applyDerivedStateFromProps(
      workInProgress,
      ctor,
      getDerivedStateFromProps,
      newProps
    );
    newState = workInProgress.memoizedState;
  }

  var shouldUpdate = checkHasForceUpdateAfterProcessing() ||
    checkShouldComponentUpdate(
      workInProgress, ctor,
      oldProps, newProps,
      oldState, newState,
      nextContext
    );

  // 当有更新的时候，执行componentWillMount和componentDidMount
  if (shouldUpdate) {
    if (
      !hasNewLifecycles &&
      (typeof instance.UNSAFE_componentWillMount === 'function' ||
      typeof instance.componentWillMount === 'function')
    ) {
      startPhaseTimer(workInProgress, 'componentWillMount');
      if (typeof instance.componentWillMount === 'function') {
        instance.componentWillMount();
      }
      if (typeof instance.UNSAFE_componentWillMount === 'function') {
        instance.UNSAFE_componentWillMount();
      }
      stopPhaseTimer();
    }
    if (typeof instance.componentDidMount === 'function') {
      workInProgress.effectTag |= Update;
    }
  } else {
    if (typeof instance.componentDidMount === 'function') {
      workInProgress.effectTag |= Update;
    }
    workInProgress.memoizedProps = newProps;
    workInProgress.memoizedState = newState;
  }
  // 更新相关属性为最新的props/state,无论是否有update
  instance.props = newProps;
  instance.state = newState;
  instance.context = nextContext;
  return shouldUpdate;
}
```

该方法的流程主要如下：

+ 如果没有调用`getDerivedStateFromProps()` 或 `getSnapshotBeforeUpdate()`的话，则调用`componentWillReceiveProps()`
+ 更新updateQueue，获取newState
+ 如果新老props和state没有差别, 并且没有forceUpdate的情况,那么组件不更新,即shouldUpdate为false
+ 如果有调用`getDerviedStateFromProps()`，则执行它


### checkShouldComponentUpdate()

作用：检查是否有props/state的更新,也是判断是否需要执行`shouldComponentUpdate()`

```
function checkShouldComponentUpdate(
  workInProgress,
  ctor,
  oldProps,
  newProps,
  oldState,
  newState,
  nextContext
) {
  var instance = workInProgress.stateNode;
  // 如果有调用 shouldComponentUpdate的话，则返回执行该方法的结果
  if (typeof instance.shouldComponentUpdate === 'function') {
    startPhaseTimer(workInProgress, 'shouldComponentUpdate');
    var shouldUpdate = instance.shouldComponentUpdate(
      newProps,
      newState,
      nextContext
    );
    stopPhaseTimer();
    return shouldUpdate;
  }
  // 如果是纯组件的话,就用浅比较来比较props/state
  if (ctor.prototype && ctor.prototype.isPureReactComponent) {
    return
      !shallowEqual(oldProps, newProps) ||
      !shallowEqual(oldState, newState);
  }
  return true;
}
```

### updateClassInstance()

作用：当已经创建实例并且不是第一次渲染的话，调用更新的生命周期`componentWillUpdate/ componentDidUpdate`

```
function updateClassInstance(
  current,
  workInProgress,
  ctor,
  newProps,
  renderExpirationTime
) {
  var instance = workInProgress.stateNode;
  cloneUpdateQueue(current, workInProgress);
  var oldProps = workInProgress.memoizedProps;
  instance.props = workInProgress.type === workInProgress.elementType
    ? oldProps
    :resolveDefaultProps(workInProgress.type, oldProps);
  var oldContext = instance.context;
  var contextType = ctor.contextType;
  var nextContext = emptyContextObject;

  if (typeof contextType === 'object' && contextType !== null) {
    nextContext = readContext(contextType);
  } else {
    var nextUnmaskedContext = getUnmaskedContext(
      workInProgress,
      ctor,
      true
    );
    nextContext = getMaskedContext(workInProgress, nextUnmaskedContext);
  }

  var getDerivedStateFromProps = ctor.getDerivedStateFromProps;
  var hasNewLifecycles = 
    typeof getDerivedStateFromProps === 'function' ||
    typeof instance.getSnapshotBeforeUpdate === 'function';

  if (
    !hasNewLifecycles &&
    (typeof instance.UNSAFE_componentWillReceiveProps === 'function' ||
     typeof instance.componentWillReceiveProps === 'function')
  ) {
    if (oldProps !== newProps || oldContext !== nextContext) {
      callComponentWillReceiveProps(
        workInProgress,
        instance,
        newProps,
        nextContext
      );
    }
  }

  resetHasForceUpdateBeforeProcessing();
  var oldState = workInProgress.memoizedState;
  var newState = instance.state = oldState;
  processUpdateQueue(
    workInProgress,
    newProps,
    instance,
    renderExpirationTime
  );
  newState = workInProgress.memoizedState;

  if (
    oldProps === newProps &&
    oldState === newState &&
    !hasContextChanged() &&
    !checkHasForceUpdateAfterProcessing()
  ) {
    //这里与resumeMountClassInstance不一样
    // updateClassInstance：componentDidUpdate/getSnapshotBeforeUpdate
    // resumeMountClassInstance(): componentDidMount
    if (typeof instance.componentDidUpdate === 'function') {
      if (
        oldProps !== current.memoizedProps ||
        oldState !== current.memoizedState
      ) {
        workInProgress.effectTag |= Update;
      }
    }
    if (typeof instance.getSnapshotBeforeUpdate === 'function') {
      if (
        oldProps !== current.memoizedProps ||
        oldState !== current.memoizedState
      ) {
        workInProgress.effectTag |= Snapshot;
      }
    }
    return false;
  }
  if (typeof getDerivedStateFromProps === 'function') {
    applyDerivedStateFromProps(
      workInProgress,
      ctor,
      getDerivedStateFromProps,
      newProps
    );
    newState = workInProgress.memoizedState;
  }
  var shouldUpdate = checkHasForceUpdateAfterProcessing() ||
    checkShouldComponentUpdate(
      workInProgress, ctor,
      oldProps, newProps,
      oldState, newState,
      nextContext
    );

  if (shouldUpdate) {
    // 此处也与resumeMountClassInstance() 不同
    // updateClassInstance():componentWillUpdate/componentDidUpdate/getSnapshotBeforeUpdate
    // resumeMountClassInstance()：componentWillMount/componentDidMount
    if (
      !hasNewLifecycles &&
      (typeof instance.UNSAFE_componentWillUpdate === 'function' ||
        typeof instance.componentWillUpdate === 'function')
    ) {
      startPhaseTimer(workInProgress, 'componentWillUpdate');
      if (typeof instance.componentWillUpdate === 'function') {
        instance.componentWillUpdate(newProps, newState, nextContext);
      }
      if (typeof instance.UNSAFE_componentWillUpdate === 'function') {
        instance.UNSAFE_componentWillUpdate(newProps, newState, nextContext);
      }
      stopPhaseTimer();
    }
    if (typeof instance.componentDidUpdate === 'function') {
      workInProgress.effectTag |= Update;
    }
    if (typeof instance.getSnapshotBeforeUpdate === 'function') {
      workInProgress.effectTag |= Snapshot;
    }
  } else {
    if (typeof instance.componentDidUpdate === 'function') {
      if (
        oldProps !== current.memoizedProps ||
        oldState !== current.memoizedState
      ) {
        workInProgress.effectTag |= Update;
      }
    }
    if (typeof instance.getSnapshotBeforeUpdate === 'function') {
      if (
        oldProps !== current.memoizedProps ||
        oldState !== current.memoizedState
      ) {
        workInProgress.effectTag |= Snapshot;
      }
    }
    workInProgress.memoizedProps = newProps;
    workInProgress.memoizedState = newState;
  }
  instance.props = newProps;
  instance.state = newState;
  instance.context = nextContext;
  return shouldUpdate;
}
```

## 接下来看updateClassComponent()中最后一种情况

```
function updateClassComponent() {
  // ...
  var nextUnitOfWork = finishClassComponent(
    current, workInProgress,
    Component, shouldUpdate,
    hasContext, renderExpirationTime
  );
}
```

### finishClassComponent()

作用：判断是否执行 `render()`,并返回render下的第一个child

```
function finishClassComponent(
  current, workInProgress,
  Component, shouldUpdate,
  hasContext, renderExpirationTime
) {
  // 无论是否更新 props/state,都必须更新ref指向
  markRef(current, workInProgress);
  // 判断是否有错误捕获
  var didCaptureError =
    (workInProgress.effectTag & DidCapture) !== NoEffect;

  // 当不需要更新/更新完毕,并且没有出现error的时候
  if (!shouldUpdate && !didCaptureError) {
    if (hasContext) {
      invalidateContextProvider(workInProgress, Component, false);
    }
    // 跳过该class上的节点,以及所有子节点的更新,即跳过调用render方法
    return bailoutOnAlreadyFinishedWork(
      current,
      workInProgress,
      renderExpirationTime
    );
  }
  var instance = workInProgress.stateNode;
  ReactCurrentOwner$1.current = workInProgress;
  var nextChildren;
  if (
    didCaptureError &&
    typeof Component.getDerivedStateFromError !== 'function'
  ) {
    // 如果出现error但是没有调用getDerivedStateFromError的话就中断渲染
    nextChildren = null;
  } else {
    nextChildren = instance.render();
  }
  workInProgress.effectTag |= PerformedWork;
  if (current !== null && didCaptureError) {
    // 强制重新计算children,因为当出错时,是渲染到节点的props/state出现问题,所以不能复用，必须重新 render
    forceUnmountCurrentAndReconcile(
      current, workInProgress,
      nextChildren, renderExpirationTime
    );
  } else {
    //将ReactElement变成fiber对象并更新,生成对应DOM的实例,并挂载到真正的DOM节点上
    reconcileChildren(
      current, workInProgress,
      nextChildren, renderExpirationTime
    );
  }
  workInProgress.memoizedState = instance.state; 
  if (hasContext) {
    invalidateContextProvider(workInProgress, Component, true);
  }
  // 返回render下的第一个节点
  return workInProgress.child;
}
```

该方法的流程如下：

+ 无论是否更新 `props/state`，都必须更新ref指向。
+ 判断是否有错误捕获，赋值给 `didCaptureError`
+ 当不需要更新/更新完毕，并没有捕获到error的时候，则执行`bailoutOnAlreadyFinishedWork()`, 跳过该 ClassInstance上的节点及所有子节点的更新，即跳过调用render方法
+ 如果捕获到了error，并且没有调用`getDerivedStateFromError`的话，就中断渲染，将 `nextChildren`设置为`null`
+ 如果没有捕获到error的话，则执行`instance.render()`，重新渲染，并返回`nextChildren`
+ 渲染后，如果捕获到error，则执行`forceUnmountCurrentAndReconcile()`，强制重新计算children。否则执行`reconcileChildren()`，将 ReactElement 变成 fiber 对象并更新，生成对应DOM的实例并挂载到真正的DOM上。
+ 最后返回render下的第一个节点`workInProgress.child`
