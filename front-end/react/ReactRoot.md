# React源码阅读

这是 React 源码阅读的第一篇文章，有以下几点事项需要注意以下：

+ 目前阅读的 <b>React 版本为 16.13.0</b>
+ 需要辅以练习代码，在结合理解进行阅读，这样对理解react源码的帮助有所提升。
+ 文中提到的源码  均以 dev 环境下的源码, 可自行 yarn 对应包查看。

以下是阅读 React 源码的 流程：

![readAll.png](https://i.loli.net/2020/03/10/DPeKbUhVsEWGYOC.png)



## React.createElement

大多数写React代码时，会采用 JSX，但是为什么一旦使用 JSX 就必须引入 React ?
这是因为 JSX 代码会被 Babel 编译成 `React.createElement`，不引入 React则不能使用 `React.createElement`了。

```
<div id="read">react</div>
// 上面的 JSX 会被babel编译成下方代码
React.createElement(‘div’, {
  id: 'read'
}, 'react')
```

那么就定位代码到  `react/cjs/react.development.js` 下,看下 `createElement`函数的实现

`function createElement(type, config, children) {...}`

首先， `createElement`函数接收三个参数，分别为以下解释:

+ type: 指代这个 ReactElement 的类型
+ config: 所有节点的属性都会以key:value的形式放在config对象中
+ children: 子节点不止会有一个, 所以children不只有一个, 从第二个参数以后都是children, 它是一个数组

然后是对于config的处理：

```javascript
if (config != null) {
  if (hasValidRef(config)) {
    ref = config.ref;
  }
  if (hasValidKey(config)) {
    key = '' + config.key;
  }
  self = config.__self === undefined ? null : config.__self;
  source = config.__source === undefined ? null : config.__source;
  for (propName in config) {
    if (
      hasOwnProperty.call(config, propName) &&
      !RESERVED_PROPS.hasOwnProperty(propName)
    ) {
      props[propName] = config[propName];
    }
  }
}
```

这段代码对 ref 以及 key 做了个验证，然后遍历config 并把内建的几个属性 剔除后丢到 props对象中。

下面是对 children 的操作

```javascript
var childrenLength = arguments.length - 2;
if (childrenLength === 1) {
  props.children = children;
} else if (childrenLength > 1) {
  var childArray = Array(childrenLength);
  for (var i = 0; i < childrenLength; i++) {
    childArray[i] = arguments[i + 2];
  }
  {
    if (Object.freeze) {
      Object.freeze(childArray);
    }
  }
  props.children = childArray;
}
```

首先把第二个参数之后的参数取出来，然后判断长度是否大于1，大于1的话就代表有多个 children，这时候 props.children 会是一个数组，否则的话就是一个对象， 因此需要注意在对 props.children 进行遍历时 注意它是否为数组。

最后返回了一个 ReactElement 对象。

```javascript
let ReactElement = function(type, key, ref, self, source, owner, props){
  var element = {
    $$typeof: REACT_ELEMENT_TYPE,
    type: type,
    key: key,
    ref: ref,
    props: props,
    _owner: owner
  };
  return element;
}
```

核心就是通过 `$$typeof` 来帮助识别这是一个 ReactElement，后面可以看到很多这样类似的类型，另外需要注意的一点是:  通过 JSX 写的 <App />代表着 ReactElement，App代表着 ReactComponent。

以下是流程内容：

![createElement.png](https://i.loli.net/2020/03/05/iURAyrXW8TBsx6p.png)



## ReactBaseClasses

上面讲到 App 代表着 ReactComponent，那么这里就来阅读这一块内容的源码。

内部代码相对简单，因为 React的开发团队将复杂的逻辑都放在了 react-dom 中，可以把 react-dom 看成 React 和 UI 之间的胶水层，这一层可以兼容很多平台，比如 Web, RN, SSR等。

这里包含了两个基本组件，分别是 Component 以及 PureComponent。首先先看 Component代码。

```
function Component(props, context, updater) {
  this.props = props;
  this.context = context;
  this.refs = emptyObject;
  this.updater = updater || ReactNoopUpdateQueue;
}
Component.prototype.isReactComponent = {};
Component.prototype.setState = function (partialState, callback) {
  this.updater.enqueueSetState(
    this,
    partialState,
    callback,
    'setState'
  );
};
Component.prototype.forceUpdate = function (callback) {
  this.updater.enqueueForceUpdate(this, callback, 'forceUpdate');
};
```

构造函数 Component 中需要注意的两点分别是 refs 和 updater，前者在下一篇文章中会介绍，后者是组件中相当重要的一个属性，可以发现 setState 和 forceUpdate 都是调用了updater 中的方法，但是 updater 是react-dom 中的内容，会在之后的文章中学习到。

另外 ReactNoopUpdateQueue 的源码，主要用于报警告的，所以暂时跳过不看。

接下来是 PureComponent 的源码。基本上与 Component 一致。

```
function ComponentDummy() {}
ComponentDummy.prototype = Component.prototype;
function PureComponent(props, context, updater) {
  this.props = props;
  this.context = context;
  this.refs = emptyObject;
  this.updater = updater || ReactNoopUpdateQueue;
}
var pureComponentPrototype
  = PureComponent.prototype
  = new ComponentDummy();
pureComponentPrototype.constructor = PureComponent;
_assign(
  pureComponentPrototype,
  Component.prototype
); // assign -> Object.assign()
pureComponentPrototype.isPureReactComponent = true;
```

PureComponent 继承自 Component， 继承方法使用了寄生组合式。

另外则两部分源码中，每个组件都有一个 isXXXXX 属性用来标志自身属于什么组件。

以上就是关于 Component 与 PureComponent 的源码。



## Refs

refs有好几种方式可以创建:

+ 字符串的形式，但是这种做法已经不推荐使用
+ `ref = {el => this.el = el}`
+ `React.createRef`

首先看下 createRef () 的源码

```
function createRef() {
  var refObject = {
    current: null
  };
  { Object.seal(refObject); }
  return refObject;
}
```

内部实现很简单，如果想使用 ref, 只需要取出其中的 current 对象即可。
还有一种是通过 props 的方式传递 ref，但是现在有了新的方式 forwardRef 去解决。

```
function forwardRef(render) {
  return {
    $$typeof: REACT_FORWARD_REF_TYPE,
    render: render
  };
}
```

这部分代码最重要的就是可以在参数中获取ref，因此如果想在函数组件中使用 ref 的话就可以写成：

```
const Button = React.forwardRef((props, ref) => (
  <button ref={ref} className="button">
    {props.children}
  </button>
))
```



## ReactChildren

这部分是这篇文章中相对复杂的一环，需要写代码配合Debug能更好理解为什么这么实现。

该部分的核心在于 mapChildren 函数中的内容。对于mapChildren 这个函数来说，通常会使用在组合组件设计模式上，比如 Antd中的 Radio.Group、Radio.Button等。

先来看下该函数的用法：

```
React.Children.map(this.props.children, c => [[c, c]])
```

对于上述代码，map也就是mapChildren函数来说返回值是 [c, c, c, c]。不管第二个参数的函数返回值是几维嵌套数组，map函数都能简化成一维数组。并且每次遍历后返回的数组中元素个数代表同一个节点需要复制几次。

通过示例代码解释：

```
<div>
  <span>1</span>
  <span>2</span>
</div>

// 通过 c => [[c, c]] 会变成以下:
<span>1</span>
<span>1</span>
<span>2</span>
<span>2</span>
```

接下来看下 mapChildren 内部的实现方法:

```
function mapChildren(children, func, context) {
  if (children == null) {
    return children;
  }
  var result = [];
  mapIntoWithKeyPrefixInternal(children, result, null, func, context);
  return result;
}

function mapIntoWithKeyPrefixInternal(
  children,
  array,
  prefix,
  func,
  context
) {
  var escapedPrefix = '';
  if (prefix != null) {
    escapedPrefix = escapeUserProvidedKey(prefix) + '/';
  }
  var traverseContext = getPooledTraverseContext(
    array,
    escapedPrefix,
    func,
    context
  );
  traverseAllChildren(
    children,
    mapSingleChildIntoContext,
    traverseContext
  );
  releaseTraverseContext(traverseContext);
}
```

这段代码引入了对象重用池的概念，分别是 `getPooledTraverseContext`和 `releaseTraverseContext`，这个概念的意思其实就是为了一个大小固定的对象重用池，每次从这个池子里取一个对象去赋值，用完了就将对象上的属性置空然后丢回池子。维护这个池子的用意就是提高性能，因为频繁地创建销毁一个有很多属性的对象会消耗性能。

接下来看到 `traverseAllChildren` 方法中的 `traverseAllChildrenImpl`，这部分源码需要分为两块来阅读。

```
function traverseAllChildrenImpl(children, nameSoFar, callback, traverseContext) {
  var type = typeof children;
  if (type === 'undefined' || type === 'boolean') {
    children = null;
  }
  var invokeCallback = false;
  
  if (children === null) {
    invokeCallback = true;
  } else {
    switch (type) {
      case 'string':
      case 'number':
        invokeCallback = true;
        break;
      case 'object':
        switch (children.$$typeof) {
          case REACT_ELEMENT_TYPE:
          case REACT_PORTAL_TYPE:
            invokeCallback = true;
        }

    }
  }

  if (invokeCallback) {
    callback(
      traverseContext,
      children,
      nameSoFar === ''
        ? SEPARATOR + getComponentKey(children, 0)
        : nameSoFar
    );
    return 1;
  }
}
```

这部分代码主要是在判断 children 的类型是什么，如果是可以渲染的节点的话，就直接调用 callback， 另外还可以发现在判断的过程中，代码中有使用到 `$$typeof` 去判断的流程。这里的 callback 指的是 `mapSingleChildIntoContext` 函数。这部分会在下面讲述到。

```
function traverseAllChildrenImpl(
  children,
  nameSoFar,
  callback,
  traverseContext
) {
  // ...
  if (Array.isArray(children)) {
    for (var i = 0; i < children.length; i++) {
      child = children[i];
      nextName = nextNamePrefix + getComponentKey(child, i);
      subtreeCount += traverseAllChildrenImpl(
        child,
        nextName,
        callback,
        traverseContext
      );
    }
  } else {
    var iteratorFn = getIteratorFn(children);
    if (typeof iteratorFn === 'function') {
      var iterator = iteratorFn.call(children);
      var step;
      var ii = 0;
      while(!(step = iterator.next()).done) {
        child = step.value;
        nextName = nextNamePrefix + getComponentKey(child, ii++);
        subtreeCount += traverseAllChildrenImpl(
          child,
          nextName,
          callback,
          traverseContext
        );
      }
    }
  }
}
```

这部分代码首先会判断 children 是否为数组，如果为数组的话，就遍历数组并把其中的每个元素都递归调用 `traverseAllChildrenImpl`，也就是说必须是单个可渲染节点才可以执行上述代码中的callback。

如果不是数组的话，就要看 children 是否可以支持迭代。原理就是通过 obj[Symbol.iterator]的方式去取迭代器，返回值如果是个函数的话就代表支持迭代，然后逻辑就和之前的一样了。

上述阅读完 `traverseAllChildrenImpl`函数，接下来看一下 `mapSingleChildIntoContext`函数中的实现。

```
function mapSingleChildIntoContext(bookKeeping, child, childKey) {
  const {result, keyPrefix, func, context} = bookKeeping;
  var mappedChild = func.call(context, child, bookKeeping.count++);

  if (Array.isArray(mappedChild)) {
    mapIntoWithKeyPrefixInternal(mappedChild, result, childKey, c => c);
  } else if (mappedChild != null) {
    if (isValidElement(mappedChild)) {
      mappedChild = cloneAndReplaceKey(
        mappedChild,
        keyPrefix + 
          (mappedChild.key && (!child || child.key !== mappedChild.key)
            ? escapeUserProvidedKey(mappedChild.key) + '/'
            : '') + childKey);
    }
    result.push(mappedChild);
  }
}
```

bookingKeeping 就是从对象池子里面取出来的东西，然后调用func并且传入节点(此时这个节点肯定是单个节点)，此时的func代表着 React.mapChildren 中的第二个参数。

接下来就是判断返回值类型的过程：如果是数组的话，还是回归之前的代码逻辑，这里传入的 func 是 c => c，因为要保证最终结果是被摊平的，如果不是数组的话，判断返回值是否是一个有效的 Element，验证通过的话就 clone 一份并且替换掉key，最后把返回值放入 result中，result其实也就是 mapChildren的返回值。

现在通过流程图描述清楚 mapChildren函数的过程。

![reactchildren.png](https://i.loli.net/2020/03/05/qjxW3fY9mU2QLJd.png)



以上就是关于 React 的部分API进行了描述与解析。










