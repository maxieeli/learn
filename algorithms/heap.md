# 二叉堆和堆排序

## 定义

二叉堆事一种特殊的二叉树，也就是堆数据结构。它能高效快速的找出最大值和最小值。常用于优先队列。它有以下两个特性：

* 它是一颗完全的二叉树，表示树的每一层都有左侧和右侧子节点(除最后一层的叶节点)，并最后一层的叶节点都可能都是左侧子节点(结构特性)
* 二叉堆不是最小堆就是最大堆，最小堆允许快速导出树的最小值，最大堆允许快速导出树的最大值。所有的节点都(>= 或 <=)每个它的节点，称为堆特性。

尽管二叉堆事二叉树，但并不一定是二叉搜索树，在二叉堆中，每个子节点都要 >= 父节点(最小堆) 或 <= 父节点(最大堆). 而在二叉搜索树中，左侧子节点总是比父节点小，右侧子节点也总是更大。

```javascript
function defaultCompare(a, b) {
  if (a === b) {
    return Compare.EQUALS;
  }
  return a < b ? Compare.LESS_THAN : Compare.BIGGER_THAN;
}
function swap(array, a, b) {
  [array[a], array[b]] = [array[b], array[a]];
}
function reverseCompare(compareFn) {
  return (a, b) => compareFn(b, a);
}

// 最小堆类
class MinHeap {
  constructor(compareFn = defaultCompare) {
    this.compareFn = compareFn
    this.heap = []
  }
  getLeftIndex(index) {
    return (2 * index) + 1
  }
  getRightIndex(index) {
    return (2 * index) + 2
  }
  getParentIndex(index) {
    if(index === 0) {
      return undefined
    }
    return Math.floor((index - 1) / 2)
  }
  size() {
    return this.heap.length
  }
  isEmpty() {
    return this.size() <= 0
  }
  
  findMinimum() {
    return this.isEmpty() ? undefined : this.heap[0] // 如果堆不为空，返回数组的第一个值
  }
  
  insert(value) {
    if(value != null) {
      const index = this.heap.length
      this.heap.push(value)
      this.siftUp(index)
      return true
    }
    return false
  }
  
  siftDown(index) {
    let element = index
    const left = this.getLeftIndex(index) // 获取左侧子节点
    const right = this.getRightIndex(index) // 获取右侧子节点
    const size = this.size()
    if(
      left < size &&
      this.compareFn(this.heap[element], this.heap[left]) === Compare.BIGGER_THAN
    ) { // 若元素比左侧子节点要小且index合法
      element = left // 和它的左侧子节点交换元素
    }
    if(
      right < size &&
      this.compareFn(this.heap[element], this.heap[right]) === Compare.BIGGER_THAN
    ) { // // 若元素比右侧子节点要小且index合法
      element = right // 和它的右侧子节点交换元素
    }
    if(index != element) { // 在找到最小子节点的位置后，检验值和element是否相同
      swap(this.heap, index, element) // 将该值和最小的element交换
      this.siftDown(element) // 重复过程直到element被放到正确的位置上
    }
  }
  
  siftUp(index) {
    let parent = this.getParentIndex(index) // 获取其父节点的位置
    while(
      index > 0 &&
      this.compareFn(this.heap[parent], this.heap[index]) === Compare.BIGGER_THAN
    ) { // 如果插入的值小于它的父节点
      swap(this.heap, parent, index) // 将该元素和父节点交换
      index = parent
      parent = this.getParentIndex(index) // 直到堆的根节点也经过了交换节点和父节点位置的操作
    }
  }
  
  extract() {
    if(this.isEmpty()) {
      return undefined // 若堆为空，没有值可以导出，返回undefined
    }
    if(this.size() === 1) {
      return this.heap.shift() // 堆中只有一个值，可以直接移除并返回它
    }
    const removedValue = this.heap[0] // 若堆中不止一个值，需要将第一个值移除
    this.heap[0] = this.heap.pop() // 存储到一个临时变量在
    this.siftDown(0) // 执行完下移操作后
    return removedValue // 返回
  }
  
  heapify(array) {
    if(array) {
      this.heap = array
    }
    const maxIndex = Math.floor(this.size() / 2) - 1)
    for(let i = 0; i <= maxIndex; i++) {
      this.siftDown(i)
    }
    return this.heap
  }
  
  getAsArray() {
    return this.heap
  }
}

// 最大堆类 - 实现跟最小堆类类似，不同在于要把所有 > 换成 <
class MaxHeap extends MinHeap {
  constructor(compareFn = defaultCompare) {
    super(compareFn)
    this.compareFn = reverseCompare(compareFn)
  }
}
```

### 2. 堆排序算法

可以使用二叉堆数据结构帮助创建一个非常著名的排序算法：堆排序算法。包含三个步骤：

* 用数组创建一个最大堆用作源数据
* 在创建最大堆后，最大的值会被存储在堆的第一个位置，要将它替换为堆的最后一个值，将堆的大小减1
* 将堆的根节点下移兵重复上一步骤直到堆的大小为1

```javascript
function heapSort(array, compareFn = defaultCompare) {
  let heapSize = array.length
  buildMaxHeap(array, compareFn) // 步骤1
  while(heapSize > 1) {
    swap(array, 0, --heapSize) // 步骤2
    heapify(array, 0, heapSize, compareFn); // 步骤3
  }
  return array;
}

// 构建最大堆
function buildMaxHeap(array, compareFn) {
  for(let i = Math.floor(array.length / 2); i >= 0; i-=1) {
    heapify(array, i, array.length, compareFn)
  }
  return array
}
```

最大堆函数会重新组织数组的顺序，进行的所有比较，只需要堆后半部分数组执行heapify下移函数(前半部分会被自动排序好，所以不需要对已经排好序的部分执行函数)

堆排序算法不是一个稳定的排序算法，缺点在于如果数组没有排好序，可能有不一样的结果。