# Redux 及 中间件 原理解析

相对React来说，Hooks的Api相当简洁，代码也相对容易理解，Redux则不同，大量的样板代码，以及各种纯函数的限制，总感觉有点不适应，React的开发，很大一部分在于Redux，而到了2020年，大多数都会使用Hooks，Redux管理数据相对 "老了"，但是，对于Redux来说，出色的调试机制和完整的模块管理功能，还是一个短时间内不可被替代的状态管理方案。

因此在熟练使用 Redux 的同时，也是有必要去研究它内部的原理，体会它的设计思想，这样不仅能加深对 Redux 本身的理解， 也能巩固原生JS的功底，锤炼编程思想。

想必使用过 Redux 会对以下代码有所印象：

```javascript
import {createStore, compose, applyMiddleware} from 'redux';

const composeEnhancers = window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__ ||
  compose;

const store = createStore(
  reducer,
  composeEnhancers(applyMiddleware(thunk))
)
```

```javascript
import {combineReducers} from 'redux-immutable';
import {reducer as recommendReducer} from './xx/store/index';

export default combineReducer({
  recommend: recommendReducer
})
```

在这里提出几个疑问，createStore 发生了什么 ？ dispatch 执行后在内部是怎样运作的？compose 函数做了什么事情？combineReducer是如何合并不同的reducer的？applyMiddleware是如何组织中间件的？

接下来就一一拆解 Redux 在背后做的这些事情。



## createStore 揭秘

createStore,顾名思义，是要创建一个仓库，是redux的核心所在，它最后要返回四个非常重要的属性，分别是 <b>getState、subscribe、dispatch、replaceReducer</b>。

```javascript
export default function createStore(reducer, preloadedState, enhancer) {
  // ...
  return {
    getState, // 获取到state
    subscribe, // 采用发布订阅形式，这个方法进行观察者的订阅
    dispatch, // 派发 action
    replaceReducer // 用新的 reducer替换现在的 reducer
  }
}
```

进入createStore，第一步是检查参数，一共可以接收三个参数，reducer表示改变store数据的纯函数，preloadedState表示初始状态，第三个参数暂且不管，后面会提及到

```javascript
export default function createStore(reducer, preloadedState, enhancer) {
  // reducer必须是函数
  // 当前 reducer
  let currentReducer = reducer
  // state数据，redux的根本
  let currentState = preloadedState
  // 订阅者集合
  let currentListeners = []
  let nextListeners = currentListeners
  // 是否 有 dispatch 在执行
  let isDispatching = false
  // ...
}
```

首先看看它的 getState 方法：

```javascript
function getState() {
  // 如果有dispatch正在执行则报错
  if(isDispatching) throw new Error('xxxx');
  return currentState
}
```

它的subscribe方法其实是基于发布订阅模式的，想一想只有一个数组来存放订阅者的时候可能会出现什么问题。

假设有10个订阅者订阅了store，然后一旦条件触发store会依次执行所有的订阅者。

这时第一个方法中，它把其他九个全部退订了，这个时候数组里只剩下一个订阅者，但是循环还在继续，从数组后面的索引拿订阅者来执行，会报错，因为不存在了。

当然还有更加复杂的情况，这些情况本质上是订阅者拥有订阅和退订的权利，也就是说，它可以改变订阅者数组，但是遍历订阅者的时候是基于最初的订阅者数组。

因此需要缓存最初的数组，在调用订阅者的时候，一切关于currentListener的改变都不允许，但是可以拷贝一份同样的数组，让它来承担订阅者对数组的改变，那个数组就是nextListeners

subscribe方法如下定义：

```javascript
function ensureCanMutateNextListeners() {
  // 如果 next 和 current 数组是一个引用，那这种情况是危险的
  // 原因上面已经谈到，我们需要 next 和 current 保持各自独立
  if (nextListeners === currentListeners) {
    nextListeners = currentListeners.slice ()
  }
}

function subscribe (listener) {
  if (typeof listener !== 'function') {
    throw new Error ('Expected the listener to be a function.')
  }
  // 如果正在有 dispatch 执行则报错
  if (isDispatching) {
    throw new Error ("xxx")
  }
  let isSubscribed = true
  ensureCanMutateNextListeners ()
  nextListeners.push (listener)
  // 返回的是一个退订的方法，将特定的 listener 从订阅者集合中删除
  return function unsubscribe () {
    // 已经退订了就不管了
    if (!isSubscribed) return;
    if (isDispatching) throw new Error ("xxx 具体信息省略")

    isSubscribed = false
    ensureCanMutateNextListeners ()
    const index = nextListeners.indexOf (listener)
    nextListeners.splice (index, 1)
  }
}
```

值得注意的是每次调用这个函数时，都会产生一个闭包，里面存储着isSubscribed的值，调用n次就会产生n个这样的闭包，用来存储n个不同的订阅情况。

接下来是 dispatch 函数：

```javascript
function dispatch (action) {
  //action 必须是一个对象
  //action.type 不能为 undefined
  if (isDispatching) {
    throw new Error ('Reducers may not dispatch actions.')
  }

  try {
    isDispatching = true
    // 执行 reducer 后返回的状态直接成为 currentState 了
    currentState = currentReducer (currentState, action)
  } finally {
    isDispatching = false
  }

  const listeners = (currentListeners = nextListeners)
  for (let i = 0; i < listeners.length; i++) {
    const listener = listeners [i]
    listener ()
  }
  return action
}
```

接下来是 replaceReducer：

```javascript
function replaceReducer (nextReducer) {
  if (typeof nextReducer !== 'function') {
    throw new Error ('Expected the nextReducer to be a function.')
  }

  currentReducer = nextReducer
  // 此时无法匹配任何的 action，但是返回的状态可以将 currentState 给更新
  // 也就是更新当前的 state，因为 reducer 更新了，老的 state 该换了
  dispatch ({ type: ActionTypes.REPLACE })
}
```



## combineReducer 做了些什么

还记得 combineReducer 的使用方式吗？

```javascript
import {combineReducer} from 'redux-immutable';
import {reducer as recommendReducer} from './xx/reducer/index';
import {reducer as singReducer} from './xx/reducer/index';

export default combineReducer({
  recommend: recommendReducer,
  sings: singReducer
});
```

combineReducer 用来组织不同模块的reducer，看下究竟是如何组织起来的。

```javascript
export default function combineReducers (reducers) {
  // 以例子来讲，reducerKeys 就是 ['recommend', 'sing']
  const reducerKeys = Object.keys (reducers)
  // finalReducers 是 reducers 过滤后的结果
  // 确保 finalReducers 里面每一个键对应的值都是函数
  const finalReducers = {}
  for (let i = 0; i < reducerKeys.length; i++) {
    const key = reducerKeys [i]
    if (typeof reducers [key] === 'function') {
      finalReducers [key] = reducers [key]
    }
  }
  const finalReducerKeys = Object.keys (finalReducers)

  // 最后依然返回一个纯函数
  return function combination (state = {}, action) {
    // 这个标志位记录初始的 state 是否和经过 reducer 后是一个引用，如果不是则 state 被改变了
    let hasChanged = false
    const nextState = {}
    for (let i = 0; i < finalReducerKeys.length; i++) {
      const key = finalReducerKeys [i]
      const reducer = finalReducers [key]
      // 原来的状态树中 key 对应的值
      const previousStateForKey = state [key]
      // 调用 reducer 函数，获得该 key 值对应的新状态
      const nextStateForKey = reducer (previousStateForKey, action)
      nextState [key] = nextStateForKey
      hasChanged = hasChanged || nextStateForKey !== previousStateForKey
    }
    // 如果没改变直接把原始的 state 返回即可
    return hasChanged ? nextState : state
  }
}
```



## compose 函数解读

compose 其实是一个工具，充分体现了高阶函数的技巧。

```javascript
export default function compose(...funcs) {
  if (funcs.length === 0) {
    return arg => arg
  }
  if (funcs.length === 1) {
    return funcs[0]
  }
  return funcs.reduce((a, b) => (...args) => a(b(...args)))
}
```



## applyMiddleware 解析

这个方法与中间件息息相关，这里直接解析不容易理解，结合上 redux-thunk 中间件为例，先看下 redux-thunk的源码

```javascript
function createThunkMiddleware(extraArgument) {
  return ({dispatch, getState}) => next => action => {
    if (typeof action === 'function') {
      return action (dispatch, getState, extraArgument)
    }
    return next(action)
  }
}
const thunk = createThunkMiddleware()
thunk.withExtraArgument = createThunkMiddleware
export default thunk
```

现在看下 applyMiddleware 的源码：

```javascript
export default function applyMiddleware (...middlewares) {
  return createStore => (...args) => {
    const store = createStore (...args)
    let dispatch = () => {
      throw new Error ('xxx')
    }
    // middlewareAPI 其实就是拿到 store 的信息
    const middlewareAPI = {
      getState: store.getState,
      dispatch: (...args) => dispatch (...args)
    }
    // 参考上面的thunk，其实就是传入store参数，
    // 剩下的部分为 next => action => { ... };
    // 传入这个参数是必须的，因为需要拿到 store 的相关属性
    // 这里的意思就是每个中间件都能拿到 store 的数据
    const chain = middlewares.map(middleware =>
      middleware (middlewareAPI)
    )
    dispatch = compose (...chain)(store.dispatch)

    return {
      ...store,
      dispatch
    }
  }
}
```

假如现在还有一个redux-logger的中间件，调用 applyMiddleware(logger, thunk)，那么走到compose逻辑的时候，相当于调用`logger(thunk(store.dispatch))`。这样就完成了中间件的机制。



## 探究 createStore 留下来的问题

在上面还没提到的关于 createStore 的第三个参数，它是如何做判断的呢，接下来描述。先给出这部分的源码：

```javascript
export default function createStore(reducer, preloadedState, enhancer) {
  // 第二个参数为函数，但是第三个参数没传
  if (
    typeof preloadedState === 'function' &&
    typeof enhancer === 'undefined'
  ) {
    enhancer = preloadedState  // 将第二个参数当做 enhancer 
    preloadedState = undefined
  }
  // 确保 enhancer 为函数
  if (typeof enhancer !== 'undefined') {
    if (typeof enhancer !== 'function') {
      throw new Error ('Expected the enhancer to be a function.')
    }
    return enhancer (createStore)(reducer, preloadedState)
  }
  //...
}
```

判断类型后返回enhancers是针对什么样的场景？

如果要用 thunk 中间件，redux官方文档是这样写的：

```
const store = createStore(reducer, applyMiddleware(thunk))
```

在这个时候其实 redux 内部的 enhancers 就变成了 applyMiddleware(thunk) 的结果了。

运行流程变成了 `applyMiddleware(thunk)(createStore)(reducer, preloadedState)`

而返回的结果赋给了 store，当前store中的dispatch属性已经成功被更改，一旦走入 dispatch，必然会经过中间件，成功集成。



### 小结

Redux 原理解读就这样了，其实理解它的源码并没有想象的复杂，最重要的还是将它的原理和使用结合起来，体会整个设计的思想，还是对自身有所帮助的。


