# JavaScript中对数组的操作

数组是 JS 中广泛使用的数据结构，数组对象提供了大量的方法。以下列出数组中对各方法的使用方法

### 数组的遍历

+ `for...of...`循环
    + 循环遍历数组项
    + 可随时使用break停止遍历

```
const strArr = ['one', 'two', 'three'];
for(const str of strArr) {
  console.log(str); // 'one', 'two', 'three'
}
```

+ `for`循环
    + 使用递增的索引变量遍历数组项,通常需要在每个循环中递增 i 变量
    + 可随时使用break语句停止遍历

```
const strArr = ['one', 'two', 'three'];
for(let i = 0; i < strArr.length; i++) {
  console.log(strArr[i]); // 'one', 'two', 'three'
}
```

+ `array.forEach()`
    + 通过在每个数组项上调用 回调函数 来遍历数组项
    + 不能中断forEach() 迭代

```
const strArr = ['one', 'two', 'three'];
strArr.forEach(function(value, index) {
  console.log(value, index);
});
// 'one', 1
// 'two', 2
// 'three', 3
```



### 数组的映射

+ `Array.map()`方法
    + 方法通过在每个数组项上使用  callback 调用结果来创建一个新数组
    + 创建一个新的映射数组，不会改变原数组

```
const numArr = [1, 2, 3];
const newNum = numArr.map(function(n) {
  return n + 1;
});
console.log(newNum); // [2, 3, 4]
```

+ `Array.from()`方法
    + 通过在每个数组项上使用 callback 调用结果创建一个新数组
    + 创建一个新的映射数组，不会改变原数组
    + 适合从类似数组的对象进行映射

```
const numArr = [1, 2, 3];
const newNum = Array.from(numArr, function(n){
  return n + 1;
});
console.log(newNum); // => [2, 3, 4]
```



### 数据的简化

+ `Array.reduce()`方法
    + 通过调用 callback 函数来将数组简化为一个值
    + 每次遍历中的 `callback(accumalator, item[, index[, array]])`使用用参数调用的累加器，当前项，索引，数组本身且应该返回累加器
    + 如果没有设置初始值，默认采用数组的第一个元素作为初始值

```
const numArr = [3, 5, 8];
function summarize(accumaltor, number) {
  return accumalator + number;
}
const sum = numArr.reduce(summarize, 0);
console.log(sum); // 16
```



### 数据的连接

+ `Array.concat()`方法
    + 将一个或多个数组连接到原始数组
    + 创建一个新数组，不改变原始数组

```
const firstArr = ['one', 'two'];
const secondArr = ['three', 'four'];
const totalArr = firstArr.concat(secondArr);
console.log(totalArr); // ['one', 'two', 'three', 'four']
```

+ 展开运算符

```
const firstArr = ['one', 'two'];
const secondArr = ['three', 'four'];
const totalArr = [...firstArr, ...secondArr];
console.log(totalArr); // ['one', 'two', 'three', 'four']
```



### 数组片段

+ `Array.slice()`方法
    + 返回数组的一个片段，从截取位置开始到截止位置结束
    + 创建一个新数组，不改变原数组

```
const numArr = [1, 2, 3, 4];
const sliceArr = numArr.slice(0, 3);
const spliceArr = numArr.splice(2);
console.log(sliceArr); // [1, 2, 3]
console.log(spliceArr); // [3, 4]
```



### 数组拷贝

+ 展开运算符
    + 创建一个浅拷贝

```
const numArr = [1, 2, 3];
const cloneArr = [...numArr];
console.log(cloneArr); // [1, 2, 3]
console.log(numArr === cloneArr); // false
```

+ `Array.concat()`
    + 创建一个浅拷贝

```
const numArr = [1, 2, 3];
const cloneArr = [].concat(numArr);
console.log(cloneArr); // [1, 2, 3]

```

+ `Array.slice()`
    + 创建一个浅拷贝

```
const numArr = [1, 2, 3];
const cloneArr = numArr.slice();
console.log(cloneArr); // [1, 2, 3]

```



### 数组查找

+ `Array.includes()`
    + 默认值是0， 返回布尔值

```
const numArr = [1, 2, 3, 4, 5];
numArr.includes(2); // true
numArr.includes(30); // false
```

+ `Array.find()`
    + 返回数组中满足提供的函数的第一个元素的值，否则返回undefined

```
const numArr = [1, 2, 3, 4, 5];
const resultArr = numArr.find(arr => {
  return arr % 2 === 0
});
console.log(resultArr); // 2
```

+ `Array.indexOf()`
    + 返回数组中第一个出现索引，默认值为0

```
const numArr = [1, 2, 3, 4, 5];
const resultArr = numArr.indexOf(2);
console.log(resultArr); // 1
```



### 数组查询

+ `Array.every()`
    + 如果数组的每一项都符合条件则返回true

```
const firstArr = [0, 2, 4,6];
const secondArr = [1, 2, 4, 6];
function isEvery(num) {
  return num % 2 === 0;
}
firstArr.every(isEvery); // true
secondArr.every(isEvery); // false
```

+ `Array.some()`
    + 如果数组至少有一项符合条件则返回true

```
const firstArr = [1, 2, 7, 9];
const secondArr = [1, 7, 9, 11];
function isSome(num) {
  return num % 2 === 0;
}
firstArr.some(isSome); // true
secondArr.some(isSome); // false
```



### 数组过滤

+ `Array.filter()`方法
    + 创建一个新数组(不改变原数组)，其包含通过所提供的条件的所有元素

```
const numArr = [1, 2, 7, 9];
const resultArr = numArr.filter(i => i > 3);
console.log(resultArr); // [7, 9]
```



### 数组插入

+ `Array.push()`
    + 将一个或多个项追加到数组的末尾，并返回新的长度，会改变原数组

```
const strArr = ['one'];
strArr.push('two');
console.log(strArr); // ['one', 'two']
```

+ `Array.unshift()`
    + 将一个或多个项追加到数组的开头，返回数组的新长度，会改变原数组


```
const strArr = ['one'];
strArr.unshift('two');
console.log(strArr); // ['two', 'one']
```

+ 展开操作符
    + 可以通过组合展开操作符和数字字面量以不可变的方式在数组中插入项

```
const strArr = ['one', 'two'];
const resultArr = [...strArr, 'three'];
console.log(resultArr); // ['one', 'two', 'three']
```



### 数组删除

+ `Array.pop()`
    + 从数组中删除最后一个元素，返回该元素，会改变原数组

```
const strArr = ['one', 'two'];
const lastArr = strArr.pop();
console.log(lastArr); // 'two'
console.log(strArr); // ['one']
```

+ `Array.shift()`
    + 从数组中删除第一个元素，然后返回该元素，会改变原数组

```
const strArr = ['one', 'two'];
const lastArr = strArr.shift();
console.log(lastArr); // 'one'
console.log(strArr); // ['two']

```

+ `Array.splice()`
    + 从数组中删除指定位置，会改变原数组

```
const strArr = ['one', 'two', 'three', 'four'];
strArr.splice(1, 2);
console.log(strArr); // ['one', 'four']
```

+ 展开运算符
    + 可以通过组合展开操作符和数据字面量以不可变的方式从数组中删除项

```
const strArr = ['one', 'two', 'three', 'four'];
const fromIndex = 1;
const removeCount = 2;
const newArr = [
  ...strArr.slice(0, fromIndex),
  ...strArr.slice(fromIndex + removeCount)
]
console.log(newArr); // ['one', 'four']
```



### 数组清空

+ `Array.length`
    + 保存数组长度的属性，同时也是可写的

```
const strArr = ['one', 'two', 'three', 'four'];
strArr.length = 0;
console.log(strArr); // []
```

+ `Array.splice()`
    + 指定位置为0，即可删除数组的所有项

```
const strArr = ['one', 'two', 'three', 'four'];
strArr.splice(0);
console.log(strArr); // []
```




### 数组填充

+ `array.fill()`
    + 可用来初始化特定长度和初始值的数组
    + 会改变原数组

```
const numArr = [1,2,3,4]
numArr.fill(0); // [0,0,0,0]
numArr.fill(); // [undefined,undefined,undefined,undefined]

const length = 3;
const resultArr = Array(length).fill(0);
console.log(resultArr); // [0, 0, 0]
```

+ `Array.from()`
    + 有助于初始化带有对象的特定长度的数组

```
const length = 4;
const emptyObjects = Array.from(Array(length), function() {
  return {};
});
```



### 数组扁平化

+ `Array.flat()`
    + 通过递归扁平属于数组的项知道一定深度来创建新数组
    + 对数组的扁平仅包含数字
    + 创建一个新数组，不会改变原始数组

```
const numArr = [0, [1,3,5], [2,4,6]];
const resultArr = numArr.flat();
console.log(resultArr); // [0,1,3,5,2,4,6]
```



### 数组排序

+ `Array.sort()`
    + 对数组的元素进行排序
    + 会改变原数组

```
const numArr = [3,5,2,6];
numArr.sort(); // [2,3,5,6]
```



### 补充：Array.reduce()扩展应用

+ 定义：对数组中的每个元素执行一个自定义的累计器，将其结果汇总为单个返回值
+ 形式：`array.reduce((t, v, i, a) => {}, initValue)`
+ 参数
    + <b>callback</b>：回调函数(必选)
    + <b>initValue</b>：初始值(可选)

+ 回调函数的参数
    + <b>total(t)</b>：累计器完成计算的返回值(必选)
    + <b>value(v)</b>：当前元素(必选)
    + <b>index(i)</b>：当前元素的索引(可选)
    + <b>array(a)</b>：当前元素所属的数组对象(可选)

#### 高级用法

+ <b>累加累乘</b>

```
function Accmulation(...arr) {
  return arr.reduce((t, v) => t + v, 0);
}
function Multiplication(...arr) {
  return arr.reduce((t, v) => t * v, 0);
}
Accmulation(1,2,3,4,5); // 15
Multiplication(1,2,3,4,5); // 120
```

+ <b>权重求和</b>

```
const scores = {
  {score: 90, sub: 'chinese', weight: 0.5},
  {score: 95, sub: 'math', weight: 0.3},
  {score: 85, sub: 'english', weight: 0.2}
}
const result = scores.reduce((t, v) => t + v.score * v.weight, 0);//[90.5]
```

+ <b>代替reverse</b>

```
function Reverse(arr = []) {
  return arr.reduceRight((t, v) => (t.push(v), t), []);
}
Reverse([1, 2, 3, 4, 5]); // [5, 4, 3, 2, 1]
```

+ <b>代替map和filter</b>

```
const numArr = [0, 1, 2, 3];
// 代替map
const a = arr.map(v => v * 2);
const b = arr.reduce((t, v) => [...t, v * 2], []);

// 代替filter
const a = arr.map(v => v > 1);
const b = arr.reduce((t, v) => v > 1 ? [...t, v] : t, []);

// 代替map和filter -> [4, 6]
const a = arr.map(v => v * 2).filter(v => v > 2);
const b = arr.reduce((t, v) => v * 2 > 2 ? [...t, v * 2] : t, [])
```

+ <b>代替some和every</b>

```
const scores = {
  {score: 50, sub: 'chinese'},
  {score: 65, sub: 'math'},
  {score: 85, sub: 'english'}
};
// 代替some，至少一门合格
const isOneClass = scores.reduce((t, v) => t || v.score >= 60, false)

// 代替every，全部合格
const isAllClass = scores.reduce((t, v) => t && v.scores >= 60, false)
```

+ <b>数组分割</b>

```
function Chunk(arr = [], size = 1) {
  return arr.length ? arr.reduce((t, v) => (t[t.length - 1].length === size ? t.push([v]) : t[t.length - 1].push(v), t), [[]]) : [];
}
const numArr = [1, 2, 3, 4, 5];
Chunk(numArr, 2); // [[1, 2], [3, 4], [5]]
```

+ <b>数组过滤</b>

```
function Diff(nArr = [], oArr = []) {
  return newArr.reduce((t, v) => (!oArr.includes(v) && t.push(v), t), []);
}

const arr1 = [1,2,3,4,5];
const arr2 = [2,3,6];
Diff(arr1, arr2); // [1, 4, 5]
```

+ <b>数组填充</b>

```
function Fill(arr = [], val = '', start = 0, end = arr.length) {
  if (start < 0 || start >= end || end > arr.length) return arr;
    return [
      ...arr.slice(0, start),
      ...arr.slice(start, end).reduce((t, v) => (t.push(val || v), t), []),
      ...arr.slice(end, arr.length)
    ];
}
const numArr = [0,1,2,3,4,5,6];
Fill(arr, 'a', 2, 5); // [0, 1, 'a', 'a', 'a', 5, 6]
```

+ <b>数组扁平</b>

```
function Flat(arr = []) {
  return arr.reduce((t, v) => t.concat(Array.isArray(v) ? Flat(v) : v), [])
}

const numArr = [0, 1, [2, 3], [4, 5, [6, 7]], [8, [9, 10, [11, 12]]]];
Flat(numArr); // [0,1,2,3,4,5,6,7,8,9,10,11,12]
```

+ <b>数组去重</b> 

```
function Uniq(arr = []) {
  return arr.reduce((t, v) => t.includes(v) ? t : [...t, v], []);
}

const numArr = [2, 1, 3, 2, 0, 6, 5, 3];
Uniq(numArr); // [2, 1, 3, 0, 6, 5]
```

+ <b>数组最大最小值</b>

```
function Max(arr = []) {
  return arr.reduce((t, v) => t > v ? t : v);
}
function Min(arr = []) {
  return arr.reduce((t, v) => t < v ? t : v);
}
const numArr = [92, 4, 45, 72, 29, 17];
Max(numArr); // 92
Min(numArr); // 4
```

+ <b>数组项拆解</b>

```
function Unzip(arr = []) {
  return arr.reduce(
    (t, v) => (v.forEach(w, i) => t[i].push(w), t),Array.from({ length: Math.max(...arr.map(v => v.length)) }).map(v => [])
  );
}
const strArr = [["a", 1, true], ["b", 2, false]];
Unzip(strArr); // [["a", "b"], [1, 2], [true, false]]
```

+ <b>数组项个数统计</b>

```
function Count(arr = []) {
  return arr.reduce((t, v) => (t[v] = (t[v] || 0) + 1， t), {});
}
const numArr = [0, 1, 1, 2, 2, 2];
Count(numArr); // { 0: 1, 1: 2, 2: 3 }
```

+ <b>数组项位置记录</b>

```
function Position(arr = [], val) {
  return arr.reduce((t, v, i) => (v === val && t.push(i), t), []);
}
const numArr = [2, 1, 5, 4, 2, 1, 6, 6, 7];
Position(numArr, 2); // [0, 4]
```

+ <b>字符串翻转</b>

```
function ReverseStr(str = '') {
  return str.split('').reduceRight((t, v) => t + v);
}
const str = 'reduce';
ReverseStr(str); // 'ecuder'
```
