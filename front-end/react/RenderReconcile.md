# React源码阅读

这是 React 源码阅读的第五篇文章，有以下几点事项需要注意以下：

+ 目前阅读的 <strong>React 版本为 16.13.0</strong>
+ 需要辅以练习代码，在结合理解进行阅读，这样对理解react源码的帮助有所提升。
+ 文中提到的源码  均以 dev 环境下的源码, 可自行 yarn 对应包查看。

以下是阅读 React 源码的 流程：

![readAll.png](https://i.loli.net/2020/03/10/DPeKbUhVsEWGYOC.png)



## 知识点

该篇主要分为两部分，讲述以下内容：

+ 调和概述以及过程
+ workLoop()

## 调和的过程

组件更新归结到底还是DOM的更新，对于React来说，这部分的内容分为两个阶段：

+ 调和阶段，基本上也就是虚拟DOM的diff算法
+ 提交阶段，也就是将上一个阶段中diff出来的内容体现到DOM上

对于整个更新的过程，其实是在反复寻找工作单元并运行它们，接下来解析源码

在 `ensureRootIsScheduled`中，`scheduleSyncCallback`和 `scheduleCallback`分别调用了 `performSyncWorkOnRoot` 和 `performConcurrentWorkOnRoot`,而在这两个函数中的 `do` 执行中指向了 `performUnitOfWork`函数.


### performUnitOfWork()

```
// 从上到下遍历操作节点,至底层后,再从下至上根据effectTag操作节点
function performUnitOfWork(unitOfWork) {
  // 获取当前节点
  var current = unitOfWork.alternate;

  var next;
  if ( (unitOfWork.mode & ProfileMode) !== NoMode) {
    startProfilerTimer(unitOfWork);

    // 进行节点操作，并创建子节点
    // 判断fiber有无更新，有更新则进行相应的组件更新,无更新则复制节点
    next = beginWork$1(current, unitOfWork, renderExpirationTime$1);
    stopProfilerTimerIfRunningAndRecordDelta(unitOfWork, true);
  } else {
    next = beginWork$1(current, unitOfWork, renderExpirationTime$1);
  }

  // 将待更新的props替换成正在用的props
  unitOfWork.memoizedProps = unitOfWork.pendingProps;
  
  // 说明已经更新到了最底层叶子节点,并且叶子节点的兄弟节点已经遍历完
  if (next === null) {
    // 从上到下遍历完后，completeUnitOfWork会从下到上根据effectTag进行一些处理
    next = completeUnitOfWork(unitOfWork);
  }
  ReactCurrentOwner$2.current = null;
  return next;
}
```

### beginWork()

```
// 判断fiber有无更新，有更新则进行相应的组件更新，无更新则复制节点
function beginWork(current, workInProgress, renderExpirationTime) {

  // 只有当调用了react.domRender时,rootFiber的expirationTime才有值，rootFiber才会更新
  // 获取fiber对象上更新的过期时间
  var updateExpirationTime = workInProgress.expirationTime;

  // 如果不是第一次渲染
  if (current !== null) {
    
    // 上次渲染完成后的props
    var oldProps = current.memoizedProps;
    
    // 新变更带来的props
    var newProps = workInProgress.pendingProps;

    // 前后props是否不相等
    // 是否有老版本的context使用，并发生了变化
    // 开发环境永为false
    if (
      oldProps !== newProps ||
      hasContextChanged() || 
      (workInProgress.type !== current.type )
    ) {
      // 判断接收到了更新 update
      didReceiveUpdate = true;
    }
    // 有更新,但是优先级不高,在本次渲染过程中不需要执行,设为false
    else if (updateExpirationTime < renderExpirationTime) {
      didReceiveUpdate = false; 
      
      // 根据workInProgress的tag，进行相应组件的更新
      switch (workInProgress.tag) {
        // 省略代码
        case HostRoot:
          pushHostRootContext(workInProgress);
          resetHydrationState();
          break;
        case ClassComponent: {
          var Component = workInProgress.type;
          if (isContextProvider(Component)) {
            pushContextProvider(workInProgress);
          }
          break;
        }
      }
      // 跳过该节点及所有子节点的更新
      return bailoutOnAlreadyFinishedWork(
        current,
        workInProgress,
        renderExpirationTime
      );
    } else {
      didReceiveUpdate = false;
    }
  } else {
    didReceiveUpdate = false;
  }
  workInProgress.expirationTime = NoWork;

  // 如果节点是有更新的
  // 根据节点类型进行组件的更新
  switch (workInProgress.tag) {
    // 省略部分代码
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

### completeUnitOfWork()

```
// 完成当前节点的work, 然后移动到兄弟节点，重复该操作, 当没有更多兄弟节点时, 返回至父节点
function completeUnitOfWork(unitOfWork) {

  // 从下到上，移动到该节点的兄弟节点，如果一直往上没有兄弟节点，就返回父节点
  workInProgress = unitOfWork;
  do {
    // 获取当前节点
    var current = workInProgress.alternate;
    // 获取父节点
    var returnFiber = workInProgress.return;

    // 如果该节点没有异常抛出，即可正常执行
    if ((workInProgress.effectTag & Incomplete) === NoEffect) {
      setCurrentFiber(workInProgress);
      var next = void 0;

      // 如果不能使用分析器的timer, 直接执行completeWork
      // 否则执行分析器timer, 并执行completeWork
      if ( (workInProgress.mode & ProfileMode) === NoMode) {
        // 完成该节点的更新
        next = completeWork(
          current,
          workInProgress,
          renderExpirationTime$1
        );
      } else {
        // 启动分析器的定时器，并赋值当前时间
        startProfilerTimer(workInProgress);
        // 完成该节点的更新
        next = completeWork(
          current,
          workInProgress,
          renderExpirationTime$1
        );
        
        // 在没报错的前提下，更新渲染持续时间
        // 记录分析器的timer的运行时间间隔,并停止timer
        stopProfilerTimerIfRunningAndRecordDelta(workInProgress, false);
      }
      stopWorkTimer(workInProgress);
      resetCurrentFiber();
      // 更新该节点的work时长和子节点的expirationTime
      resetChildExpirationTime(workInProgress);

      // 如果next存在，则表示产生了新work
      if (next !== null) {
        return next;
      }

      // 如果父节点存在, 并且其 Effect 链没有被赋值的话
      if (
        returnFiber !== null && 
        (returnFiber.effectTag & Incomplete) === NoEffect
      ) {
        // 子节点的完成顺序会影响副作用的顺序
        // 如果父节点没有挂载 firstEffect的话,将当前节点的firstEffect赋值给父节点的firstEffect
        if (returnFiber.firstEffect === null) {
          returnFiber.firstEffect = workInProgress.firstEffect;
        }

        // 根据当前节点的lastEffect,初始化父节点的lastEffect
        if (workInProgress.lastEffect !== null) {
        
          // 如果父节点的lastEffect有值的话,将nextEffect赋值
          // 目的是串联Effect链
          if (returnFiber.lastEffect !== null) {
            returnFiber.lastEffect.nextEffect =
              workInProgress.firstEffect;
          }
          returnFiber.lastEffect = workInProgress.lastEffect;
        }

        // 获取副作用标记
        var effectTag = workInProgress.effectTag;

        // 如果该副作用标记大于PerformedWork
        if (effectTag > PerformedWork) {
          //当父节点的lastEffect不为空的时候,将当前节点挂载到父节点的副作用链的最后
          if (returnFiber.lastEffect !== null) {
            returnFiber.lastEffect.nextEffect = workInProgress;
          } 
          // 否则，将当前节点挂载在父节点的副作用链的头-firstEffect上
          else {
            returnFiber.firstEffect = workInProgress;
          }

          //无论父节点的lastEffect是否为空，都将当前节点挂载在父节点的副作用链的lastEffect上
          returnFiber.lastEffect = workInProgress;
        }
      }
    }
    // 如果该fiber节点未能完成work的话
    else {
      // 节点未能完成更新，捕获其中的错误
      var _next = unwindWork(workInProgress);

      // 由于该 fiber 未能完成，所以不必重置它的 expirationTime
      if ((workInProgress.mode & ProfileMode) !== NoMode) {
      
        // 记录分析器的timer的运行时间间隔，并停止timer
        stopProfilerTimerIfRunningAndRecordDelta(workInProgress, false);
        
        // 虽然报错了，但仍然会累计 work 时长
        var actualDuration = workInProgress.actualDuration;
        var child = workInProgress.child;
        while (child !== null) {
          actualDuration += child.actualDuration;
          child = child.sibling;
        }
        workInProgress.actualDuration = actualDuration;
      }

      // 如果next存在，则表示产生了新 work
      if (_next !== null) {
        // 停止失败的work计时
        stopFailedWorkTimer(workInProgress);
        
        // 更新effectTag,标记是restart的
        _next.effectTag &= HostEffectMask;
        
        // 返回next,以便执行新work
        return _next;
      }
      stopWorkTimer(workInProgress);
      
      // 如果父节点存在的话，重置它的effect链,标记为未完成
      if (returnFiber !== null) {
        returnFiber.firstEffect = returnFiber.lastEffect = null;
        returnFiber.effectTag |= Incomplete;
      }
    }

    // 获取兄弟节点
    var siblingFiber = workInProgress.sibling;
    if (siblingFiber !== null) {
      return siblingFiber;
    }

    // 如果能执行到这一步的话，说明 siblingFiber 为 null，
    // 那么就返回至父节点
    workInProgress = returnFiber;
  } while (workInProgress !== null);

  //当执行到这里的时候，说明遍历到了 root 节点，已完成遍历
  //更新workInProgressRootExitStatus的状态为「已完成」
  if (workInProgressRootExitStatus === RootIncomplete) {
    workInProgressRootExitStatus = RootCompleted;
  }
  return null;
}
```

### 小结

以上就是组件在调和过程中的源码解析，用一个流程图来表示workLoop的运行机制

![workLoop.png](https://i.loli.net/2020/03/09/3Ar5KkfMtQsZY7C.png)


