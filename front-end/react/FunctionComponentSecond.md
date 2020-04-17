# React源码阅读

这是 React 源码阅读的第七篇文章，有以下几点事项需要注意以下：

+ 目前阅读的 <b>React 版本为 16.13.0</b>
+ 需要辅以练习代码，在结合理解进行阅读，这样对理解react源码的帮助有所提升。
+ 文中提到的源码  均以 dev 环境下的源码, 可自行 yarn 对应包查看。

以下是阅读 React 源码的 流程：

![readAll.png](https://i.loli.net/2020/03/10/DPeKbUhVsEWGYOC.png)



## 知识点

该内容分为两个文章解析，分别讲述以下内容：

+ 第一章，讲述渲染函数组件过程中对hooks API的使用 ([详情](https://github.com/owenCoderLi/learn/blob/master/front-end/react/FunctionComponentFirst.md))
+ 第二章，将ReactElement变成fiber对象并更新的过程中对节点类型的实现(本文)



## 前言

上一篇结尾讲述到更新过程中对几种节点类型的实现，接下来来继续解析各种不同类型对应的不同函数调用。代码定位：

```javascript
function reconcileChildFibers(...) {
  // ...省略部分代码，详情请看上文
  var isObject = typeof newChild === 'object' && newChild !== null;
  // element节点
  if (isObject) {
    switch (newChild.$$typeof) {
      // ReactElement节点
      case REACT_ELEMENT_TYPE:
        return placeSingleChild(
          reconcileSingleElement(
            returnFiber, currentFirstChild, newChild, expirationTime
          )
        );
    }
  }
  // 文本节点
  if (typeof newChild === 'string' || typeof newChild === 'number') {
    return placeSingleChild(
      reconcileSingleTextNode(
        returnFiber, currentFirstChild, '' + newChild, expirationTime
      )
    );
  }

  // 数组节点
  if (isArray$1(newChild)) {
    return reconcileChildrenArray(
      returnFiber, currentFirstChild, newChild, expirationTime
    );
  }
  // 如果未符合上述element节点的要求，则报错
  if (isObject) {
    throwOnInvalidObjectType(returnFiber, newChild);
  }
  // 如果旧节点还存在，但是更新的节点是null的话，需要删除旧节点的内容
  return deleteRemainingChildren(returnFiber, currentFirstChild);
}
```



### reconcileSingleElement()

作用：

+ 当子节点不为null，则复用子节点并删除它的兄弟节点
+ 当子节点为null，则创建新的fiber节点

```javascript
function reconcileSingleElement(
  returnFiber,
  currentFirstChild, // 旧
  element, // 新
  expirationTime
) {
  var key = element.key;
  var child = currentFirstChild;
  
  // 从当前已有的所有子节点中，找到可以复用的fiber对象，并删除它们的兄弟节点
  while (child !== null) {
  
    //key 相同的话复用节点
    //ReactElement里面的key，也就是开发加的 key，如<div key={'a'}></div>
    //当多个相同element放在一组时，React 建议设置key,方便不产生更新的节点能复用
    //而且设置ReactElement的key不影响fiber对象的 key 值一直为 null
    //所以这边fiber.key等于ReactElement.key的情况,大多数应为 null===null

    if (child.key === key) {
      switch (child.tag) {
        case Fragment: {
          if (element.type === REACT_FRAGMENT_TYPE) {
            deleteRemainingChildren(returnFiber, child.sibling);
            var existing = useFiber(child, element.props.children);
            existing.return = returnFiber;
            return existing;
          }
          break;
        }
        case Block:
        default: {
          if (child.elementType === element.type ||
            (isCompatibleFamilyForHotReloading(child, element))
          ) {
          
            // 复用 child，删除它的兄弟节点
            // 因为旧节点它有兄弟节点，新节点只有它一个
            deleteRemainingChildren(returnFiber, child.sibling);
            
            // 复制 fiber 节点，并重置 index 和 sibling
            // existing就是复用的节点
            var _existing3 = useFiber(child, element.props);
            
            // 设置正确的ref
            _existing3.ref = coerceRef(returnFiber, child, element);
            
            // 父节点
            _existing3.return = returnFiber;
            return _existing3;
          }
          break;
        }
      }
      deleteRemainingChildren(returnFiber, child);
      break;
    } else {
      deleteChild(returnFiber, child);
    }
    child = child.sibling;
  }
  if (element.type === REACT_FRAGMENT_TYPE) {
  
    // 创建Fragment类型的 fiber 节点
    var created = createFiberFromFragment(
      element.props.children,
      returnFiber.mode,
      expirationTime,
      element.key
    );
    created.return = returnFiber;
    return created;
  } else {
  
    //创建Element类型的 fiber 节点
    var _created4 = createFiberFromElement(
      element,
      returnFiber.mode,
      expirationTime
    );
    _created4.ref = coerceRef(returnFiber, currentFirstChild, element);
    _created4.return = returnFiber;
    return _created4;
  }
}
```

(1) 针对child.key === ReactElement.key的情况，在开发过程中，大多数的React组件都是复用的，因为它们都是列表中的第一项

(2) 不能根据旧几点复用成新节点的话，则通过`createFiberFromElement`创建 Fragment 或 Element类型的fiber节点


### useFiber() 

作用：复制fiber节点，并重置index和sibling

```javascript
function useFiber(fiber, pendingProps) {
  var clone = createWorkInProgress(fiber, pendingProps);
  clone.index = 0;
  clone.sibling = null;
  return clone;
}
```

### createWorkInProgress()

作用：通过doubleBuffer重用未更新的fiber对象

```javascript
function createWorkInProgress(current, pendingProps) {
  var workInProgress = current.alternate;

  if (workInProgress === null) {
    // 因为fiber树顶多有两个版本,所以当某一fiber节点不更新时,在更新fiber树的时候
    // 不会去重新创建跟之前一样的 fiber 节点，而是从另一个版本的 fiber 树上重用它
    workInProgress = createFiber(
      current.tag,
      pendingProps,
      current.key,
      current.mode
    );
    workInProgress.elementType = current.elementType;
    workInProgress.type = current.type;
    workInProgress.stateNode = current.stateNode;
    workInProgress.alternate = current;
    current.alternate = workInProgress;
  } else {
    workInProgress.pendingProps = pendingProps; 
    workInProgress.effectTag = NoEffect;
    workInProgress.nextEffect = null;
    workInProgress.firstEffect = null;
    workInProgress.lastEffect = null;
  }
  workInProgress.childExpirationTime = current.childExpirationTime;
  workInProgress.expirationTime = current.expirationTime;
  workInProgress.child = current.child;
  workInProgress.memoizedProps = current.memoizedProps;
  workInProgress.memoizedState = current.memoizedState;
  workInProgress.updateQueue = current.updateQueue; 

  var currentDependencies = current.dependencies;
  workInProgress.dependencies = currentDependencies === null 
    ? null 
    : {
      expirationTime: currentDependencies.expirationTime,
      firstContext: currentDependencies.firstContext,
      responders: currentDependencies.responders
    };

  workInProgress.sibling = current.sibling;
  workInProgress.index = current.index;
  workInProgress.ref = current.ref;

  return workInProgress;
}
```

### reconcileSingleTextNode()

作用：复用或创建文本节点

```javascript
function reconcileSingleTextNode(
  returnFiber,
  currentFirstChild,
  textContent,
  expirationTime
) {
  //判断第一个节点
  if (
    currentFirstChild !== null &&
    currentFirstChild.tag === HostText
  ) {

    //删掉旧节点
    //returnFiber是当前正在更新的节点
    //currentFirstChild是第一个子节点
    deleteRemainingChildren(returnFiber, currentFirstChild.sibling);

    // 复用
    var existing = useFiber(currentFirstChild, textContent);

    // 指定父节点
    existing.return = returnFiber;
    return existing;
  }

  // 如果第一个节点不是文本节点的话，删除所有
  deleteRemainingChildren(returnFiber, currentFirstChild);
  // 作用创建文本类型的fiber
  var created = createFiberFromText(
    textContent,
    returnFiber.mode,
    expirationTime
  );
  created.return = returnFiber;
  return created;
}
```

### reconcileChildrenArray()

作用：更新数组节点

```javascript
function reconcileChildrenArray(
  returnFiber,
  currentFirstChild,
  newChildren, // 待更新的数组节点
  expirationTime
) {
  var resultingFirstChild = null;
  var previousNewFiber = null;
  // 数组中的第一个节点
  var oldFiber = currentFirstChild;
  var lastPlacedIndex = 0;
  var newIdx = 0;
  var nextOldFiber = null;

  // 复用节点的时候，会尽量减少数组遍历的次数
  // 跳出循环的条件是，在遍历新老数组的过程中,找到第一个不能复用的节点
  for (; oldFiber !== null && newIdx < newChildren.length; newIdx++) {
    // 当要更新的节点的 index 大于 newIndex 时，
    // 说明它不在所期盼的位置上，则需要“认真处理”oldFiber
    if (oldFiber.index > newIdx) {
      nextOldFiber = oldFiber;
      oldFiber = null;
    }
    // 否则，则处理该节点的下一个兄弟节点
    else {
      nextOldFiber = oldFiber.sibling;
    }

    // 复用或新建节点
    var newFiber = updateSlot(
      returnFiber, oldFiber,
      newChildren[newIdx], expirationTime
    );

    // 说明key不相同，节点不能复用，此时就跳出循环
    // 如果不跳出循环，说明可以是相同的
    // 也就是说当跳出循环的时候，可以知道目前复用节点的个数和不可复用节点的index
    if (newFiber === null) {
      if (oldFiber === null) {
        oldFiber = nextOldFiber;
      }
      break;
    }

    // 初次渲染
    if (shouldTrackSideEffects) {
      // newFiber.alternate表示并没有复用oldFiber来赋值,而是return了新的fiber
      if (oldFiber && newFiber.alternate === null) {
        // 删除存在的 旧的fiber
        deleteChild(returnFiber, oldFiber);
      }
    }

    // 将newFiber节点挂载到DOM树上,返回最新移动的index
    lastPlacedIndex = placeChild(newFiber, lastPlacedIndex, newIdx);

    // 表示是新节点
    if (previousNewFiber === null) {
      resultingFirstChild = newFiber;
    } else {
      previousNewFiber.sibling = newFiber;
    }
    previousNewFiber = newFiber;
    oldFiber = nextOldFiber;
  }

  // 所有节点都是可以复用的,可以删除老节点
  if (newIdx === newChildren.length) {
    deleteRemainingChildren(returnFiber, oldFiber);
    return resultingFirstChild;
  }

  if (oldFiber === null) {
    // 老节点已经被复用完,但是仍有部分新节点没有被创建
    for (; newIdx < newChildren.length; newIdx++) {
      // 新建节点
      var _newFiber = createChild(
        returnFiber,
        newChildren[newIdx],
        expirationTime
      );
      if (_newFiber === null) {
        continue;
      }
      lastPlacedIndex = placeChild(_newFiber, lastPlacedIndex, newIdx);
      if (previousNewFiber === null) {
        resultingFirstChild = _newFiber;
      } else {
        previousNewFiber.sibling = _newFiber;
      }
      previousNewFiber = _newFiber;
    }
    return resultingFirstChild;
  }

  // 数组可能存在顺序变化,oldFiber和newFiber还有可以复用的fiber节点
  var existingChildren = mapRemainingChildren(returnFiber, oldFiber);

  // 继续遍历剩下的new节点
  for (; newIdx < newChildren.length; newIdx++) {
    var _newFiber2 = updateFromMap(existingChildren, returnFiber, newIdx, newChildren[newIdx], expirationTime);
    if (_newFiber2 !== null) {
      if (shouldTrackSideEffects) {
        if (_newFiber2.alternate !== null) {
          existingChildren.delete(_newFiber2.key === null
            ? newIdx 
            : _newFiber2.key
          );
        }
      }
      lastPlacedIndex = placeChild(_newFiber2, lastPlacedIndex, newIdx);
      if (previousNewFiber === null) {
        resultingFirstChild = _newFiber2;
      } else {
        previousNewFiber.sibling = _newFiber2;
      }
      previousNewFiber = _newFiber2;
    }
  }
  if (shouldTrackSideEffects) {
    // 删除没有复用的节点
    existingChildren.forEach(function (child) {
      return deleteChild(returnFiber, child);
    });
  }
  return resultingFirstChild;
}
```

1. 在循环数组节点中，主要做了以下几点操作：

  + 将oldFiber的 index 与 newIdx 进行比较，如果前者大，则将 oldFiber赋值给 nextOldFiber，如果后者大，则将 oldFiber.sibling赋值给 nextOldFiber。
  +  执行`updateSlot()`，复用或新建节点，返回的结果赋值给 newFiber
  + 如果 newFiber 的值为空时，说明该节点不能复用，跳出循环。
  + 如果是第一次渲染(`shouldTrackSideEffects`为true)，并且 newFiber 没有要复用的 oldFiber 的话，则删除该 fiber下的所有子节点。
  + 执行 `placeChild`，将 newFiber 节点挂载到DOM树上，并判断更新后是否移动过。如果移动，则需要重新挂载，返回最新移动的 index，并赋值给 lastPlacedIndex。

2. 在跳出循环后，如果 newIdx 和更新数组长度相等，则表示所有节点都可以复用，那么就执行 `deleteRemainingChildren()`，删除旧节点。

3. 如果旧节点已经被复用完了，但是仍有部分新节点需要被创建的话，则循环剩余数组的长度，并依次创建新节点。

4. 如果仍有旧节点剩余的话，则执行`mapRemainingChildren`方法，将这些旧节点用Map结构集合起来，看有没有方便 newFiber 复用的节点。

5. 继续执行剩下的 new 节点
  + 执行`updateFromMap`，查找有没有 key/index 相同的点，方便复用

6. 如果是第一次渲染的话，则删除没有复用的节点。
7. 最终返回更新后的数组的第一个节点(根据它的sibling属性，可找到其他节点)



以上就是关于 FunctionComponent的实现解析。