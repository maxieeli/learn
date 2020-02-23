# ES6概述

es6是ECMA为JavaScript指定的第六个标准版本。

es6更新的内容主要分为以下几点：
1.	表达式： 声明，结构赋值
2.	内置对象：字符串 / 数值 / 对象 / 数组 / 函数 / 正则 等扩展， Symbol基本数据类型
3.	语句与运算：Module、Iterator
4.	异步编程：Promise、 Generator

### 声明
const： 一般声明常量，因为声明后必须赋值
let：声明变量

<blockquote>作用范围</blockquote>
+ var命令在全局代码中执行
+ let / const 只能在代码块中执行

<blockquote>使用注意</blockquote>
+ 不允许重复声明
+ 未定义就使用会报错，const 、let不存在变量提升
+ 暂时性死区：在代码块内使用 let 声明变量之前，该变量都不可用



### 解构赋值

<strong>字符串解构：</strong> `const [a, b, c, d, e] = "hello"`

<strong>数值解构：</strong> `const {toString: s} = 123`

<strong>布尔解构：</strong> `const {toString: b} = true`

<strong>对象解构：</strong>

+ 形式: `const {x, y} = {x: 1, y: 2}`
+ 默认: `const {x, y = 2} = {x: 1}`
+ 改名: `const {x, y: z} = {x: 1, y: 2}`

<strong>数组解构</strong>

+ 规则: 数据结构具有 Iterator接口 可采用数组形式的解构赋值
+ 形式: `const [x, y] = [1, 2]`
+ 改名: `const [x, y = 2] = [1]`

<strong>函数参数解构</strong>

+ 数组解构: `function Func([x = 0, y = 1]) {}`
+ 对象解构: `function Func({x = 0, y = 1}) {}`



<blockquote>应用</blockquote>
+ 交换变量值： `[x, y] = [y, x]`
+ 返回函数多个值: `const [x, y, z] = Func()`
+ 定义函数参数: `Func([1, 2])`
+ 提取JSON数据: `const {name, version} = packageJson`
+ 定义函数参数默认值: `function Func({x = 1, y = 2} = {}) {}`
+ 遍历map解结构: `for(let [k, v] of Map) {}`
+ 输入模块指定属性和方法: `const {readFile, writeFile} = require('fs')`

<blockquote>注意</blockquote>
+ 匹配模式：只要等号两边的模式相同，左边的变量就会被赋予对应的值
+ 解构赋值规则：只要等号右边的值不是对象或数组，就先将其转成对象
+ 解构默认值生效条件：属性值严格等于undefined
+ 解构遵循匹配模式
+ 解构不成功时变量的值 = undefined
+ undefined 和 null 无法转为对象，因此无法进行解构



### 字符串扩展

+ <strong>Unicode表示</strong>：大括号表示Unicode字符 ( \u{0xXX} )
+ <strong>字符串遍历</strong>：可通过 for...of... 遍历字符串
+ <strong>字符串模板</strong>：可单行、多行可插入变量的字符串
+ <strong>标签模板</strong>：函数参数的特殊调用
+ <strong>String.raw()</strong>：返回把字符串所有变量替换且斜杠进行转义的结果
+ <strong>String.fromCodePoint()</strong>：返回码点对应字符
+ <strong>normalize()</strong>：把字符的不同表示方法统一为同样形式
+ <strong>repeat()</strong>：把字符串重复n次，返回新的字符串
+ <strong>matchAll()</strong>：返回正则表达式在字符串的所有匹配
+ <strong>includes()</strong>：是否存在指定字符串
+ <strong>startsWith()</strong>：是否存在字符串头部指定字符串
+ <strong>endsWith()</strong>：是否存在字符串尾部指定字符串



### 数值扩展
+ <strong>二进制表示法</strong>：0b / 0B 开头表示二进制（0bXX / 0BXX)
+ <strong>八进制表示法</strong>：0o / 0O开头表示二进制（0oXX / 0Oxx)
+ <strong>Number.EPSILION</strong>：数值最小精度
+ <strong>Number.MIN_SAFE_INTEGER</strong>：最小安全数值
+ <strong>Number.MAX_SAFE_INTEGER</strong>：最大安全数值
+ <strong>Number.parseInt()</strong>：返回转换值的整数部分
+ <strong>Number.parseFloat()</strong>： 返回转换值的浮点数部分
+ <strong>Number.isFinite()</strong>：是否为有效数值
+ <strong>Number.isNaN()</strong>：是否为NaN
+ <strong>Number.isInteger()</strong>：是否为整数



### 对象扩展

+ <strong>简洁表示法</strong>：直接写入变量和函数作为对象的属性和方法( {prop, method() {})
+ <strong>属性名表达式</strong>：字面量定义对象时使用 [] 定义([prop], 不能与上同时使用)
+ <strong>方法的name属性</strong>，返回方法函数名
     + getter / setter
     + bind返回的函数
     + Function构造函数返回的函数实例
+ <strong>属性的可枚举性和遍历</strong>：描述对象的enumerable
+ <strong>super关键字</strong>： 指向当前对象的原型对象（智能在对象的简写方法中）
+ <strong>Object.is()</strong>：对比两值是否相等
+ <strong>Object.assign()</strong>：合并对象（浅拷贝），返回原对象
+ <strong>Object.getPrototypeOf()</strong>：返回对象的原型对象
+ <strong>Object.setPrototypeOf()</strong>：设置对象的原型对象
+ <strong>`__proto__`</strong>：返回或设置对象的原型对象



<blockquote>属性遍历</blockquote>

+ 描述：可继承，非枚举，Symbol
+ 遍历
    + for-in: 遍历对象 自身可继承可枚举属性
    + Object.keys()：返回对象自身可枚举属性的键组成的数组
    + Object.getOwnPropertyNames(): 返回对象自身可继承属性的键组成的数组
    + Object.getOwnPropertySymbols(): 返回对象Symbol属性的键组成的数组
    + Reflect.ownKeys()：返回对象自身可继承Symbol属性的键组成的数组
+ 规则
    + 首先遍历所有数值键，按照数值升序排列
    + 其次遍历所有字符串键，按照加入时间升序排列
    + 最后遍历所有symbol键，按照加入时间升序排列



### 数组扩展
+ <strong>扩展运算符(...)</strong>：转换数组为用逗号分隔的参数序列
+ <strong>Array.from()</strong>：转换具有iterator接口的数据结构为真正数组， 返回新数组
    + 类数组对象：包含length 、argument、nodelist对象
    + 可遍历对象：String、Set解构、Map解构、Generator函数
+ <strong>Array.of()</strong>：转换一组值为真正数组，返回新数组
+ <strong>copyWithin(): </strong> 把指定位置的成员复制到其他位置，返回原数组
+ <strong>find()：</strong> 返回第一个符合条件的成员
+ <strong>findIndex()：</strong>返回第一个符合条件的成员索引值
+ <strong>fill()：</strong> 根据指定值填充整个数组， 返回原数组
+ <strong>keys()：</strong>返回以索引值为遍历器的对象
+ <strong>values()：</strong>返回以属性值为遍历器的对象
+ <strong>entries()：</strong>返回以索引值和属性值为遍历器的对象
+ <strong>数组空位：</strong>ES6明确将数组空位转为undefined



<blockquote>应用</blockquote>
+ 克隆数组：`const arr = [...arr]`
+ 合并数组：`const arr = [...arr1, ...arr2]`
+ 拼接数组：`arr.push(...arr1)`
+ 代替apply： `Math.max.apply(null, [x, y]) => Math.max(...[x, y])`
+ 转换字符串为数组： `[...'hello']`
+ 转换类数组对象为数组： `[...Arguments, ...nodeList]`
+ 转换可遍历对象为数组： `[...String, ...Set, ...Map, ...Generator]`
+ 与数组解构赋值结合： `const [x, ...rest/spread] = [1, 2, 3]`
+ 计算unicode字符长度： `Array.from('hello').length => [...'hello'].length`



### 函数扩展
+ <strong>参数默认值</strong>：为函数参数指定默认值
    + 形式：`function Func(x = 1, y = 2) {}`
    + 参数赋值：惰性求值（函数调用后才求值）
    + 参数位置：尾参数
    + 参数作用域：函数作用域
    + 声明方式：默认声明， 不能用const 、let再次声明
    + length：返回没有指定默认值的参数个数
    + 与解构赋值默认值结合：`function Func({x = 1, y = 2}) {}`
+ <strong>rest/spread参数</strong>：返回函数多余参数
    + 形式：以数组的形式存在，之后不能再有其他参数
    + 作用：代替arguments对象
    + length：返回没有指定默认的参数个数但不包括rest/spread参数
+ <strong>严格模式</strong>：在严格条件下运行js
    + 应用：只要函数参数使用默认值，解构赋值，扩展运算符，那么函数内部就不能设定严格模式
+ <strong>name属性</strong>：返回函数的函数名
    + 将匿名函数赋值给变量：空字符串、变量名
    + 将具名函数赋值给变量：函数名
    + bind返回的函数：函数名
    + Function构造函数返回的函数实例：anonymous
+ <strong>箭头函数</strong>：函数简写
    + 无参数： () => {}
    + 单个参数：x => {}
    + 多个参数：(x, y) => {}
    + 解构参数：({x, y}) => {}



<blockquote>注意事项</blockquote>
+ 函数体内的this是定义时所在的对象而不是使用时所在的对象
+ 可让this指向固定化，这种特性很有利于封装回调函数
+ 不可当做构造函数，因此箭头函数不可使用new
+ 不可使用argument对象，此对象在函数体内不存在
+ 返回对象时必须在对象外面加上括号



###  正则扩展
+ <strong>变更RegExp构造函数入参</strong>：允许首参数为正则对象，尾参数为正则修饰符（返回的正则表达式会忽略原正则的修饰符）
+ <strong>正则方法调用变更</strong>：字符串对象的match、replace、search、split
+ <strong>flags</strong>： 正则表达式的修饰符



### Symbol
+ 定义：独一无二的值
+ 声明： `const set = Symbol(str)`
+ 入参：字符串
+ 方法
    + <strong>Symbol()</strong>：创建以参数作为描述的Symbol值
    + <strong>Symbol.for()</strong>：创建以参数作为描述的Symbol值,如存在此参数则返回原有的Symbol值
    + <strong>Symbol.keyFor()</strong>：返回已登记的Symbol值描述
    + <strong>Object.getOwnPropertySymbols()</strong>：返回对象中所有用作属性名的symbol值的数组



<blockquote>数据类型</blockquote>
+ <strong>Undefined</strong>
+ <strong>Null</strong>
+ <strong>String</strong>
+ <strong>Number</strong>
+ <strong>Boolean</strong>
+ <strong>Object</strong>(包含Array、Function、Date、RegExp、Error)
+ <strong>Symbol</strong>

<blockquote>应用</blockquote>
+ 唯一化对象属性名：属性名属于Symbol类型，就都是独一无二的，可保证不会与其他属性名产生冲突
+ 遍历属性名：只能通过Object.getOwnPropertySymbols() 返回
+ 启动模块的Singleton模式：调用一个类在任何时候返回同一个实例

<blockquote>重点</blockquote>
+ Symbol()生成一个原始类型的值不是对象，因此Symbol() 前不能使用new
+ Symbol()参数表示对当前Symbol值的描述，相同参数的Symbol返回值不相等
+ Symbol值可通过String() / toString() 显式转为字符串
+ Symbol值作为对象属性名时，此属性是公开属性，但不是私有属性
+ Symbol值作为对象属性名时，只能用方括号运算符读取
+ Symbol值作为对象属性名时，不会被常规方法遍历得到



### Set
#### Set
+ 定义：类似于数组的数据结构，成员值都是唯一且没有重复的值
+ 声明：`const set = new Set(str)`
+ 入参：具有Iterator接口的数据结构
+ 属性：
    + <strong>constrcutor</strong>： 构造函数， 返回set
    + <strong>size</strong>：返回实例成员总数
+ 方法
    + <strong>add()</strong>：添加值，返回实例
    + <strong>delete()</strong>：删除值，返回布尔值
    + <strong>has()</strong>：检查值，返回布尔值
    + <strong>clear()</strong>：清空所有成员
    + <strong>keys()</strong>：返回以属性值为遍历器的对象
    + <strong>values()</strong>：返回以属性值为遍历器的对象
    + <strong>entries()</strong>：返回以属性值和属性值为遍历器的对象
    + <strong>forEach()</strong>：使用回调函数遍历每个成员

<blockquote>应用</blockquote>
+ 去重字符串：`[...new Set(str)].join('')`
+ 去重数组：`[...new Set(arr)] / Array.from(new Set(str))`
+ 集合数组：
    + 声明：`const a = new Set(arr1)`
    + 并集：`new Set([...a, ...b])`
    + 交集：`new Set([...a].filter(v => b.has(v)))`
    + 差集：`new Set([...a].filter(v => !b.has(v)))`
+ 映射集合：
    + 声明：`let set = new Set(str)`
    + 映射：`set = new Set([...set].map(v => v + 2))`

<blockquote>重点</blockquote>
+ 遍历顺序：插入顺序
+ 没有键只有值，可认为键和值相等
+ 添加多个NaN时，只会存在一个NaN
+ 添加相同的对象时，会认为是不同的对象
+ 添加值时不会发生类型转换
+ keys / values 的行为完全一致，entries()返回的遍历器同时包括键值



#### WeakSet
+ 定义：和Set结构类似，成员值只能是对象
+ 声明：`const set = new WeakSet(str)`
+ 入参：具有Iterator接口的数据结构
+ 属性：
    + <strong>constructor</strong>：构造函数，返回WeakSet
+ 方法：
    + <strong>add()</strong>：添加值，返回实例
    + <strong>delete()</strong>：删除值，返回布尔值
    + <strong>has()</strong>：检查值，返回布尔值

<blockquote>应用</blockquote>
+ 存储DOM节点：DOM节点被移除时自动释放此成员，不用担心这些节点从文档移除时会引发内存泄漏
+ 临时存放一组对象或存放跟对象绑定的信息：只要这些对象在外部消失，它在WeakSet结构中的引用就会自动消失

<blockquote>重点</blockquote>
+ 成员都是弱引用，垃圾回收机制不考虑WeakSet结构对此成员的引用
+ 成员不适合引用，它会随时消失，因此ES6规定WeakSet结构不可遍历
+ 其他对象不再引用成员时，垃圾回收机制会自动回收此成员所占用的内存，不考虑此成员是否还存在于WeakSet结构中



### Map
#### Map
+ 定义： 类似于对象的数据结构，成员键可以是任何类型的值
+ 声明：`const set = new Map(str)`
+ 入参：具有Iterator接口且每个成员都是一个双元素数组的数据结构
+ 属性
    + <strong>constructor</strong>：构造函数，返回Map
    + <strong>size</strong>：返回实例成员总数
+ 方法
    + <strong>get()</strong>：返回键值对
    + <strong>set()</strong>：添加键值对，返回实例
    + <strong>delete()</strong>：删除键值对，返回布尔值
    + <strong>has()</strong>：检查键值对，返回布尔值
    + <strong>clear()</strong>：清空所有成员
    + <strong>keys()</strong>：返回以键为遍历器的对象
    + <strong>values()</strong>：返回以值为遍历器的对象
    + <strong>entries()</strong>：返回键值为遍历器的对象
    + <strong>forEach()</strong>：使用回调函数遍历每个成员

<blockquote>重点</blockquote>
+ 遍历顺序：插入顺序
+ 对同一个键多次赋值：后面的值覆盖前面的值
+ 对同一个对象的引用：被视为一个键
+ 对同样值的两个实例：被视为两个键
+ 键跟内存地址绑定：只要内存地址不一样就视为两个键
+ 添加多个以NaN作为键时，只会存在一个以NaN作为键的值
+ Object结构提供 字符串-值 的对应，Map结构提供 值-值 的对应



#### WeakMap
+ 定义：和Map结构类似，成员键只能是对象
+ 声明：`const set = new WeakMap(str)`
+ 入参：具有 Iterator接口 且每个成员都是一个双元素数组的数据结构
+ 属性
    + <strong>constructor</strong>：构造函数，返回WeakMap
+ 方法：
    + <strong>get()</strong>：返回键值对
    + <strong>set()</strong>：添加键值对，返回实例
    + <strong>delete()</strong>：删除键值对，返回布尔值
    + <strong>has()</strong>：检查键值对，返回布尔值



<blockquote>应用</blockquote>
+ 存储DOM节点：DOM节点被移除时自动释放此成员键，不用担心这些节点从文档移除时会引发内存泄漏
+ 部署私有属性：内部属性是实例的弱引用，删除实例时它们也会随之消失，不会造成内存泄漏

<blockquote>重点</blockquote>
+ 成员键都是弱引用，垃圾回收机制不考虑WeakMapt结构对此成员键的引用
+ 成员键不适合引用，它会随时消失，因此ES6规定WeakMap结构不可遍历
+ 其他对象不再引用成员键时，垃圾回收机制会自动回收此成员所占用的内存，不考虑此成员是否还存在于WeakMap结构中
+ 一旦不再需要，成员会自动消失，不用手动删除引用
+ 弱引用的只是键而不是值，值依然是正常引用
+ 即使在外部消除了成员键的引用，内部的成员值依然存在



### Proxy
+ 定义：修改某些操作的默认行为
+ 声明：`const proxy = new Proxy(target, handler)`
+ 入参
    + <strong>target</strong>：拦截的目标对象
    + <strong>handler</strong>：定制拦截行为
+ 方法
    + <strong>Proxy.revocable()</strong>：返回可取消的Proxy实例(返回{proxy, revoke}，可通过revoke取消代理
+ 拦截方式
    + <strong>get()</strong>：拦截对象属性 读取
    + <strong>set()</strong>：拦截对象属性设置，返回布尔值
    + <strong>has()</strong>：拦截对象属性检查 k in obj， 返回布尔值
    + <strong>deleteProperty()</strong>：拦截对象属性删除delete obj[k], 返回布尔值
    + <strong>defineProperty()</strong>：拦截对象属性定义: Object.defineProperty)()
    + <strong>ownKeys()</strong>：拦截对象属性遍历 for-in, Object.keys() 返回数组
    + <strong>getOwnPropertyDescriptor()</strong>：拦截对象属性描述读取
    + <strong>getPrototypeOf()</strong>：拦截对象原型读取
    + <strong>setPrototypeOf()</strong>：拦截对象原型设置
    + <strong>isExtensible()</strong>：拦截对象是否可扩展读取
    + <strong>preventExtensions()</strong>：拦截对象不可扩展设置
    + <strong>apply()</strong>：拦截Proxy实例作为函数调用
    + <strong>construct()</strong>：拦截Proxy实例作为构造函数调用



<blockquote>应用</blockquote>
+ Proxy.revocable()：不允许直接访问对象，必须通过代理访问，一旦访问结束就收回代理权，不允许再次访问
+ get()：读取位置属性报错，读取数组负数索引的值，封装链式操作，生成DOM嵌套节点
+ set()：数据绑定，确保属性值设置符合要求，防止内部属性被外部读写
+ has()：隐藏内部属性不被发现，排除不符合属性条件的对象
+ deleteProperty()：保护内部属性不被删除
+ defineProperty()：阻止属性被外部定义
+ ownKeys()：保护内部属性不被遍历

<blockquote>重点</blockquote>
+ 要使Proxy起作用，必须针对实例进行操作，而不是针对目标对象进行操作
+ 没有设置任何拦截时，等于直接通向原对象
+ 属性被定义为 不可读写 /扩展/配置/枚举，使用拦截方法会报错
+ 代理下的目标对象，内部this指向Proxy代理

### Class
+ 定义：对一类具有共同特征的事物的抽象（构造函数语法糖）

+ 原理：类本身指向构造函数，所有方法定义在 prototype 上，可看作构造函数的另一种写法(class === Class.prototype.constructor)

+ 方法和关键字
    + <strong>constructor()</strong>：构造函数，new命令生成实例时自动调用
    + <strong>extends</strong>：继承父类
    + <strong>super</strong>：新建父类的this
    + <strong>static</strong>：定义静态属性方法
    + <strong>get</strong>：取值函数，拦截属性的取值行为
    + <strong>set</strong>：存值函数，拦截属性的存值行为
    
+ 属性
    + `__proto__`：构造函数的继承（总是指向父类）
    + `__proto__.__proto__`：子类的原型的原型，即父类的原型
    + `prototype.__proto__`：属性方法的继承（总指向父类的prototype）

+ 静态属性：定义类完成后赋值属性，该属性不会被实例继承，只能通过类来调用

+ 静态方法：使用static定义方法，该方法不会被实例继承，只能通过类来调用(方法中的this指向类，而不是实例)

+ 继承
    + 实质
        + ES5实质：先创造子类实例的this，再将父类的属性方法添加到this
        + ES6实质：先将父类实例的属性方法加到this上，再用子类构造函数修改this
    + super
        + 作为函数调用：只能在构造函数中调用super()，内部this指向继承的当前子类
        + 作为对象调用：在普通方法中指向父类的原型对象，在静态方法中指向父类
    + 显示定义：使用constructor() {super()} 定义继承父类，没有书写则显示定义
    + 子类继承父类：子类使用父类的属性方法时，必须在构造函数中调用super()，否则得不到父类的this
        + 父类静态属性方法可被子类继承
        + 子类继承父类后，可从super上调用父类静态属性方法
    
+ 实例：类相当于实例的原型，所有在类中定义的属性方法都会被实例继承
    + 显式指定属性方法：使用this指定到自身上(使用 Class.hasOwnProperty()可检测到)
    + 隐式指定属性方法：直接声明定义在对象原型上(使用`Class.__proto__.hasOwnProperty()`可检测到)
    
+ 表达式
    + 类表达式：`const Class = class {}`
    + name属性：返回紧跟class 后的类名
    + 属性表达式：[prop]
    + Generator方法：`* methods() {}`
    + Async方法：`async methods() {}`
    
+ this指向：解构实例属性或方法时会报错
    + 绑定this：`this.method = this.method.bind(this)`
    + 箭头函数：`this.method = () => this.method()`
    
+ 属性定义位置：
    + 定义在构造函数中并使用this指向
    + 定义在类最顶层
    
+ new.target： 确定构造函数是如何使用的



<blockquote>原生构造函数</blockquote>
+ String()
+ Number()
+ Boolean()
+ Array()
+ Object()
+ Fuunction()
+ Date()
+ RegExp()
+ Error()

<blockquote>重点</blockquote>
+ 在实例上调用方法，实质是调用原型上的方法
+ `Object.assign()`可方便地一次向类添加多个方法(`Object.assign(Class.prototype, {...})`)
+ 类内部所有定义的方法是不可枚举的(non-enumerable)
+ 构造函数默认返回实例对象(this)，可指定返回另一个对象
+ 取值函数和存值函数设置在属性的Description对象上
+ 类不存在变量提升
+ 子类继承父类后，this指向子类实例，通过super对某个属性赋值，赋值的属性会变成子类实例的属性
+ 使用super时，必须显式指定是作为函数还是作为对象调用
+ extends不仅可继承类还可继承原生的构造函数


<blockquote>私有属性方法</blockquote>
```
const name = Symbol('name');
const print = Symbol('print');
class Person {
	constructor(age) {
		this[name] = 'frontend';
		this.age = age;
	}
	[print]() {
		console.log(`${this[name]}`)
	}
}
```



### Module
+ 命令
    + <strong>export</strong>：规定模块对外接口
        + 默认导出：`export default Person`
        + 单独导出：`export const name = 'frontend'`
        + 按需导出：`export {age. name}`
        + 改名导出：`export {name as newName}`
    + <strong>import</strong>：导入模块内部功能
        + 默认导入：`import Person from 'person'`
        + 整体导入：`import * as Person from 'person'`
        + 按需导入：`import {age, name} from 'person'`
        + 改名导入：`import {name as newName} from 'person'`
        + 自执导入：`import 'person'`
        + 复合导入：`import Person, {name} from 'person'`
    + <strong>复合模式</strong>： export / import 结合在一起写成一行，变量实质没有被导入当前模块，相当于对外转发接口，导致当前模块无法直接使用其导入变量
        + 默认导入导出：`export {default} from 'person'`
        + 整体导入导出：`export * from 'person'`
        + 按需导入导出：`export {age, name} from 'person'`
        + 改名导入导出：`export {name as newName} from 'person'`
        + 具名改默认导入导出：`export {name as default} from 'person'`
        + 默认改具名导入导出：`export {default as name} from 'person'`
+ 继承：默认导出跟改名导出结合使用可使模块具备继承性
+ 设计思想：尽量的静态化，使得编译就能确定模块的依赖关系，以及输入输出的变量
+ 严格模式：ES6模块自动采用严格模式



<blockquote>加载方式</blockquote>
+ 运行时加载
    + 定义：整体加载模块生成一个对象，再从对象上获取需要的属性和方法进行加载(全部)
    + 影响：只有运行时才能得到是这个对象，导致无法在编译时做静态优化
+ 编译时加载
    + 定义：直接从模块中获取需要的属性和方法进行加载(按需加载)
    + 影响：在编译时就完成模块加载，效率比其他方案高，但无法引用模块本身(本身不是对象)

<blockquote>加载实现</blockquote>
+ 传统加载：通过 <script>进行 同步/异步 加载脚本
    + 同步加载：`<script src=""></script>`
    + Defer异步加载：`<script src='defer'></script>`(顺序加载 ，渲染完再执行)
    + Async异步加载：`<script src='async'></script>`(乱序加载，下载完就执行)

+ 模块加载：`<script type='module' src=''></script>`(默认是Defer异步加载)


<blockquote>重点</blockquote>
+ ES6模块中，顶层this指向undefined，不应该在顶层代码使用this
+ 一个模块就是一个独立的文件，该文件内部的所有变量，外部无法获取
+ export命令输出的接口与其对应的值是动态绑定关系，即通过该接口可获取模块内部实时的值
+ import命令大括号里的变量名必须与被导入模块对外接口的名称相同
+ import命令输入的变量只读，不允许在加载模块的脚本里改写接口
+ import命令具有提升效果，会提升到整个模块的头部，首先执行
+ 重复执行同一句import语句，只会执行一次
+ export default命令只能使用一次
+ export default导出的整体模块，在执行import时其后不能跟大括号
+ export default本质是输出一个名为default的变量，后面不能跟变量声明语句
+ export default和export可同时存在
+ export/import可出现在模块的任何位置，只要处于模块顶层即可，不能处于块级作用域
+ import加载模块成功后，此模块会作为一个对象，当做then的参数，可使用对象解构赋值来获取输出接口
+ 同时动态加载多个模块时，可使用Promise.all()和import()相结合实现
+ import() 和结合async/await 来书写同步操作的代码



### Iterator
+ 定义：为各种不同的数据结构提供统一的访问机制
+ 原理：创建一个指针指向首个成员，按照次序使用next()指向下一个成员，直接到结束为止
+ 作用
    + 为各种数据结构提供一个统一的简便的访问接口
    + 使得数据结构成员能够按照某种次序排列
    + ES6创造了新的遍历命令for-of, Iterator接口
+ 形式：for-of
+ 数据结构
    + 集合：Array、Object、Set、Map
    + 原生具备接口的数据结构：String / Array / Set / Map / TypedArray / Arguments / NodeList
+ 部署：默认部署在Symbol.iterator
+ 遍历器对象
    + <strong>next()</strong>：下一步操作，返回{done, value}(必须部署)
    + <strong>return()</strong>：for-of 提前退出应用，返回 {done: true}
    + <strong>throw()</strong>：不使用，配合Generator函数使用



<blockquote>for-of循环</blockquote>
+ 定义：调用 Iterator 接口产生遍历器对象
+ 遍历字符串：for-in获取索引，for-of获取值
+ 遍历数组：for-in 获取索引，for-of获取值
+ 遍历对象：for-in获取键，for-of需自行部署
+ 遍历Set：for-of获取值 => `for (const v of set)`
+ 遍历Map：for-of获取键值对 => `for(const [k, v] of map)`
+ 遍历类数组：包含length / Argument / NodeList
+ 计算生成数据结构：Array / Set / Map
    + keys()：返回遍历器对象，遍历所有的键
    + values()：返回遍历器对象，遍历所有的值
    + entries()：返回遍历器对象，遍历所有的键值对
+ 与 for-in区别
    + 有着同for-in一样简洁语法，但是没有for-in的缺点
    + 不同于forEach()，它可与break, continue, return配合使用
    + 提供遍历所有数据结构的统一操作接口

<blockquote>应用</blockquote>
+ 改写具有Iterator接口的数据结构的Symbol.iterator
+ 解构赋值：对Set进行解构
+ 扩展运算符： 将部署Iterator接口的数据结构转为数组
+ `yield*`： `yield*` 后跟一个可遍历的数据结构，会调用其遍历器接口
+ 接受数组作为参数的函数：for-of、Array.form()、new Set()、new WeakSet()、new Map()、new WeakMap()、Promise.all()、Promise.race()



### Promise
+ 定义：包含异步操作结果的对象
+ 状态
    + <strong>进行中</strong>：pending
    + <strong>已成功</strong>：resolved
    + <strong>已失败</strong>：rejected
+ 特点
    + 对象的状态不受外界影响
    + 一旦状态改变就不会在变，任何时候都可得到这个结果
+ 声明：`new Promise((resolve, reject) => {})`
+ 出参
    + resolve：将状态从未完成变为成功，在异步操作成功时调用，并将异步操作的结果作为参数传递出去
    + reject：将状态从未完成变为失败，在异步操作失败时调用，并将异步操作的错误作为参数传递出去
+ 方法
    + <strong>then()</strong>：分别制定resolved状态和rejected状态的回调函数
        + 第一参数：状态变为resolved时调用
        + 第二参数：状态变为rejected时调用
    + <strong>catch()</strong>：指定发生错误时的回调函数
    + <strong>Promise.all()</strong>：将多个实例包装成一个新实例，返回全部实例状态变更后的结果数组
    + <strong>Promise.race()</strong>：将多个实例包装成一个新实例，返回全部实例状态优先变更后的结果
    + <strong>Promise.resolve()</strong>：将对象转为Promise对象(`new Promise(resolve => resolve())`)
    + <strong>Promise.reject()</strong>：将对象转为状态为rejected的Promise对象(`new Promise((resolve, reject) => reject())`)



<blockquote>应用</blockquote>
+ 加载图片
+ Ajax转Promise对象

<blockquote>重点</blockquote>
+ 只有异步操作的结果可决定当前状态是哪一种，其他操作都无法改变这个状态
+ 状态改变只有两种可能：从pending到resolved，从pending到rejected
+ 一旦新建Promise对象就会立即执行，无法中途取消
+ 不设置回调函数，内部抛错不会反应到外部
+ 当处于pending时，无法得知目前进展到哪一个阶段
+ 实例状态变为resolved，rejected时，会触发then()绑定的回调函数
+ resolve()和reject()的执行总是晚于本轮循环的同步任务
+ then()返回新实例，其后可再调用另一个then()
+ then()运行中抛出错误会被catch()捕获
+ reject()的作用等同于抛出错误
+ 实例状态已变成resolved时，再抛出错误是无效的，不会被捕获，等于没有抛出
+ 实例状态的错误具有冒泡薪资，会一直向后传递知道捕获为止，错误总是被下一个catch()捕获
+ 不要在then()中定义rejected状态的回调函数
+ 没有使用catch()捕获错误，实例抛错不会传递到外层代码，即不会有任何反应



### Generator
+ 定义：封装多个内部状态的异步编程解决方案
+ 形式：调用 Generator函数(该函数不执行)返回指定内部状态的指针对象
+ 声明： `function* Func() {}`
+ 方法
    + <strong>next()</strong>：使指针移向下一个状态，返回{done, value}
    + <strong>return()</strong>：返回指定值且终结遍历Generator函数，返回{done: true, value: 入参}
    + <strong>throw()</strong>：在Generator函数体抛出错误，返回自定义的new Error()
+ yield命令：声明内部状态的值(return声明结束返回的值)
    + 遇到yield命令就暂停执行后面的操作，并将其后表达式的值作为返回对象的value
    + 下次调用next()时，再继续往下执行直到遇到下一个yield命令
    + 没有再遇到yield命令就一直运行到Generator函数结束，直到遇到return语句为止并将其后表达式的值作为返回对象的value
    + Generator函数没有return语句则返回对象的value为undefined
+ `yield*`：在一个Generator函数里执行另一个Generator函数
+ 遍历：通过 for-of 自动调用next()
+ 作为对象属性
    + 全写：`const obj = { method: function*() {} }`
    + 简写：`const obj = { * method() {} }`
+ 上下文：执行产生的上下文环境一旦遇到yield命令就会暂时退出堆栈(但不消失)，所有变量和对象会冻结在当前状态，等到对它执行next()时，这个上下文环境会重新加入调用栈，冻结的变量和对象恢复执行



<blockquote>应用</blockquote>
+ 异步操作下的同步表达
+ 控制流管理
+ 为对象部署Iterator接口：把Generator函数赋值给对象的Symbol.iterator，从而使该对象具有Iterator接口
+ 作为具有Iterator接口的数据结构

<blockquote>重点</blockquote>
+ 每次调用next()，指针就从 函数头部 或 上次停下的位置开始执行，直到遇到下一个yield命令或return语句为止
+ 函数内部可不用yield命令，但会变成单纯的暂缓执行函数
+ yield命令是暂停执行的标记，next是恢复执行的操作
+ yield命令本身没有返回值，可认为是返回undefined
+ yield为惰性求值，等next()执行到此才求值
+ 函数调用后生成遍历器对象，此对象的Symbol.iterator是此对象本身
+ 一旦next()返回对象的done为true，for-of遍历会终止且不包含该返回对象 
+ 函数内部部署try-finally且正在执行try, 那么return()会立即进入finally，执行完finally以后整个函数才会结束
+ 函数内部没有部署try-catch，throw()抛错将被外部try-catch捕获
+ throw() 被捕获以后，会附带执行下一条yield命令

<blockquote>首次next()可传值</blockquote>
```
function Func(func) {
	return function(...args){
		const generator = func(...args);
		generator.next();
		return generator;
	}
}
const console = Func(function* () {
	console.log(`first input: ${yield}`);
	return 'done';
});
console().next('welcome');
```



## ES2016

#### 数值扩展
+ 指数运算符(**)：数值求幂(相当于 Math.pow())

#### 数组扩展
+ includes()：是否存在指定成员



## ES2017

#### 字符串扩展
+ padStart()
+ padEnd()

#### 对象扩展
+ Object.getOwnPropertyDescriptors()：返回对象所有自身属性
+ Object.values()：返回以值组成的数组
+ Object.entries()：返回以键和值组成的数组

#### 函数扩展
+ 函数参数尾逗号：允许函数最后一个参数有逗号

#### Async
+ 定义：使异步函数以同步函数的形式书写(Generator函数语法糖)
+ 原理：将Generator函数和自动执行器spawn包装在一个函数里
+ 形式：将Generator函数的 * 替换成 async，将 yield 替换成 await
+ 声明
    + 具名函数：`async function Func() {}`
    + 函数表达式：`const func = async function() {}`
    + 箭头函数：`const func  = async() => {}`
    + 对象方法：`const obj = { async func() {} }`
    + 类方法：`class Cla { async Func() {} }`
+ await：等待当前Promise对象状态变更完毕
    + 正常情况：后面是Promise对象则返回其结果，否则返回对应的值
    + 后随Thenable对象：将其等同于Promise对象返回其结果
+ 错误处理： 将 await 的Promise对象放到try-catch中

<blockquote>重点</blockquote>
+ Async函数返回Promise对象，可使用then() 添加回调函数
+ 内部 return返回值 会成为后续then() 的出参
+ 内部抛出错误会导致返回的Pronmise对象变为rejected状态，被catch()捕获
+ 任何一个await的Promise对象变为rejected状态，整个Async函数都会中断执行
+ 数组使用forEach()执行 async/await会失效，可用 for-of 或 Promise.all代替



## ES2018

#### 字符串扩展
+ 放松对标签模板里字符串转义的限制：遇到不合法的字符串转义返回undefined，并且从raw上可获取原字符串

#### 对象扩展
+ 扩展运算符(...)：转换对象为用逗号分隔的参数序列

<blockquote>扩展</blockquote>
+ 克隆对象：`const obj = { __proto__ : Object.getPrototypeOf(obj1), ...obj1}`
+ 合并对象：`const obj = {...obj1, ...obj2}`
+ 修改现有对象部分属性：`const obj = {x: 1, ...{x: 2}}`

#### Promise
+ finally()： 指定不管最后状态如何都会执行的回调函数

#### Async
+ 异步迭代器(for-await-of)：循环等待每个Promise对象变为resolved状态才进入下一步



## ES2019

#### 字符串扩展
+ 字符串可直接输入 行分隔符 和 段分隔符
+ JSON.stringify()：可返回不符合UTF-8标准的字符串
+ trimStart()：消除字符串头部空格，返回新字符串
+ trimEnd()：消除字符串尾部空格，返回新字符串

#### 对象扩展
+ Object.fromEntries()：返回以键和值组成的对象(Object.entries()的逆操作)

#### 数组扩展
+ flat()：扁平化数组，返回新数组
+ flatMap()：映射且扁平化数组，返回新数组

#### 函数扩展
+ toString()：返回函数原始代码、
+ catch()：catch() 中的参数可省略

#### Symbol
+ description：返回Symbol值的描述

