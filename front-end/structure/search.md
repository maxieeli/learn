# 搜索算法

## 定义

在组织信息中(存储数据结构)，搜索算法广泛地运用在待解决的日常问题中。接下来会尝试运用下 顺序搜索 / 内插搜索 / 二分搜索 / 随机 算法。

```javascript
// 基础使用方法

const Compare = {
  LESS_THAN: -1,
  BIGGER_THAN: 1,
  EQUALS: 0
}

function defaultEquals(a, b) { // 比较对象或值是否相等
  return a === b
}

function defaultCompare(a, b) {
  if (a === b) {
    return Compare.EQUALS
  }
  return a < b ? Compare.LESS_THAN : Compare.BIGGER_THAN
}

function defaultDiff(a, b) {
  return Number(a) - Number(b)
}
```

### 1.顺序搜索

顺序或线性搜索是最基本的搜索算法。它的机制是将每一个数据结构中的元素和要找的元素做比较，顺序搜索是一种低效的搜索算法。

```javascript
const DOES_NOT_EXIST = -1
function sequentialSearch(array, value, equalsFn = defaultEquals) {
  for(let i = 0; i < array.length; i++) {
    if(equalsFn(value, array[i])) {
      return i
    }
  }
  return DOES_NOT_EXIST
}
```

### 2. 二分搜索

该算法要求被搜索的数据结构已排序。 步骤是：

* 选择数组中间值
* 如果选中值是待搜索值，执行完毕
* 如果待搜索值比选中值小，返回步骤1并在选中值左边的子数组中寻找
* 如果待搜索值比选中值小，返回步骤1并在选中值右边的子数组中寻找

```javascript
function binarySearch(array, value, compareFn = defaultCompare) {
  const sortedArray = quickSort(array)
  let low = 0
  let high = sortedArray.length - 1
  while(lesserOrEquals(low, high, compareFn) { // 得到中间项索引并取得中间值
    const mid = Math.floor((low + high) / 2)
    const element = sortedArray[mid]
    if(compareFn(element, value) === Compare.LESS_THAN) { //比较中项值和搜索值
      low = mid + 1
    } else if(compareFn(element, value) === Compare.BIGGER_THAN) {
      high = mid - 1
    } else {
      return mid
    }
  }
  return DOES_NOT_EXIST
}

function lesserOrEquals(a, b, compareFn) {
  const comp = compareFn(a, b)
  return comp === Compare.LESS_THAN || comp === Compare.EQUALS
}
```

### 3. 内插搜索

内插搜索是改良版的二分搜索，二分搜索总是检查mid位置上的值，而内插搜索可能会根据要搜索的值检查数组中的不同地方。

该算法要求被搜索的数据结构已排序，有以下的步骤：

* 使用position公式选中一个值
* 如果这个值是待搜索值，执行完毕
* 如果待搜索值比选中值小，返回第一步骤并在选中值左边的子数组中找(较小)
* 如果待搜索值比选中值大，返回第一步骤并在选中值右边的子数组中找(较大)

```javascript
function interpolationSearch(
  array, value,
  compareFn = defaultCompare,
  equalsFn = defaultEquals,
  diffFn = defaultDiff
) {
  const {length} = array
  let low = 0
  let high = length - 1
  let position = -1
  let delta = -1
  while(
    low <= high &&
    biggerEquals(value, array[low], compareFn) &&
    lesserEquals(value, array[high], compareFn)
  ) {
    delta = diffFn(value, array[low]) / diffFn(array[high], array[low]) // 该算法在数组中的值都是均匀分布时性能最好(delta值最小)
    position = low + Math.floor((high - low) * delta) // 计算要比较值的位置
    if(equalsFn(array[position], value)) { // 找到搜索值并返回
      return position
    }
    if(compareFn(array[position], value) === Compare.LESS_THAN) { // 待搜索值小于当前位置的值，使用左边或右边的子数组重复执行
      low = position + 1
    } else {
      high = position - 1
    }
  }
  return DOES_NOT_EXIST
}

function lesserOrEquals(a, b, compareFn) {
  const comp = compareFn(a, b)
  return comp === Compare.LESS_THAN || comp === Compare.EQUALS
}

function biggerOrEquals(a, b, compareFn) {
  const comp = compareFn(a, b)
  return comp === Compare.BIGGER_THAN || comp === Compare.BIGGER_THAN
}
```