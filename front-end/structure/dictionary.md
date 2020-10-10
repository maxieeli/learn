# 字典

## 定义

在字典中，存储的是[键，值]对，其中键名是用来查询特定元素的，与集合相似，字典也称作映射，关联数组

### 1.1 创建字典类

```javascript
class Dictionary {
  constructor(toStrFn = defaultToString) {
    this.toStrFn = toStrFn // 将key转化为字符串的函数
    this.table = {} // object实例存储字典中的元素
  }
}

function defaultToString(item) {
  if(item === null) {
    return 'NULL'
  } else if(item === undefined) {
    return 'UNDEFINED'
  } else if(typeof item === 'string' || item instanceof String) {
    return `${item}`
  }
  return item.toString()
}
```

### 1.2. 检测一个键是否存在字典中

```javascript
hasKey(key) {
  return this.table[this.toStrFn(key)] != null
}
```

### 1.3. 字典设置键和值

```javascript
set(key, value) {
  if(key != null && value != null) {
    const tableKey = this.toStrFn(key)
    this.table[tableKey] = new ValuePair(key, value) // 创建一个新的键值对并将其赋值给table对象上的key属性
    return true
  }
  return false
}

class ValuePair {
  constructor(key, value) {
    this.key = key
    this.value = value
  }
  toString() {
    return `[${this.key}: ${this.value}]`
  }
}
```

### 1.4. 从字典移除值

```javascript
remove(key) {
  if(this.hasKey(key)) {
    delete this.table[this.toStrFn(key)]
    return true
  }
  return false
}
```

### 1.5. 从字典检索值

```javascript
get(key) {
  const valuePair = this.table[this.toStrFn(key)] // 检索存储在给定key属性中的对象
  return valuePair === null ? undefined : valuePair.value // 如果存在返回该值
}
```

### 1.6. keys/values/valuePairs 方法

* valuePairs：以数组形式返回字典中的所有valuePair对象。
* keys：返回Dictionary类中用于识别值的所有键名。
* values：返回一个字典包含的所有值构成的数组。

```javascript
keyValues() {
  return Object.values(this.table)
}

keyValues() { // 非Object处理
  const valuePairs = []
  for(const k in this.table) {
    if(this.hasKey(k)) {
      valuePairs.push(this.table[k])
    }
  }
  return valuePairs
}

keys() {
  return this.keyValues().map(pair => pair.key)
}

values() {
  return this.keyValues().map(pair => pair.value)
}
```

### 2. 散列表

散列算法的作用是尽可能快的在数据结构中找到一个值。使用散列函数，就知道值的具体位置，因此能够快速检索到该值，散列函数的作用是给定一个键值，然后返回值在表中的地址。

### 2.1. 创建散列表

```javascript
class HashTable {
  constructor(toStrFn = defaultToString) {
    this.toStrFn = toStrFn
    this.table = {}
  }
}
```

首先要实现一个散列函数:

```javascript
loseHashCode(key) {
  if(typeof key === 'number') { // 检验key是否是数
    return key
  }
  const tableKey = this.toStrFn(key) // 将key转换为一个字符串
  let hash = 0;
  for(let i = 0; i < tableKey.length; i++) {
    hash += tableKey.charCodeAt(i); // 将ASCII表中查到的每个字符对应的ASCII值加到hash变量中
  }
  return hash % 37 // 使用hash值和一个任意数做除法的余数，规避操作数超过数值变量最大表示范围的风险
}

hashCode(key) {
  return this.loseHashCode(key)
}
```

### 2.2. 将键和值加入散列表

```javascript
put(key, value) {
  if(key != null && value != null) { // key和value是否合法
    const position = this.hashCode(key) // 在表中找到一个位置
    this.table[position] = new ValuePair(key, value)
    return true
  }
  return false
}
```

### 2.3 从散列表中获取一个值

```javascript
get(key) {
  const valuePair = this.table[this.hashCode(key)]
  return valuePair == null ? undefined : valuePair.value
}
```

### 2.4 从散列表中移除一个值

```javascript
remove(key) {
  const hash = this.hashCode(key)
  const valuePair = this.table[hash]
  if(valuePair != null) {
    delete this.table[hash]
    return true
  }
  return false
}
```

### 3. 处理散列表中的冲突

有时一些键会有相同的散列值，不同的值在散列表中对应相同位置时，称其为冲突。
使用一个数据结构来保存数据的目的显然不是丢失这些数据，而是通过某种方法将它们保存起来，处理冲突有几种方法：分离链接，线性探查。

### 3.1. 分离链接

分离链接包括为散列表的每一个位置创建一个链表并将元素存储在里面。它是解决冲突的最简单的方法，但在HashTable实例外还需要额外的存储空间

```javascript
class HashTableSeparateChaining {
  constructor(toStrFn = defaultToString) {
    this.toStrFn = toStrFn
    this.table = {}
  }
  // loseHashCode(key) {}
  // hashCode(key) {}
  
  put(key, value) {
    if(key != null && value != null) {
      const position = this.hashCode(key)
      if(this.table[position] == null) { // 要加入新元素的位置是否已经被占据
        this.table[position] = new LinkedList()
      }
      this.table[position].push(new ValuePair(key, value))
      return true
    }
    return false
  }
  
  get(key) { // get方法:用来获取给定键的值。
    const position = this.hashCode(key)
    const linkedList = this.table[position] // 检索linkedList
    if(linkedList != null && !linkedList.isEmpty()) { // 是否存在linkedList实例
      let current = linkedList.getHead() // 迭代前先要获取链表表投的引用
      while(current != null) { // 从链表的头部迭代到尾部
        if(current.element.key === key) { // 是否为要找的键
          return current.element.value // key相同返回node值
        }
        current = current.next
      }
    }
    return undefined
  }
  
  remove(key) { // remove方法，需要从链表中移除一个元素
    const position = this.hashCode(key)
    const linkedList = this.table[position]
    if(linkedList != null && !linkedList.isEmpty()) {
      let current = linkedList.getHead()
      while(current != null) {
        if(current.element.key === key) { // 如果链表中的current元素就是要找的元素
          linkedList.remove(current.element) // 从链表中移除然后验证
          if(linkedList.isEmpty()) { // 如果链表为空
            delete this.table[position] // 将散列表的该位置删除
          }
          return true
        }
        current = current.next
      }
    }
    return false
  }
  
}
```


### 3.2 线性探查

之所以叫线性探查，是因为它处理冲突的方法是将元素直接存储到表中，而不是单独的数据结构中。

当想向表中某个位置添加一个新元素时，如果索引为position的位置已经被占据了，就尝试position + 1的位置，以此类推，直到在散列表中找到一个空闲的位置。

该技术分为两种，一种是软删除方法。使用一个特殊值来表示键值对被删除了(惰性或软)，而不是真正删除，经过一段时间，散列表被操作后，会得到一个标记了若干删除位置的散列表。

```javascript

class ValuePairLazy {
  constructor(key, value, isDeleted = false) {
    super(key, value)
    this.key = key
    this.value = value
    this.isDeleted = isDeleted
  }
}

class HashTableLinearProbingLazy {
  constructor(toStrFn = defaultToString) {
    this.toStrFn = toStrFn
    this.table = {}
  }
  loseHashCode(key) {
    if (typeof key === 'number') {
      return key
    }
    const tableKey = this.toStrFn(key)
    let hash = 0
    for (let i = 0; i < tableKey.length; i++) {
      hash += tableKey.charCodeAt(i)
    }
    return hash % 37
  }
  hashCode(key) {
    return this.loseHashCode(key)
  }
  put(key, value) {
    if(key != null && value != null) {
      const position = this.hashCode(key)
      if(
        this.table[position] == null ||
        (this.table[position] != null && this.table[position].isDeleted)
      ) {
        this.table[position] = new ValuePairLazy(key, value)
      } else {
        let index = position + 1
        while(
          this.table[index] != null &&
          !this.table[position].isDeleted
        ) {
          index++
        }
        this.table[index] = new ValuePairLazy(key, value)
      }
      return true
    }
    return false
  }
  get(key) {
    const position = this.hashCode(key)
    if(this.table[position] != null) {
      if(
        this.table[position].key === key &&
        !this.table[position].isDeleted
      ) {
        return this.table[position].value
      }
      let index = position + 1
      while(
        this.table[index] != null &&
        (this.table[index].key !== key || this.table[index].isDeleted
      ) {
        if(this.table[index].key === key && this.table[index].isDeleted) {
          return undefined
        }
        index++
      }
      if(
        this.table[index] != null &&
        this.table[index].key === key &&
        !this.table[index].isDeleted
      ) {
        return this.table[position].value
      }
    }
    return undefined
  }
  remove(key) {
    const position = this.hashCode(key)
    if(this.table[position] != null) {
      if(
        this.table[position].key === key &&
        !this.table[position].isDeleted
      ){
        this.table[position].isDeleted = true
        return true
      }
      let index = position + 1
      while(
        this.table[index] != null &&
        (this.table[index].key !== key || this.table[index].isDeleted)
      ) {
        index++
      }
      if(
        this.table[index] != null &&
        this.table[index].key === key &&
        !this.table[index].isDeleted
      ) {
        this.table[index].isDeleted = true
        return true
      }
    }
    return false
  }
  isEmpty() {
    return this.size() === 0
  }
  size() {
    let count = 0
    Object.values(this.table).forEach(valuePair => {
      count += valuePair.isDeleted === true ? 0 : 1
    })
    return count
  }
}
```

第二种方法需要验证是否有必要将一个或多个元素移动到之前的位置。当搜索一个键时，这种方法可以避免找到一个空位置，如果移动元素是必要的，就需要在散列表中移动键值对。

```javascript
class ValuePair {
  constructor(key, value) {
    this.key = key
    this.value = value
  }
  toString(){
    return `[#${this.key}: ${this.value}]`
  }
}

class HashTableLinearProbing {
  constructor(toStrFn = defaultToString) {
    this.toStrFn = toStrFn
    this.table = {}
  }
  // loseHashCode(key) {}
  // hashCode(key) {}
 
  // 实现put方法
  put(key, value) {
    if(key != null && value != null) {
      const position = this.hashCode(key)
      if(this.table[position] == null) { // 1
        this.table[position] = new ValuePair(key, value) // 2
      }
    }
  }
  
  // get方法
  get(key) {
    const position = this.hashCode(key);
    if (this.table[position] != null) { // 确认键的存在
      if (this.table[position].key === key) { // 检查要找的值是否就是原始位置上的值
        return this.table[position].value; // 是就返回该值
      }
      let index = position + 1; // 若不是就在下一个位置继续查找
      while (
        this.table[index] != null &&
        this.table[index].key !== key
      ) { // 按位置递增的顺序查找散列表上的元素直到找到要找的元素或找到一个空位置
        index++;
      }
      if (
        this.table[index] != null &&
        this.table[index].key === key
      ) { // 验证元素的键是否是要找的键
        return this.table[position].value; // 返回值
      }
    }
    return undefined;
  }
  
  remove(key) {
    const position = this.hashCode(key);
    if (this.table[position] != null) {
      if (this.table[position].key === key) {
        delete this.table[position]; // 从散列表中删除元素，可以直接从原始hash位置找到元素
        this.verifyRemoveSideEffect(key, position); // 将冲突的元素移动至一个之前的位置
        return true;
      }
      let index = position + 1;
      while (
        this.table[index] != null &&
        this.table[index].key !== key
      ) {
        index++;
      }
      if (
        this.table[index] != null &&
        this.table[index].key === key
      ) {
        delete this.table[index]; // 若有冲突并被处理了可以在另一个位置找到元素
        this.verifyRemoveSideEffect(key, index); // 将冲突的元素移动至一个之前的位置
        return true;
      }
    }
    return false;
  }
  
  // 冲突元素转移
  verifyRemoveSideEffect(key, removePosition) {
    const hash = this.hashCode(key) // 获取被删除的key的hash值
    let index = removedPosition + 1 // 下一个位置迭代散列表
    while(this.table[index] != null) { // 直到找到一个空位置
      const posHash = this.hashCode(this.table[index].key) // 计算当前位置上元素的hash值
      if(posHash <= hash || posHash <= removedPosition) { // 当前元素的hash <= 原始的hash值
        this.table[removedPosition] = this.table[index] // 需要将当前元素移动至removedPosition的位置
        delete this.table[index]
        removedPosition = index
      }
      index++
    }
  }
}
```

### 4. 优化散列函数

 lose散列函数并不是一个良好的散列函数，因为会产生太多的冲突，一个良好的散列函数是由几个方面构成：
 
*  插入和检索元素的时间(性能)
*  较低冲突的可能性

```javascript
betterHashCode(key) {
  const tableKey = this.toStrFn(key) // 将键转为字符串
  let hash = 5381 // 通用质数
  for(let i = 0; i < tableKey.length; i++) { // 迭代参数key
    hash = (hash * 33) + tableKey.charCodeAt(i) // 和当前迭代到的字符的ASCII码值相加
  }
  return hash % 1013 // 使用相加的和与另一个随机质数相除的余数
}
```