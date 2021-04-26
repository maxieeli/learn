# useState()

useState使得开发者可以在函数组件里使用状态，它接受一个参数，就是当前状态的初始值，返回两个变量，第一个是状态变量，第二个是改变这个状态的函数

## 源码分析(基于react17.0.2)

首先进入到 `react/umd/react.development.js` , 找到入口

```javascript
function useState(initialState) {
    var dispatcher = resolveDispatcher();
    return dispatcher.useState(initialState);
}
```

可以得知useState函数是挂载到dispatcher对象上面的，于是由`resolveDispatcher()` 推断到 `ReactCurrentDispatcher.current`

```javascript
var ReactCurrentDispatcher = {
    current: null
};
```

之前的文章描述过，hooks的过程是调用了 `renderWithHooks()` 这一个函数。而且hooks是挂载在两个对象上的, 分别是 `HooksDispatcherOnMount` `HooksDispatcherOnUpdate`上

```javascript
// 组件第一次render
HooksDispatcherOnMountInDEV = {
  // 省略部分代码...
  useState: function (initialState) {
    return mountState(initialState);
  }
};

// 非首次渲染
HooksDispatcherOnUpdateInDEV = {
  // 省略部分代码...
  useState: function (initialState) {
    return updateState(initialState);
  }
};
```

接下来分两个分支来看两个函数

### mountState()

```javascript
// 初次render的state
function mountState(initialState) {
  // 创建新的hook对象并加到链表上
  var hook = mountWorkInProgressHook();
  if (typeof initialState === 'function') {
    initialState = initialState();
  }
  // 获取初始值并初始化hook对象
  hook.memoizedState = hook.baseState = initialState;
  // 创建更新队列，并初始化
  var queue = hook.queue = {
    pending: null, // 最后一次update的对象
    dispatch: null, // 更新函数
    lastRenderedReducer: basicStateReducer,
    lastRenderedState: initialState // 前面最后一次更新的state值，可能是函数，函数计算需要用到前一个state的值
  };
  var dispatch = queue.dispatch = dispatchAction.bind(
    null,
    currentlyRenderingFiber$1, // 绑定当前fiber和queue
    queue
  );
  return [hook.memoizedState, dispatch];
}
```

#### mountWorkInProgressHook(): 创建hook对象

```javascript
function mountWorkInProgressHook() {
  var hook = { // 创建hook对象
    memoizedState: null,
    baseState: null,
    baseQueue: null,
    queue: null,
    next: null
  };
  // 如果是组件内部的第一个hook
  if (workInProgressHook === null) {
    currentlyRenderingFiber$1.memoizedState =
    workInProgressHook =
    hook;
  } else { // 不是第一个hook对象,就直接把新创建的hook对象加到hook链表的末尾
    workInProgressHook = workInProgressHook.next = hook;
  }
  return workInProgressHook;
}
```

#### dispatchAction(): 更改状态值时调用的函数

```javascript
function dispatchAction(fiber, queue, action) {
  var eventTime = requestEventTime();
  var lane = requestUpdateLane(fiber);
  // 创建update对象，保存本次更新的相关信息及新状态
  var update = {
    lane: lane,
    action: action,
    eagerReducer: null,
    eagerState: null,
    next: null
  };
  // 将update加到queue上,更新queue的pending为当前的update，queue是环形链表
  var pending = queue.pending;
  if (pending === null) {
    update.next = update; // 环形链表
  } else {
    update.next = pending.next;
    pending.next = update;
  }
  queue.pending = update; // 更新queue的pending为当前update
}
```

### updateState()

```javascript
function updateState(initialState) {
  return updateReducer(basicStateReducer);
}

function basicStateReducer(state, action) {
  return typeof action === 'function' ? action(state) : action;
}
```

#### updateReducer()

```javascript
function updateReducer(reducer, initialArg, init) {
  var hook = updateWorkInProgressHook(); // 获取当前正在工作的hook
  var queue = hook.queue; // 更新队列
  queue.lastRenderedReducer = reducer; // 更新reducer
  var current = currentHook;
  // fiber渲染后，尚未执行完成update更新，存于currentFiber
  var baseQueue = current.baseQueue;
  // 最后一次的update对象
  var pendingQueue = queue.pending;
  if (pendingQueue !== null) {
    current.baseQueue = baseQueue = pendingQueue;
    queue.pending = null;
  }
  // 将未完成update更新插入到链表的头部
  if (baseQueue !== null) {
    var first = baseQueue.next;
    var newState = current.baseState;
    var newBaseState = null;
    var newBaseQueueFirst = null;
    var newBaseQueueLast = null;
    var update = first;
    // 调度，获取最新的state
    do {
      var updateLane = update.lane;
      if (!isSubsetOfLanes(renderLanes, updateLane)) {
        var clone = {
          lane: updateLane,
          action: update.action,
          eagerReducer: update.eagerReducer,
          eagerState: update.eagerState,
          next: null
        };
        if (newBaseQueueLast === null) {
          newBaseQueueFirst = newBaseQueueLast = clone;
          newBaseState = newState;
        } else {
          newBaseQueueLast = newBaseQueueLast.next = clone;
        }
        currentlyRenderingFiber$1.lanes = mergeLanes(currentlyRenderingFiber$1.lanes, updateLane);
        markSkippedUpdateLanes(updateLane);
      } else {
        if (newBaseQueueLast !== null) {
          var _clone = {
            lane: NoLane,
            action: update.action,
            eagerReducer: update.eagerReducer,
            eagerState: update.eagerState,
            next: null
          };
          newBaseQueueLast = newBaseQueueLast.next = _clone;
        }
        // 对应dispatchAction函数中的update.eagerReducer赋值
        if (update.eagerReducer === reducer) {
          newState = update.eagerState;
        } else { // 否则计算最新的state
          var action = update.action;
          newState = reducer(newState, action);
        }
      }
      update = update.next;
    } while (update !== null && update !== first);

    if (newBaseQueueLast === null) {
      newBaseState = newState;
    } else {
      newBaseQueueLast.next = newBaseQueueFirst;
    }
    if (!objectIs(newState, hook.memoizedState)) {
      markWorkInProgressReceivedUpdate();
    }
    hook.memoizedState = newState;
    hook.baseState = newBaseState;
    hook.baseQueue = newBaseQueueLast;
    queue.lastRenderedState = newState;
  }
  var dispatch = queue.dispatch;
  return [hook.memoizedState, dispatch];
}
```

上述方法得出总结： react通过单链表来存储hooks.在执行useState函数时，每个hook节点通过循环链表存储所有的更新操作，在update阶段会依次执行update循环链表中的所有更新操作，最终拿到最新的state并返回。