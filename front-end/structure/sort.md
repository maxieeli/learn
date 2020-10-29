# 排序算法

## 定义

在组织信息中(存储数据结构)，排序和搜索算法广泛地运用在待解决的日常问题中。接下来会尝试运用下 冒泡排序 / 选择排序 / 希尔排序 / 归并排序 / 快速排序 / 计数排序 / 桶排序 / 基数排序。

### 1. 排序算法

```javascript
// 基础工具

const Compare = {
  LESS_THAN: -1,
  BIGGER_THAN: 1,
  EQUALS: 0
}

function defaultCompare(a, b) {
  if (a === b) {
    return Compare.EQUALS
  }
  return a < b ? Compare.LESS_THAN : Compare.BIGGER_THAN
}

function swap(array, a, b) {
  [array[a], array[b]] = [array[b], array[a]]
}
```

### 1.1. 冒泡排序

```javascript
function bubbleSort(array, compareFn = defaultCompare) {
  const {length} = array; // 1
  for(let i = 0; i < length; i++) { // 2
    for(let j = 0; j < length - 1 - i; j++) {
      if(compareFn(array[j], array[j+1]) === Compare.BIGGER_THAN) {
        swap(array, j, j + 1)
      }
    }
  }
  return array
}
```

### 1.2. 选择排序

选择排序是一种原址比较排序算法，选择排序大致的思路就是找到数据结构中的最小值并将其放置在第一位，接着找到第二小的值并将其放在第二位，以此类推。

```javascript
function selectionSort(array, compareFn = defaultCompare) {
  const {length} = array
  let indexMin
  for(let i = 0; i < length - 1; i++) {
    indexMin = i
    for(let j = i; j < length; j++) {
      if(compareFn(array[indexMin], array[j]) === Compare.BIGGER_THAN){
        indexMin = j
      }
    }
    if(i !== indexMin) {
      swap(array, i, indexMin)
    }
  }
  return array
}
```

### 1.3. 插入排序

插入排序每次排一个数组项，以此构建最后的排序数组。

```javascript
function insertionSort(array, compareFn = defaultCompare) {
  const {length} = array
  let temp
  for(let i = 1; i < length; i++) {
    let j = i
    temp = array[i]
    while(j > 0 && compareFn(array[j - 1], temp) === Compare.BIGGER_THAN) {
      array[j] = array[j - 1]
      j--
    }
    array[j] = temp
  }
  return array
}
```

### 1.4. 归并排序

这是一个可以实际使用的排序算法，思路就是将原始数组切分成较小的数组，直到每个小数组只有一个位置，接着将小数组归并成较大的数组，直到最后只有一个排序完毕的大数组。但归并排序也是递归的。要将算法分为两个函数：第一个负责将大数组分为多个小数组并调用用来排序的辅助函数。

```javascript
function mergeSort(array, compareFn = defaultCompare) {
  if(array.length > 1) { // 判断数组的长度是否为1
    const {length} = array
    const middle = Math.floor(length / 2) // 找到数组中间位
    // 数组分成两个小数组，left数组由索引0至中间索引的元素组成，而right数组由中间件索引至原始数组最后一个位置的元素组成。数组对自身调用mainSort直到数组的大小小于1
    const left = mergeSort(array.slice(0, middle), compareFn)
    const right = mergeSort(array.slice(middle, length), compareFn)
    array = merge(left, right, compareFn)
  }
  return array
}

// 负责合并和排序小数组来产生大数组,直到原始数组并排序完成
function merge(left, right, compareFn) {
  let i = 0
  let j = 0
  const result = []
  while(i < left.length && j < right.length) {
    result.push(
      compareFn(left[i], right[j] === Compare.LESS_THAN ? left[i++] : right[i++])
    )
  }
  return result.concat(i < left.length ? left.slice(i) : right.slice(j))
}
```

### 1.5. 快速排序

快速排序是最常用的排序算法。复杂度O(nlog(n)), 切性能通畅比其他的排序算法要好，和归并排序一样，将原始数组分为较小的数组(但没有归并排序那样分割)

```javascript
function quick(array, left, right, compareFn) {
  return quick(array, 0, array.length - 1, compareFn)
}

function quick(array, left, right, compareFn) {
  let index
  if(array.length > 1) {
    index = partition(array, left, right, compareFn)
    if(left < index - 1) {
      quick(array, left, index-1, compareFn)
    }
    if(index < right) {
      quick(array, index, right, compareFn)
    }
  }
  return array
}

function partition(array, left, right, compareFn) {
  const pivot = array[Math.floor((right + left) / 2)]
  let i = left
  let j = right
  while(i <= j) {
    while(compareFn(array[i], pivot) === Compare.LESS_THAN) {
      i++
    }
    while(compareFn(array[j], pivot) === Compare.BIGGER_THAN) {
      j--
    }
    if(i <= j) {
      swap(array, i, j)
      i++
      j--
    }
  }
  return i
}
```

### 1.6. 计数排序

计数排序使用一个用来存储每个元素在原始数组中出现次数的临时数组，在所有元素都计数完成后，临时数组已排好序并可迭代以构建排序后的结果数组。

```javascript
function countingSort(array) {
  if(array.length < 2) { // 数组为空或只有一个元素
    return array
  }
  const maxValue = findMaxValue(array) // 找数组中最大值
  const counts = new Array(maxValue + 1) // 计数直到最大索引value+1
  array.forEach(element => {
    if(!counts[element]) {
      counts[element] = 0
    }
    counts[element]++
  })
  let sortedIndex = 0
  counts.forEach((count, i) => {
    while(count > 0) {
      array[sortedIndex++] = i
      count--
    }
  })
  return array
}

function findMaxValue(array) {
  let max = array[0]
  for(let i = 1; i < array.length; i++) {
    if(array[i] > max) {
      max = array[i]
    }
  }
  return max
}
```

### 1.7. 桶排序

桶排序也是分布式排序算法，它将元素分为不同的桶(较小的数组)，再使用一个简单的排序算法(插入排序)，来对每个桶进行排序。然后将所有的桶合并为结果数组。

```javascript
function bucketSort(array, bucketSize = 5) { // 指定需要桶来排序各个元素
  if(array.length < 2) {
    return array
  }
  // 创建桶并将元素分布到不同桶中
  const buckets = createBuckets(array, bucketSize)
  // 包含每个桶执行插入排序算法和将所有桶合并为排序后的结果数组
  return sortBuckets(buckets)
}

function createBuckets(array, bucketSize) {
  let minValue = array[0]
  let maxValue = array[0]
  for(let i = 1; i < array.length; i++) {
    if(array[i] < minValue) {
      minValue = array[i]
    } else if(array[i] > maxValue) {
      maxValue = array[i]
    }
  }
  const bucketCount = Math.floor((maxValue - minValue) / bucketSize) + 1 // 计算每个桶需要分布的元素个数
  const buckets = []
  for(let i = 0; i < bucketCount; i++) { // 初始化每个桶
    buckets[i] = []
  }
  for(let i = 0; i < array.length; i++) {
    const bucketIndex = Math.floor((array[i] - minValue) / bucketSize) // 计算将元素放在哪个桶中
    buckets[bucketIndex].push(array[i])
  }
  return buckets
}

function sortBuckets(buckets) {
  const sortedArray = [] // 创建一个用于结果数组的新数组
  for(let i = 0; i < buckets.length; i++) {
    if(buckets[i] != null) {
      insertionSort(buckets[i]) // 迭代每个可迭代桶并插入排序
      sortedArray.push(...buckets[i]) // 排好序的桶中的元素加入到结果数组中
    }
  }
  return sortedArray
}
```

### 1.8. 基数排序

基数排序是一个分布式排序算法，它根据数字的有效位或基数，将整数分布到桶中。基数是基于数组中值的记数制的。比如对于十进制数，使用的基数是10，算法使用10个桶用来分布元素并且首先基于个位数字进行排序，然后基于十位/百位，以此类推。

```javascript
function radixSort(array, radixBase = 10) {
  if(array.length < 2) {
    return array
  }
  const minValue = findMinValue(array)
  const maxValue = findMaxValue(array)
  let significantDigit = 1 // 从最后一位开始排序所有的数,也可以被修改成支持排序字母字符
  while((maxValue - minValue) / significantDigit >= 1) { // 直到没有待排序的有效位
    array = countingSortForRadix(array, radixBase, significantDigit, minValue)
    significantDigit *= radixBase
  }
  return array
}

// 基于有效位(基数)排序
function countingSortForRadix(array, radixBase, significantDigit, minValue) {
  let bucketsIndex
  const buckets = []
  const aux = []
  for(let i = 0; i < radixBase; i++) { // 初始化桶
    buckets[i] = 0
  }
  for(let i = 0; i < array.length; i++) { // 遍历数组中的有效位
    bucketsIndex = Math.floor(
      ((array[i] - minValue) / significantDigit) % radixBase
    ) // 进行计数排序
    buckets[bucketsIndex]++
  }
  for(let i = 1; i < radixBase; i++) { // 需要计算累积结果得到正确的计数值
    buckets[i] += buckets[i - 1]
  }
  for(let i = array.length - 1; i >= 0; i--) {
    bucketsIndex = Math.floor(
      ((array[i] - minValue) / significantDigit) % radixBase
    ) // 再次获取有效位并将值移动到aux数组中
    aux[--buckets[bucketsIndex]] = array[i] //从buckets数组中减去它的计数值
  }
  for(let i = 0; i < array.length; i++) { // 将aux数组中的每个值转回原始数组中，除了返回array之外，还可以直接返回aux数组而不需要复制它的值
    array[i] = aux[i]
  }
  return array
}
```