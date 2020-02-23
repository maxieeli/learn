# JavaScript原型

### 构造函数创建一个对象

首先，用一个构造函数创建一个对象：

```
function Person() {}
var person = new Person();
person.name = 'javascript';
console.log(person.name); // javascript
```

上述的代码，Person就是一个构造函数，利用 new 创建了一个实例对象person



### prototype

每个函数都有一个prototype属性，例如：

```
function Person () {}
Person.prototype.name = 'javascript';
const person1 = new Person();
const person2 = new Person();
console.log(person1.name); // javascript;
console.log(person2.name); // javascript;
```

函数的 prototype 属性指向了一个对象， 这个对象就是调用该构造函数而创建的实例的原型，也就是 person1 和 person2 的原型。

也可以理解成：每一个JavaScript对象(null除外),在创建的时候,就会与之关联另一个对象,这个对象就指的是原型,每一个对象都会从 原型 当中继承属性。

用一张图表示构造函数和实例原型之间的关系：



![prototype1.png](https://i.loli.net/2020/02/23/cAQPDs9pHOdhauv.png)



在这张图中用 Object.prototype 表示实例原型。
上述说道的是 构造函数 和 实例原型之间的关系，那么实例 与实例原型之间的关系，也就是 person 与 Person.prototype 之间的关系， 则是由 `__proto__` 属性关联。



### `__proto__`

这是每一个 JavaScript 对象(null除外)都具有的一个属性，叫 `__proto__`，该属性会指向该对象的原型。比如以下代码所示：

```
function Person() {}
const person = new Person();
console.log(person.__proto__ === Person.prototype); // true
```

更新上述图例中的关系：



![prototype2.png](https://i.loli.net/2020/02/23/xfm9PMa8N1QAKYj.png)



既然实例对象和构造函数都可以指向原型，那么原型是否有属性指向构造函数或者实例呢？



### Constructor

构造函数指向实例倒是没有，因为一个构造函数可以生成多个实例，但是原型指向构造函数倒是有的，也就是 constructor，每个原型都有一个 constructor 属性指向关联的构造函数。

```
function Person () {}
console.log(Person === Person.prototype.constructor); // true
```

现在再次更新上述图例的关系：



![prototype3.png](https://i.loli.net/2020/02/23/f4UrX2WYGcxHNeR.png)



综上所述，可以得出以下几个结论：

```
function Person() {}
const person = new Person();
console.log(person.__proto__ === Person.prototype); // true
console.log(Person.prototype.constructor === Person); // true
// es6获取对象原型的方法
console.log(Object.getPrototypeOf(person) === Person.prototype); // true
```

以上就是构造函数、实例原型、和实例之间的关系，接下来描述下实例和原型之间的关系：



### 实例和原型

当读取实例的属性时，如果找不到属性，就会查找与对象关联的原型中的属性，如果也没有查找到，就会去原型的原型查找，一直找到最顶层为止。

例子：

```
function Person() {}
Person.prototype.name = 'javascript';

const person = new Person();
person.name = 'ecmascript';
console.log(person.name); // ecmascript

delete person.name;
console.log(person.name); // javascript
```

给实例对象person添加了name属性，当输出person.name的时候，结果为emascript。当删除了 person的name属性时，读取 person.name，从person对象中找不到name属性就会从person的原型也就是 `person.__proto__` ， 也就是 Person.prototype 中查找，结果找到了 name 属性，也就是 `javascript`

如果还是没找到该属性，那么则往上找 原型的原型



### 原型的原型

上面描述到，原型也是一个对象，既然是对象，那么就可以用原始的方式创建出来，即：

```
var obj = new Object();
obj.name = 'javascript';
console.log(obj.name); // javascript
```

其实原型对象就是通过Object构造函数生成的，结合之前所描述，实例的 `__proto__` 指向构造函数的prototype， 根据得出的结论，再次更新关系：



![prototype4.png](https://i.loli.net/2020/02/23/zuhZlX4EApNPdkq.png)



### 原型链

那 Object.prototype的原型是什么？可以输出：

```
console.log(Object.prototype.__proto__ === null); // true
```

其实 `Object.prototype.__proto__`的值为null 跟 Object.prototype没有原型，是同一个意思。
所以查找属性的时候查到 Object.prototype就可以停止查找了。

更新关于顶层Object的关系：



![prototype5.png](https://i.loli.net/2020/02/23/PSHdN1levbYLZxF.png)



### 补充

##### constructor

先看个例子：

```
function Person() {}
const person = new Person();
console.log(person.constructor === Person); // true
```

当获取person.constructor时，其实person中没有constructor属性，当不能读取到 constructor 属性时，会从 person 的原型也就是 Person.prototype中读取，正好原型中有该属性，所以：

```
person.constructor === Person.prototype.constructor
```



##### `__proto__`

其次是 `__proto__`， 绝大部分浏览器都支持这个非标准的方法访问原型，然而它并不存在于Person.prototype中，实际上，它来自于Object.prototype，与其说是一个属性，不如说是一个 getter/setter，当使用 `obj.__proto__`时，可以理解成返回了 Object.getPrototypeOf(obj)  -- 该段引用来自 [冴羽的博客](https://github.com/mqyqingfeng/Blog/issues/2) 当中所说

