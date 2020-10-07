# 栈

## 定义

栈是一种遵从后进先出(LIFO)原则的有序集合。新添加或待删除的元素都保存在栈的同一端，称为栈顶，另一端就叫栈底。在栈里，新元素都靠近栈顶，旧元素都接近栈底。

### 1. 创建一个基于数组的栈

创建一个类来表示栈：

```javascript
class Stack {
  constructor() {
    this.items = []; // 需要一种数据结构保存栈里的元素
  }
}
```

接下来要为栈声明一些方法:

* push(elements(s)): 添加一个或多个新元素到栈顶
* pop(): 移除栈顶的元素，同时返回被移除的元素
* peek(): 返回栈顶的元素，不对栈做任何修改
* isEmpty(): 如果栈里没有任何元素就会返回true，反之false
* clear(): 移除栈里的所有元素
* size(): 返回栈里的元素个数，该方法和数组的length属性类似。

### 1.1.添加元素

该方法负责往栈里添加新元素(只添加元素到栈顶，也就是栈末尾)

```javascript
push(element) {
  this.items.push(element)
}
```

### 1.2. 移除元素

该方法主要用来移除栈里的元素。因遵从LIFO原则，因此移出的是最后添加进去的元素。

```javascript
pop() {
  return this.items.pop()
}
```

### 1.3. 查看栈顶元素

```javascript
peek() {
  return this.items[this.items.length - 1]
}
```

### 1.4. 检查栈是否为空

```javascript
isEmpty() {
  return this.items.length === 0
}
```

使用isEmpty方法，能简单的判断内部数组的长度是否为0。
类似于数组的length属性，对于集合，最好使用size代替length。因为栈的内部使用数组保存元素，所以能简单的返回栈的长度。

```javascript
size() {
  return this.items.length
}
```

### 1.5. 清空栈元素

```javascript
clear() {
  this.items = []
}
```

### 1.6. 使用

```javascript
const stack = new Stack();
stack.isEmpty(); // true
stack.push(1);
stack.push(2);
stack.peek(); // 2
stack.size(); // 2
stack.isEmpty(); // false
stack.pop(); -> stack.size(); // 1
```

### 2. 创建一个基于Javascript对象的Stack类

创建一个stack类最简单的方式是使用一个数组来存储元素。在处理大量数据的时候，同样需要评估如何操作数据是最高效的，在使用数组时，大部分方法的时间复杂度是O(n)。在最坏的情况下需要迭代数组的所有位置。其中n代表数组的长度。另外，数组时元素的一个有序集合，为了保证元素排列有序，会占用更多的内存空间。

如果能直接获取元素，占用较少的内存空间，并且仍然保证所有元素按照需要排列是否更好？可以使用一个Javascript对象来存储所有的栈元素，保证它们的顺序并且遵循LIFO原则。

```javascript
class Stack {
  constructor() {
    this.count = 0 // 帮助记录栈的大小
    this.items = {}
  }
}
```

### 2.1 栈插入元素

由于现在使用了一个对象，所以push方法只允许一次插入一个元素

```javascript
push(element) {
  this.items[this.count] = element
  this.count++
}
```

在Javascript中，对象时一系列键值对的集合，要向栈中添加元素，将使用count变量作为items对象的键名，插入的元素是它的值，插入元素后，递增count变量.

### 2.2. 验证栈是否为空和大小

```javascript
size() {
  return this.count
}

isEmpty() {
  return this.count === 0
}
```

### 2.3. 栈弹出元素

由于没有使用数组存储元素，需要手动实现移除元素的逻辑:

```javascript
pop() {
  if(this.isEmpty()) { // 检验栈是否为空
    return undefined
  }
  this.count--
  const result = this.items[this.count] // 保存栈顶的值
  delete this.items[this.count] // 删除值
  return result
}
```

### 2.4. 查看栈顶的值并将栈清空

```javascript
peek() {
  if(this.isEmpty()) {
    return undefined
  }
  return this.items[this.count - 1]
}
```

要清空该栈，只需要将它的值复原为构造函数中使用的值即可。

```javascript
clear() {
  this.items = {}
  this.count = 0
}
```

也可以遵循LIFO原则，使用下面逻辑移除栈中所有的元素。

```javascript
while(!this.isEmpty()) {
  this.pop()
}
```

### 2.5. 创建toString方法

```javascript
toString() {
  if(this.isEmpty()) {
    return ''
  }
  let objString = `${this.items[0]}` // 第一个元素作为字符串的初始值
  for(let i = 1; i < this.count; i++) {
    objString = `${objString}, ${this.items[i]}`
  }
  return objString
}
```

### 3. 保护数据结构内部元素

在创建别的开发者也可以使用的数据结构或对象时，希望保护内部的元素，只有暴露出的方法才能修改内部结构。对于Stack类来说，要确保元素只会被添加到栈顶，而不是栈底或其他位置

有一种数据类型可以确保属性是私有的，就是ES6中的WeakMap:

```javascript
const item = new WeakMap()
class Stack {
  constructor() {
    items.set(this, []) // 以this为键，把代表栈的数组存入items
  }
  push(element) {
    const s = items.get(this) // 从weakmap中取出值
    s.push(element)
  }
  pop() {
    const s = items.get(this)
    const r = s.pop()
    return r
  }
}
```