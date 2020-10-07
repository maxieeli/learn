# 队列

## 定义

队列是一种遵从先进先出(FIFO)原则的有序项。队列在尾部添加新元素，并从顶部移除元素，最新添加的元素必须排在队列的末尾。

### 1. 创建队列

```javascript
class Queue {
  constructor() {
    this.count = 0 // 控制队列的大小
    this.lowestCount = 0 // 追踪第一个元素
    this.items = {} // 存储元素
  }
}
```

### 1.1 向队列添加元素

新添加的项只能添加到队列末尾

```javascript
enqueue(element) {
  this.items[this.count] = element
  this.count++
}
```

### 1.2 从队列移除元素

由于队列是先进先出，最先添加的项也是最先被移除的。

```javascript
dequeue() {
  if(this.isEmpty()) {
    return undefined
  }
  const result = this.items[this.lowestCount] // 暂存队列头部的值
  delete this.items[this.lowestCount]
  this.lowestCount++
  return result
}
```

### 1.3 查看队列头元素

该方法会返回队列最前面的项(把lowestCount作为键名来获取元素值)

```javascript
peek() {
  if(this.isEmpty()) {
    return undefined
  }
  return this.items[this.lowestCount]
}
```

### 1.4 检查队列是否为空并获取长度

```javascript
isEmpty() {
  return this.count - this.lowestCount === 0
}

size() {
  return this.count - this.lowestCount
}
```

## 双端队列数据结构

### 定义

双端队列是一种允许同时从前端后端添加和移除元素的特殊队列。由于双端队列同时遵守了先进先出和后进先出原则，可以说它是把队列和栈香结合的一种数据结构。

### 2.1 创建Deque类

```javascript
class Deque {
  constructor() {
    this.count = 0
    this.lowestCount = 0
    this.items = {}
  }
}
```

### 2.2  向双端队列的前端添加元素

```javascript
addFront(element) {
  if(this.isEmpty()) {
    this.addBack(element)
  } else if(this.lowestCount > 0) { // 元素已被从队列的前端移除
    this.lowestCount--
    this.items[this.lowestCount] = element
  } else {
    for(let i = this.count; i > 0; i--) { // 将所有元素后移一位空出第一位置
      this.items[i] = this.items[i - 1]
    }
    this.count++
    this.lowestCount = 0
    this.items[0] = element // 新元素覆盖
  }
}
```

### 应用

回文检查器，回文：正反都能读通的单词，词组，数或者一系列字符的序列。

```javascript
function palindromeCheck(aString) {
  if(
    aString === undefined ||
    aString === null ||
    (aString !== null && aString.length === 0)
  ) {
    return false
  }
  const deque = new Deque()
  const lowerString = aString.toLocaleLowerCase().split(' ').join('')
  let isEqual = true
  let firstChar, lastChar
  for(let i = 0; i < lowerString.length; i++) {
    deque.addBack(lowerString.charAt(i))
  }
  while(deque.size() > 1 && isEqual) {
    fistChar = deque.removeFront()
    lastChar = deque.removeBack()
    if(firstChar !== lastChar) {
      isEqual = false
    }
  }
  return isEqual
}
```