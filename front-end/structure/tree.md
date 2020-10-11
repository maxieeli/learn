# 树

## 定义

一个树结构包含一系列存在父子关系的节点，每一个节点都有一个父节点(除顶部的第一个节点)以及0或多个字节点。

节点的一个属性是深度，节点的深度取决于它的祖先节点的数量。树的高度取决于所有节点深度的最大值，一棵树可以被分解成多层级。除了根节点，字节点的层数+1，以此类推。

### 二叉树和二叉搜索树

二叉树中的节点最多只能有两个子节点：一个是左侧子节点，一个是右侧子节点。这个定义有助于更高效的在书中插入，查找和删除节点。

二叉搜索树是二叉树的一种，但只允许在左侧节点存储比父节点小的值，在右侧节点存储比父节点大的值

### 1.1 创建树类

实现一些树数据结构需要用到的工具类

```javascript
// 声明一些工具类
const Compare = {
  LESS_THAN: -1,
  BIGGER_THAN: 1,
  EQUALS: 0
}

function defaultCompare(a, b) {
  if(a === b) {
    return Compare.EQUALS
  }
  return a < b ? Compare.LESS_THAN : Compare.BIGGER_THAN
}

// 创建树的节点
class Node {
  constructor(key) {
    this.key = key // 键，也是字典中对应的项
    this.left = null // 左侧子节点引用
    this.right = null // 右侧子节点引用
  }
}
```

声明树类的基本结构

```javascript
class BinarySearchTree {
  constructor(compareFn = defaultCompare) {
    this.compareFn = compareFn // 用来比较节点的值
    this.root = null // Node类型的根节点
  }
  
  // 向树插入一个新键
  insert(key) {
    if(this.root == null) { // 是否为第一个节点
      this.root = new Node(key) // 创建Node类实例并将它赋值给root属性来将root指向新节点
    } else {
      this.insertNode(this.root, key) // 将节点添加到根节点以外的其他位置
    }
  }
  
  // 插入键方法
  insertNode(node, key) {
    if(this.compareFn(key, node.key) === Compare.LESS_THAN) { // 新节点的键小于当前节点的键
      if(node.left == null) { // 如果没有左侧子节点
        node.left = new Node(key) // 插入新节点
      } else {
        this.insertNode(node.left, key) // 有左侧子节点就递归调用
      }
    } else if(node.right == null) { // 若节点的键比当前节点的键大，同时当前节点没有右侧子节点
      node.right = new Node(key) // 插入新的节点
    } else {
      this.insertNode(node.right, key) // 用来和新节点比较的节点将会是右侧子节点
    }
  }
  
  getRoot() {
    return this.root
  }
  
  search(key) {
    return this.searchNode(this.root, key)
  }
  
  searchNode(node, key) {
    if(node == null) { // 2
      return false
    }
    if(this.compareFn(key, node.key) === Compare.LESS_THAN) { // 要找的键比当前的节点小
      return this.searchNode(node.left, key) // 从左侧子节点开始搜索
    }
    if(this.compareFn(key, node.key) === Compare.BIGGER_THAN) { // 要找的键比当前的节点大
      return this.searchNode(node.right, key) // 从右侧子节点开始搜索
    } else {
      return true
    }
  }
  
  // 寻找树的最小键
  min() {
    return this.minNode(this.root)
  }
  
  minNode(node) {
    let current = node
    while(current != null && current.left != null) { // 遍历左侧树直到树的最下层
      current = current.left
    }
    return current
  }
  
  // 寻找树的最大键
  max() {
    return this.maxNode(this.root)
  }
  
  maxNode(node) {
    let current = node
    while(current != null && current.right != null) { // 遍历左侧树直到树的最下层
      current = current.right
    }
    return current
  }
  
  // 中序遍历: 一种以上行顺序访问BST所有节点的便利方式(从最小到最大)
  inOrderTraverse(callback) {
    this.inOrderTraverseNose(this.root, callback)
  }
  
  inOrderTraverseNode(node, callback) {
    if(node != null) { // 检查以参数形式传入的节点是否为null
      this.inOrderTraverseNode(node.left, callback) //递归访问左侧子节点
      callback(node.key) // 对根节点做操作
      this.inOrderTraverseNode(node.right, callback) //递归访问右侧子节点
    }
  }
  
  // 先序遍历: 以优先于后代节点的顺序访问每个节点
  preOrderTraverse(callback) {
    this.preOrderTraverseNode(this.root, callback)
  }
  
  preOrderTraverseNode(node, callback) {
    if(node != null) {
      callback(node.key) // 先访问节点本身
      this.preOrderTraverseNode(node.left, callback) // 访问左侧子节点
    }
  }
  
  // 后序遍历：先访问节点的后代节点，再访问节点本身
  postOrderTraverse(callback){
    this.postOrderTraverseNode(this.root, callback)
  }
  
  postOrderTraverseNode(node, callback) {
    if(node != null) {
      this.postOrderTraverseNode(node.left, callback)
      this.postOrderTraverseNode(node.right, callback)
      callback(node.key)
    }
  }
  
  // 移除一个节点
  remove(key) {
    this.root = this.removeNode(this.root, key)
  }
  
  removeNode(node, key) {
    if(node == null) { // 如果正在检测的节点为null,说明键不存于树中
      return null
    }
    if(this.compareFn(key, node.key) === Compare.LESS_THAN) {
      node.left = this.removeNode(node.left, key)
      return node
    } else if(this.compareFn(key, node.key) === Compare.BIGGER_THAN) { // 如果要找的键比当前节点的值大
      node.right = this.removeNode(node.right, key) // 沿着树的右边找到下一个节点
      return node
    } else {
      if(node.left == null && node.right == null) { // 该节点是一个没有左侧或右侧子节点的叶节点
        node = null // 该节点没有任何子节点但有一个父节点
        return node // 需通过返回null来将对应的父节点指针赋予null值
      }
      if(node.left == null) { // 若该节点没有左侧子节点
       node = node.right // 有右侧子节点就改为对它右侧子节点的引用
       return node // 更新并返回
      } else if(node.right == null) {
        node = node.left
        return node
      }
      const aux = this.minNode(node.right) //找到要移除的节点，需要找到它右边子树中最小的节点
      node.key = aux.key //用它右侧子树中最小节点的键去更新这个节点的值
      node.right = this.removeNode(node.right, aux.key) // 上述操作之后树中就有两个拥有相同键的节点，需把右侧子树的最小节点移除，因为它已经被移至要移除的节点位置
      return node //向它的父节点返回更新后节点的引用
    }
  }
}
```
