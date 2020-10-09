# 集合

## 定义

这是一种不允许值重复的顺序数据结构。集合是由一组无序且唯一的项组成的。该数据结构使用了与有限集合相同的数学概念。但应用在计算机的数据结构中。

### 1.1. 创建集合类

```javascript
class Set {
  constructor() {
    this.items = {}
  }
}
```

### 1.2. has(element)方法

该方法用来检验某个元素是否存在与集合中。

```javascript
has(element) {
  return element in items
}

has(element) {
  return Object.prototype.hasOwnProperty.call(this.items, element)
}
```

### 1.3. add/delete方法

```javascript
add(element) {
  if(!this.has(element)){ // 检查是否存在集合中
    this.items[element] = element
    return true
  }
  return false
}

delete(element) {
  if(this.has(element)){ // 验证给定element是否存在于集合中
    delete this.items[element]
    return true
  }
  return false
}
```

### 1.4. size方法

有三种实现方式：

* 使用一个length变量
* Javascript中Object类的一个内置方法
* 手动提取items对象的每一个属性，记录属性的个数并返回

```javascript
size() {
  return Object.keys(this.items).length
}

sizeLegacy() {
  let count = 0
  for(let key in this.items) {
    if(this.items.hasOwnProperty(key)) {
      count++
    }
  }
  return count
}
```

### 1.5. values方法

```javascript
values() {
  return Object.values(this.items)
}

valuesLegacy() {
  let values = []
  for(let key in this.items) {
    if(this.items.hasOwnProperty(key)) {
      values.push(key)
    }
  }
  return values
}
```

### 2. 并集

```javascript
union(otherSet) {
  const unionSet = new Set() // 代表两个集合的并集
  this.values().forEach(value => unionSet.add(value)) // 迭代并全部添加到代表并集的集合中
  otherSet.values().forEach(value => unionSet.add(value)) // 同上
  return unionSet
}
```

### 3. 交集

```javascript
intersection(otherSet) {
  const intersectionSet = new Set()
  const values = this.values()
  for(let i = 0; i < values.length; i++) {
    if(otherSet.has(values[i])) { // 验证是否存在otherSet实例之中
      intersectionSet.add(values[i])
    }
  }
  return intersectionSet
}
```

### 4. 差集

```javascript
difference(otherSet) {
  const differenceSet = new Set()
  this.values().forEach(value => {
    if(!otherSet.has(value)) { // 检查当前值是否存在于给定集合中
      differenceSet.add(value);
    }
  })
  return differenceSet
}
```

### 5. 子集

```javascript
isSubsetOf(otherSet) {
  if(this.size() > otherSet.size()) {
    return false
  }
  let isSubset = true // 如果当前实例是给定集合的子集
  this.values().every(value => {
    if(!otherSet.has(value)) { // 验证元素也存在于otherSet中
      isSubset = false // 如果有任何元素不存在于otherSet中，就意味着它不是一个子集
      return false
    }
    return true
  })
  return isSubset
}
```