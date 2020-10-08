# 链表

## 定义

链表存储有序的元素集合，但不同于数组，链表中的元素在内存中并不是连续放置的，每个元素由一个存储元素本身的节点和一个指向下一个元素的引用组成。

相对于传统的数组，链表的好处在于，添加或移除元素时不需要移动其他元素，然而链表需要使用指针。在数组中可以直接访问任何位置的任何元素，在链表中，则需要从起点开始迭代链表找到所需的元素。

### 1. 创建链表

```javascript
function defaultEquals(a, b) { // 比较对象或值是否相等
  return a === b
}

class Node { // 表示想要添加到链表中的项
  constructor(element) {
    this.element = element
    this.next = undefined
  }
}

class LinkedList {
  constructor(equalsFn = defaultEquals) {
    this.count = 0
    this.head = undefined // 第一个元素的引用
    this.equalsFn = equalsFn
  }
}
```

### 1.1. 向链表尾部添加元素

向LinkedList对象尾部添加一个元素时，可能有两种场景：链表为空，添加的是第一个元素；链表不为空，追加元素。

```javascript
push(element) {
  const node = new Node(element) // 创建node项
  let current // 指向链表中的变量
  if(this.head == null) { // 如果head元素是null/undefined
    this.head = node // head指向node元素
  } else {
    current = this.head // 第一个元素的引用
    while(current.next != null) { // 循环找到最后一项
      current = current.next
    }
    current.next = node // 当前next指针指向想要添加的链表节点
  }
  this.count++
}
```

当一个Node实例被创建时，它的next指针总是undefined，因为知道它会是链表的最后一项。

### 1.2. 从链表中移除元素

* 从特定位置移除一个元素(removeAt)
* 根据元素的值移除元素

```javascript
removeAt(index) {
  if(index >= 0 && index < this.count) { // 验证index有效性
    let current = this.head // 对链表中第一个元素的引用
    if(index === 0) { // 链表中移除第一个元素
      this.head = current.next
    } else {
      const previous = this.getElementAt(index - 1)
      current = previous.next
      previous.next = current.next
    }
    this.count--
    return current.element
  }
  return undefined // index无效位置旧返回
}

remove(element) {
  const index = this.indexOf(element)
  return this.removeAt(index)
}

```

### 1.3. 循环迭代链表到目标位置

```javascript
getElementAt(index) {
  if(index >= 0 && index <= this.count) {
    let node = this.head // 从链表的第一个元素head迭代
    for(let i = 0; i < index && node != null; i++) { // 直到目标index
      node = node.next
    }
    return node
  }
  return undefined
}
```

### 1.4. 在任意位置插入元素

```javascript
insert(element, index) {
  if(index >= 0 && index <= this.count) {
    const node = new Node(element)
    if(index === 0) {
      const current = this.head
      node.next = current
      this.head = node
    } else {
      const previous = this.getElementAt(index - 1)
      const current = previous.next
      node.next = current
      previous.next = node
    }
    this.count++
    return true
  }
  return false
}
```

### 1.5. 返回一个元素的位置

```javascript
indexOf(element) {
  let current = this.head
  for(let i = 0; i < this.count && current != null; i++) {
    if(this.equalsFn(element, current.element)) { // 迭代时验证current节点的元素和目标元素是否相等
      return i
    }
    current = current.next // 如果不是目标元素就迭代下一个链表节点
  }
  return -1 // 链表为空或没找到目标
}
```

### 2. 双向链表

双向链表跟普通链表区别：在链表中，一个节点只有链向下一个节点的链接；而在双向链表中，链接是双向的；一个链向下一个元素，另一个链向前一个元素。

### 2.1 实现

```javascript
class DoublyNode extends Node {
  constructor(element, next, prev) {
    super(element, next)
    this.prev = prev
  }
}

class DoublyLinkedList extends LinkedList {
  constructor(equalsFn = defaultEquals) {
    super(equalsFn)
    this.tail = undefined // 对链表最后一个元素的引用
  }
}
```

### 2.2 任意位置插入新元素

双向链表中插入一个新元素跟链表类似，区别在于：链表只要控制一个next指针，而双向链表则要同时控制next跟prev两个指针

```javascript
insert(element, index) {
  if(index >= 0 && index <= this.count) {
    const node = new DoublyNode(element)
    let current = this.head
    if(index === 0) {
      if(this.head === null) { // 如果链表为空，head/tail指向新节点
        this.head = node;
        this.tail = node;
      } else {
        node.next = this.head
        current.prev = node // 由undefined指向新元素
        this.head = node // 变成双向链表中的第一个元素
      }
    } else if(index === this.count) {
      current = this.tail // 引用最后一个元素
      current.next = node // node -> undefined
      node.prev = current
      this.tail = node // 更新tail
    } else {
      const previous = this.getElementAt(index - 1) // 找到位置
      current = previous.next // 在current和previous插入新元素
      node.next = current
      previous.next = node // 保证不会丢失节点之间的链接
      current.prev = node
      node.prev = previous
    }
    this.count++
    return true
  }
  return false
}
```

### 2.3. 任意位置移除元素

```javascript
removeAt(index) {
  if(index >= 0 && index < this.count) {
    let current = this.head
    if(index === 0) { // 头部移除元素
      this.head = current.next // 改变head引用，为下一个元素
      if(this.count === 1) { // 检查要移除的元素是否是第一个元素
        this.tail = undefined
      } else {
        this.head.prev = undefined // 指向双向链表中新的第一个元素
      }
    } else if(index === this.count - 1) { // 从最后一个位置移除元素
      current = this.tail // 最后一个元素的引用
      this.tail = current.prev // 把tail的引用更新为链表的倒数第二个元素
      this.tail.next = undefined // next指向undefined
    } else { // 从双向链表中间移除元素
      current = this.getElementAt(index - 1)
      const previous = current.prev
      previous.next = current.next
      current.next.prev = previous
    }
    this.count--
    return current.element
  }
  return undefined
}
```

### 3.循环链表

循环链表像链表一样只有单向引用，也可以像双向链表一个双向引用。区别在于：最后一个元素指向下一个元素的指针不是引用undefined，而是指向第一个元素head。

```javascript
class CircularLinkedList extends LinkedList {
  constructor(equalsFn = defaultEquals) {
    super(equalsFn)
  }
}
```

### 3.1 在任意位置插入新元素

```javascript
insert(element, index) {
  if(index >= 0 && index <= this.count) {
    const node = new Node(element)
    let current = this.head
    if(index === 0) { // 第一个位置插入新元素
      if(this.head == null) {
        this.head = node
        node.next = this.head
      } else {
        node.next = current
        current = this.getElementAt(index - 1)
        this.head = node
        current.next = this.head
      }
    } else { // 非空循环链表的第一个位置插入元素
      const previous = this.getElementAt(index - 1)
      node.next = previous.next
      previous.next = node
    }
    this.count++
    return true
  }
  return false
}
```

### 3.2. 从任意位置移除元素

```javascript
removeAt(index) {
  if(index >= 0 && index < this.count) {
    let current = this.head
    if(index === 0) {
      if(this.size() === 1) {
        this.head = undefined
      } else {
        const removed = this.head
        current = this.getElementAt(this.size())
        this.head = this.head.next
        current.next = this.head
        current = removed
      }
    } else {
      const previous = this.getElementAt(index - 1)
      current = previous.next
      previous.next = current.next
    }
    this.count--
    return current.element
  }
  return undefined
}
```
