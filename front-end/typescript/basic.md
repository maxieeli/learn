# TypeScript基础

TypeScript是Microsoft公司注册开发的。

TypeScript具有类型系统，且是JavaScript的超集，它可以编译成普通的JavaScript代码，TypeScript支持任意浏览器，任意环境，且开源。

## 目录

  + <a href="#basic">基础类型</a>
  + <a href="#interface">接口</a>
  + <a href="#class">类</a>
  + <a href="#function">函数</a>
  + <a href="#generic">泛型</a>
  + <a href="#enum">枚举</a>
  + <a href="#type">类型推论</a>
  + <a href="#higher">高级类型</a>
  + <a href="#module">模块</a>
  + <a href="#namespace">命名空间</a>
  + <a href="#merge">声明合并</a>



### <a name="basic">基础类型</a>

#### 介绍

TypeScript支持与JavaScript几乎相同的数据类型，此外还提供了实用的枚举类型方便使用。

#### 布尔值

最基本的数据类型就是 true/false 值，称为 boolean

```typescript
let isFalse: boolean = false;
let isTrue: boolean = true;
```

#### 数字

和JavaScript一样，TypeScript里的所有数字都是浮点数，这些浮点数的类型是number，除了支持十进制和十六进制字面量，TypeScript还支持ECMAScript2015中引入的二进制和八进制字面量。

```typescript
let decNum: number = 6;
let hexNum: number = 0xf00d;
let binaryNum: number = 0b1010;
let octalNum: number = 0o744;
```

#### 字符串

JavaScript使用string表示文本数据类型。 和JavaScript一样，可以使用双引号（"）或单引号（'）表示字符串。

```typescript
let name: string = 'xiaoming';
let sentence: string = `hello, ${name}`;
```

#### 数组

TypeScript像JavaScript一样可以操作数组元素，有以下两种定义方式。

```typescript
// 可再元素类型后面接上[]，表示由此类型元素组成的一个数组
let list: number[] = [1,2,3];

// 使用数组泛型，Array<元素类型>
let list: Array<number> = [1,2,3];
```

#### 元组Tuple

元组类型允许表示一个已知元素数量和类型的数组，各元素的类型不必相同。

```typescript
let x: [string, number];
x = ['hello', 10]; // ok
x = [10, 'hello']; // error

// 当访问一个已知索引的元素，会得到正确的类型
x[0].substr(1); // ok
x[1].substr(1); // error, 数字类型没有substr

// 当访问一个越界的元素，会使用联合类型代替
x[3] = 'hello'; // ok,字符串可以赋值给(string|number)类型
x[4].toString(); // ok，'string'和'number'都有toString
x[5] = true; // error, 布尔值不是(string|number)类型
```

#### 枚举

enum类型是对JavaScript标准数据类型的一个补充。

```typescript
// 默认情况下，从0开始为元素编号
enum Color { Red, Green, Blue }
let c: Color = Color.Green; // 1

// 也可以手动指定成员的数值
enum Color { Red = 1, Green, Blue }
let c: Color = Color.Green; // 2

// 或者全部手动赋值
enum Color {Red = 1, Green = 2, Blue = 4}
let c: Color = Color.Blue; // 4
```

枚举类型提供的一个便利是你可以由枚举的值得到它的名字，例：知道数值为2，但是不确定它映射到Color里的哪个名字，可以查找相应的名字:

```typescript
enum Color { Red = 1, Green, Blue }
let colorName: string = Color[2];
console.log(colorName); // Green
```

#### 任意值

有时会想要为那些在编程阶段还不清楚类型的变量指定一个类型，这些值可能来自于动态的内容，这种情况下，开发者不希望类型检查器对这些值进行检查而是直接让它们通过编译阶段的检查，那么就可以用 any 类型来标记这些变量:

```typescript
let notSure: any = 4;
notSure = 'maybe string'; // ok
notSure = false; // ok
```

在对现有代码进行改写时，any 类型十分有用，它允许在编译时可选择包含或移除类型检查，你可能认为 Object 有相似的作用，但是 Object 类型的变量只是允许给它赋任意值，却不能在它上面调用任意方法。

```typescript
let notSure: any = 4;
notSure.toFixed(); // ok

let objSure: Object = 4;
objSure.toFixed(); // error, Object不存在’toFixed‘
```

#### 空值

void类型与any类型相反，它表示没有任何类型，当一个函数没有返回值时，你通常会讲到其返回值类型是void

```typescript
function warnUser(): void {
  console.log('warning message');
}

// 声明一个void类型的变量没什么作用，因为只能赋予 undefined和null
let unusable: void = undefined;
```

#### Null和Undeined

TypeScript中，undefined和null两者各自有自己的类型，默认情况下 null 和 undefined 是所有类型的子类型，当指定了 `--strictNullChecks`标记，null和undefined只能赋值给void和自身。

```typescript
let u: undefined = undefined;
let n: null = null;
```

#### Never

never类型表示的是那些永远不存在的值的类型，never类型是任何类型的子类型，也可以赋值给任何类型，然而，没有类型是never的子类型或可以赋值给never类型，即使any也不可以赋值给never

```typescript
function error(msg: string): never {
  throw new Error(msg);
}
```

#### Object

object表示非原始类型，也就是除了number,string,boolean,symbol,null,undefined之外的类型

```typescript
declare function create(o: object | null): void;
create({props: 0}); // ok
create(null); // ok
create(54); // error
```

#### 类型断言

有时开发者会比TypeScript更了解某个值的详细信息，通常这会发生在你清楚的知道一个实体具有比它现有类型更确切的类型，通过类型断言告诉编译器。它没有运行时的影响，知识在编译阶段起作用。有以下两种断言形式：

```typescript
// 尖括号
let someValue: any = 'string';
let strLength: number = (<string>someValue).length;

// as语法
let someValue: any = 'string';
let strLength: number = (someValue as string).length;
```

两种方式等价，但是在JSX里面使用TypeScript，只有as语法断言是被允许的。



### <a name="interface">接口</a>

TypeScript的核心原则之一是对值所具有的结构进行类型检查，在TypeScript中，接口的作用就是为这些类型命名和为代码定义契约。

#### 接口初探

```typescript
function print(labelObj: { label: string }) {
  console.log(labelObj.label);
}
let myObj = { size: 10, label: 'size' }
print(myObj); // size
```

类型检查器会查看print的调用，print有一个参数，并要求这个对象参数有一个名为label类型的属性。下面用接口重写上述例子:

```typescript
interface labelValue {
  label: string;
}
function print(labelObj: labelValue) {
  console.log(labelObj.label);
}
```

另：类型检查器不会去检查属性的顺序，只要相应的属性存在并且类型也是正确的即可。

#### 可选属性

接口里的属性不全都是必需的，可选属性在应用 ’option bags‘模式时很常用，即给函数传入的参数对象中只有部分属性赋值了。

```typescript
interface SquareConfig {
  color?: string;
  width?: number;
}

function createSquare(config: SquareConfig):
  {color: string, area: number} {
  let newSquare = {color: 'white', area: 100};
  if(config.color) {
    newSquare.color = config.color;
  }
  if(config.width) {
    newSquare.area = config.width * config.width;
  }
  return newSquare;
}
let mySquare = createSquare({color, 'black'});
```

#### 只读属性

一些对象属性只能在对象刚创建的时候修改其值，可以在属性名前用readonly来指定只读属性

```typescript
interface Point {
  readonly x: number;
  readonly y: number;
}

let p: Point = {x: 10, y: 20};
p.x = 20; // error
```

#### 函数类型

接口能够描述JavaScript中对象拥有的各种各样的外形，除了描述带有属性的普通对象外，接口也可以描述函数类型。

为了使用接口表示函数类型，需要给接口定义一个调用签名，它就像是一个只有参数列表和返回值类型的函数定义，参数列表里的每个参数都需要名字和类型。

```typescript
interface SearchFunc {
  (source: string, subString: string): boolean;
}

// 定义后，可以像使用其它接口一样使用这个函数类型的接口
let mySearch: SearchFunc;
mySearch = function(source: string, sub: string) {
  let result = source.search(sub);
  return result > -1;
}
```

#### 可索引类型

与使用接口描述函数类型差不多，也可以描述那些能够’通过索引得到‘的类型，比如 a[10], a['age']，可索引类型具有一个索引签名，它描述了对象索引的类型，还有相应的索引返回值类型。

TypeScript支持两种索引签名：字符串和数字，可以同时使用两种类型的索引，但是数字索引的返回值必须是字符串索引返回值类型的子类型，这是因为当使用number来索引时，JavaScript会将它转换成string然后再去索引对象。

```typescript
interface StringArray {
  [index: number]: string;
}
let myArr: StringArray;
myArr = ['bob', 'owen'];

let myStr: string = myArr[0];
```

#### 类类型

与Java里接口的基本作用一样，TypeScript也能够用它来明确的强制一个类去符合某种契约。

接口描述了类的公共部分，而不是公共和私有两部分，它不会帮你检查类是否具有某些私有成员。

```typescript
interface ClockInterface {
  currentTime: Date;
}

class Clock implements ClockInterface {
  currentTime: Date = new Date();
  constructor(hour: number) {}
}
```

#### 继承接口

和类一样，接口也可以相互继承，能从一个接口里复制成员到另一个接口里，可以更灵活的将接口分割到可重用的模块里

```typescript
interface Shape { color: string; }
interface Stroke { width: number; }
interface Square extends Shape, Stroke {
  size: number;
}

let square = <Square>();
square.color = 'blue';
square.size = 10;
square.width = 5;
```

#### 混合类型

接口能够描述JavaScript里丰富的类型，因为JavaScript其动态灵活的特点。例：一个对象可以同时为函数和对象使用，并带有额外的属性。

```typescript
interface Counter {
  (start: number): string;
  interval: number;
  reset(): void;
}

function getCounter(): Counter {
  let counter = <Counter>function(start: number): string { return '' };
  counter.interval = 12;
  counter.reset = function() {}
  return counter;
}

let c = getCounter();
c(20);
c.reset();
c.interval = 5;
```



### <a name="class">类</a>

传统的JavaScript程序使用函数和基于原型的继承来创建可重用的组件。

#### 类

```typescript
class Greeter {
  greeting: string;
  constructor(msg: string) {
    this.greeting = msg;
  }
  greet() {
    return this.greeting;
  }
}

let greeter = new Greeter('hello');
```

使用new构造了Greeter类的一个实例，它会调用之前定义的构造函数，创建一个Greeter类型的新对象，并执行构造函数初始化它。


#### 继承

在TypeScript中，可以使用常用的面向对象模式。允许使用继承来扩展现有的类。

```typescript
class Animal {
  move(distance: number = 0) {
    console.log('animal move ${distance}');
  }
}

class Dog extends Animal {
  bark() {
    console.log('dog bark');
  }
}

const dog = new Dog();
dog.bark();
dog.move(10);
```

#### 修饰符

在类对象里面，可以自由的访问程序里定义的成员。在TypeScript中，分别用 `public, private, protected, readonly`分别修饰对成员的访问形式。

```typescript
// public: 可以访问任何定义的成员
class Animal {
  public name: string;
}

// private: 不能在声明它的类的外部访问
class Animal {
  private name: string;
  constructor(AnoName: string) { this.name = AnoName; }
}
new Animal('cat').name; // error

// protected: 与private相似，但是 protected成员在派生类中仍然可以访问
class Person {
  protected name: string;
  constructor(name: string) { this.name = name; }
}

class Employee extends Person {
  private department: string;
  constructor(name: string, department: string) {
    super(name)
    this.department = department;
  }
  public getPitch() {
    return `${this.department}`;
  }
}

let test = new Employee('test', 'developer');
test.getPitch(); // ok
test.name; // error

// readonly: 设置属性为只读，只读属性必须在声明时或构造函数里被初始化。
```

#### 存取器

TypeScript支持通过getters/setters来截取对对象成员的访问。它能帮助有效的控制对对象成员的访问。

```typescript
let pass = 'pass';
class Employee {
  private _fullName: string;
  get fullName(): string {
    return this._fullName;
  }
  set fullName(newName: string) {
    if(pass && pass == 'pass') {
      this._fullName = newName;
    } else {
      console.log('error')
    }
  }
}

let employee = new Employee();
employee.fullName = 'bob';
if(employee.fullName) {
  console.log(employee.fullName);
}
```

#### 静态属性

之前只讨论了类的实例成员，那些仅当被实例化的时候才会被初始化的属性，也可以创建类的静态成员，这些属性存在于类本身上面而不是类的实例上。每个实例想要访问这个属性的时候，都需要在成员前面加上类名。

```typescript
class Grid {
    static origin = {x: 0, y: 0};
    calculateDistance(point: {x: number; y: number;}) {
        let xDis = (point.x - Grid.origin.x);
        let yDis = (point.y - Grid.origin.y);
        return Math.sqrt(xDis * xDis + yDis * yDis) / this.scale;
    }
    constructor (public scale: number) { }
}

let grid1 = new Grid(1.0);  // 1x scale
let grid2 = new Grid(5.0);  // 5x scale

console.log(grid1.calculateDistance({x: 10, y: 10}));
console.log(grid2.calculateDistance({x: 10, y: 10}));
```

#### 抽象类

抽象类作为其他派生类的基类使用。它们一般不会直接被实例化。不同于接口，抽象类可以包含成员的实现细节，用关键字 abstract 表示。

抽象类中的抽象方法不包含具体实现并且必须在派生类中实现。 抽象方法的语法与接口方法相似。 两者都是定义方法签名但不包含方法体。 然而，抽象方法必须包含abstract关键字并且可以包含访问修饰符。

```typescript
abstract class Department {
  constructor(public name: string) {}
  printName(): void {
    console.log(this.name);
  }
  abstract printMeeting(): void;
}

class AccountDepartment extends Department {
  constructor() {}
  printMeeting(): void {
    console.log('print meeting methods');
  }
  generate(): void {
    console.log('generate method');
  }
}

let department: Department; // 允许创建一个对抽象类型的引用
department = new Department(); // 错误: 不能创建一个抽象类的实例
department = new AccountingDepartment(); // 允许对一个抽象子类进行实例化和赋值
department.printName();
department.printMeeting();
department.generateReports(); // 错误: 方法在声明的抽象类中不存在
```



### <a name="function">函数</a>

函数是JavaScript应用程序的基础，它帮助实现抽象层，模拟类，模块。在TypeScript中，虽然已经支持类，命名空间，当函数仍然是主要的定义行为的地方。

#### 函数类型

为函数定义类型：

```typescript
function add(x: number, y: number): number {
  return x + y;
}
let myAdd = function(x: number, y:number): number {
  return x + y;
}

// 以下为完整函数类型
let myAdd:(x: number, y: number) => number
  = function(x: number, y: number): number {
  return x + y;
}
```

函数类型包含两部分：参数类型和返回值类型。只要参数类型是匹配的，那么就认为它是有效的函数类型，而不在意参数名是否正确。

第二部分是返回值类型，对于返回值，在函数和返回值类型之前使用 => 符号，返回值类型是函数类型的必要部分，如果函数没有返回任何值，也必须制定返回值类型(void)而不能为空。

#### 可选参数和默认参数

TypeScript里的每个函数参数都是必须的，这不是指不能传递null或undefined作为参数，而是说编译器检查用户是否为每个参数都传入了值，简单来说，传递给一个函数的参数个数必须与函数期望的参数个数一致。

```typescript
function buildName(fName: string, lName: string) {
  return `${fName}-${lName}`;
}

let r1 = buildName('test'); // error
let r2 = buildName('test1', 'test2', 'test3'); // error
let r3 = buildName('test1', 'test2'); // ok

// 与普通可选参数不同，带默认值的参数不需要放在必须参数后面，如果带默认值的参数出现在
// 必须参数前面，用户必须明确的传入undefined值来获得默认值
function buildName(fName = 'john', lName: string) {
  return `${fName} - ${lName}`;
}
let r1 = buildName('test'); // error
let r2 = buildName('test1', 'test2', 'test3'); // error
let r3 = buildName('test1', 'test2'); // ok
let r4 = buildName(undefined, 'test2'); // ok
```

#### 剩余参数

必要参数，默认参数和可选参数有个共同点，它们表示某一个参数，在JavaScript中，可以使用 arguments 来访问所有传入的参数。

剩余参数会被当做不限的可选参数，可以一个都没有，可以是任意个。

```typescript
function buildName(fName: string, ...restName: string[]) {
  return fName + ' ' + restName.join(' ')
}
let result = buildName('test1', 'test2', 'test3');
```

#### this和箭头函数

JavaScript里，this的值在函数被调用时才会指定，这是个强大灵活的特点，但是需要了解函数调用的上下文。

但是在TypeScript中，如果设置了 `--noImplicitThis`标记，它里面的this类型为any。这是因为this来自对象字面量里的函数表达式。修改的方法为，提供一个显式的this参数：

```typescript
interface Card {
  suit: string;
  card: number;
}
interface Deck {
  suits: string[];
  cards: number[];
  createCardPicker(this: Deck): () => Card;
}
let deck: Deck = {
  suits: ["hearts", "spades", "clubs", "diamonds"],
  cards: Array(20),
  createCardPicker: function(this: Deck) {
    return () => {
      let pickedCard = Math.floor(Math.random() * 20);
      let pickedSuit = Math.floor(pickedCard / 4);
      return {suit: this.suits[pickedSuit], card: pickedCard % 5};
    }
  }
}

let cardPicker = deck.createCardPicker();
let pickedCard = cardPicker();
```

现在TypeScript知道createCardPicker期望在某个Deck对象上调用，也就是this是Deck类型的，而非any。

当将一个函数传递给某个库函数调用时，可能会碰到回调函数的this会报错，因为当回调函数被调用时，它会被当成一个普通函数调用，this为undefined。解决方式可以通过this参数来避免，指定this的类型。

#### 重载

JavaScript本身是动态语言，所以函数根据不同的参数而返回不同类型的数据是常见的。

为了让编译器能够选择正确的检查类型，与JavaScript处理相似，它查找重载列表，尝试使用第一个重载定义。如果匹配就使用，因为在定义重载时，一定要最精确的定义放在最前面。

```typescript
let suits = ["hearts", "spades", "clubs", "diamonds"];

function pickCard(x: {suit: string; card: number; }[]): number;
function pickCard(x: number): {suit: string; card: number; };
function pickCard(x): any {
  if (typeof x == "object") {
    let pickedCard = Math.floor(Math.random() * x.length);
      return pickedCard;
  }
  else if (typeof x == "number") {
    let pickedSuit = Math.floor(x / 5);
    return { suit: suits[pickedSuit], card: x % 5 };
  }
}

let myDeck = [{ suit: "suit1", card: 2 }, { suit: "suit2", card: 10 }, { suit: "suit3", card: 4 }];
let pickedCard1 = myDeck[pickCard(myDeck)];
console.log(`card:${pickedCard1.card} of ${pickedCard1.suit}`);
let pickedCard2 = pickCard(15);
console.log(`card:${pickedCard2.card} of ${pickedCard2.suit}`);
```



### <a name="generic">泛型</a>

像Java语言中，可以使用泛型来创建可重用的组件，一个组件可以支持多种类型的数据。这样用户就可以以自己的数据类型来使用组件。

```typescript
function identity<T>(arg: T): T {
  return arg;
}
```

这里给identity添加了类型变量T，T帮助捕获用户传入的类型(如number)，之后用户就可以使用这个类型。之后可以再次使用 T 当做返回值类型，现在可以知道的是参数类型与返回值类型是相同的，这允许跟踪函数里使用的类型的信息。

#### 使用泛型变量

使用泛型创建像identity这样的泛型函数时，编译器要求在函数体必须正确的使用这个通用的类型。也就是说，必须把这些参数当做是任意或所有类型。

```typescript
function lenIdentity<T>(arg: T): T {
  console.log(arg.length); // error,没有指明arg具有这个属性
  return arg;
}

// lenIdentity的类型: 泛型函数lenIdentity,接收类型参数T和参数arg
// 它是个元素类型是T的数组，并返回元素类型为T的数组。
function lenIdentity<T>(arg: Array<T>): Array<T> {
  console.log(arg.length); // ok
  return arg;
}
```

#### 泛型类型

泛型函数的类型与非泛型函数的类型没有什么不同，只是有一个类型参数在最前面：

```typescript
function identity<T>(arg: T): T{
  return arg;
}
let myIdentity: <T>(arg: T) => T = identity;
// 也可以使用带有调用签名的对象字面量来定义
let myIdentity: {<T>(arg: T): T} = identity;

// 也可以使用接口的形式
interface IdentityFn {
  <T>(arg: T): T;
}
function identity<T>(arg: T): T {
  return arg;
}
let myIdentity: IdentityFn = identity;
```

除了泛型接口，还可以创建泛型类，但是无法创建泛型枚举和泛型命名空间。

#### 泛型类

泛型类与泛型接口差不多，泛型类使用<>括起泛型类型，跟在类名后面。
与接口一样，直接把泛型类型放在类后面，可以帮助我们确认类的所有属性都在使用相同的类型。
在类的时候说过，类有静态部分和实例部分，泛型类指的是实例部分的类型，所以静态属性不能使用这个泛型类型。

```typescript
class GenericNumber<T> {
  zeroValue: T;
  add(x: T, y: T) => T
}
let myNumber = new GenericNumber<number>();
myNumber.zeroValue = 0;
myNumber.add = function(x, y) { return x + y; };
```

#### 泛型约束

相比于操作any所有类型，用户想要限制函数去处理任意带有属性的所有类型，只要传入的类型有这个属性则允许，为此，需要列出对T的约束要求。

```typescript
interface LenLimit {
  length: number;
}
function lenIdentity<T extends LenLimit>(arg: T): T {
  console.log(arg.length);
  return arg;
}
lenIdentity(10); // error
lenIdentity({length: 10, value: 3}); // ok
```



### <a name="enum">枚举</a>

使用枚举可以定义一些带名字的常量，使用枚举可以清晰的表达意图或创建一组有区别的用力，TypeScript支持数字和基于字符串的枚举。

#### 数字枚举

```typescript
// 可默认，可自定义
enum Direction {
  Up,
  Down,
  Left,
  Right
}
```

#### 字符串枚举

在一个字符串枚举里，每个成员都必须用字符串字面量，或另外一个字符串枚举成员进行初始化、

```typescript
enum Direction {
  Up = 'UP',
  Down = 'DOWN',
  Left = 'LEFT',
  Right = 'RIGHT'
}
```

#### 联合枚举

存在一种特殊的非计算的常量枚举成员的子集：字面量枚举成员。字面量枚举成员是指不带有初始值的常量枚举成员。或者是值被初始化为：

  + 任何字符串字面量（例如："foo"，"bar"，"baz"）
  + 任何数字字面量（例如：1, 100）
  + 应用了一元符号的数字字面量（例如：-1, -100）

```typescript
enum Shape {
  Circle,
  Square
}
interface Circle {
  kind: Shape.Circle;
  radius: number;
}
interface Square {
  kind: Shape.Square;
  side: number;
}

let c: Circle = {
  kind: Shape.Square,
  radius: 20
}
```



### <a name="higher">高级类型</a>

#### 交叉类型

交叉类型是将多个类型合并为一个类型，可以把现有的多种类型叠加到一起成为一种类型，它包含了所需的所有类型的特性。
开发者大多是在mixin或者它不适合典型面向对象模型的地方看到交叉类型的使用。

```typescript
function extend<First, Second>(first: First, second: Second)
  : First & Second {
  const result: Partial<First & Second> = {};
  for(const prop in first) {
    if(first.hasOwnProperty(prop)) {
      (<First>result)[prop] = first[prop];
    }
  }
  for(const prop in second) {
    if(second.hasOwnProperty(prop)) {
      (<Second>result)[prop] = second[prop];
    }
  }
  return <First & Second>result;
}

class Person {
  constructor(public name: string) { }
}

interface Loggable {
  log(name: string): void;
}

class ConsoleLogger implements Loaggable {
  log(name) {
    console.log(name);
  }
}

const test = extend(new Person('Test'), ConsoleLogger.prototype);
test.log(test.name); // Test
```

#### 联合类型

联合类型表示一个值可以是几种类型之一。 用竖线（|）分隔每个类型，所以number | string | boolean表示一个值可以是number，string，或boolean。
如果一个值是联合类型，只能访问此联合类型的所有类型里共有的成员。

```typescript
interface Bird {
  fly();
  eggs();
}
interface Fish {
  swim();
  eggs();
}
function getPet(): Bird | Fish {
  // ...
}
let pet = getPet();
pet.eggs(); // ok
pet.swim(); // error
```

#### 类型别名

类型别名会给一个类型起个新名字。 类型别名有时和接口很像，但是可以作用于原始值，联合类型，元组以及其它任何你需要手写的类型。

```typescript
type Color = string;
type ColorResolver = () => string;
type ColorOrResolver = Color | ColorResolver;
function getColor(c: ColorOrResolver): Color {
  if(typeof c === 'string') {
    return c;
  } else {
    return c();
  }
}
```

#### 字面量类型

字符串字面量类型允许你指定字符串必须的固定值。 在实际应用中，字符串字面量类型可以与联合类型，类型守卫和类型别名很好的配合。 通过结合使用这些特性，你可以实现类似枚举类型的字符串。

TypeScript还具有数字字面量类型

```typescript
type Easing = "ease-in" | "ease-out" | "ease-in-out";
class UIElement {
  animate(dx: number, dy: number, easing: Easing) {
    if (easing === "ease-in") { }
    else if (easing === "ease-out") { }
    else if (easing === "ease-in-out") { }
    else { }
  }
}
let button = new UIElement();
button.animate(0, 0, "ease-in");
button.animate(0, 0, "uneasy"); // error

// 数字字面量类型
function foo(): 1 | 2 | 3 | 4 { // ... }
```

#### 多态的this类型

多态的this类型表示的是某个包含类或接口的子类型。 这被称做F-bounded多态性。 它能很容易的表现连贯接口间的继承，例： 在计算器类里，在每个操作之后都返回this类型：

```typescript
class BasicCalculator {
  public constructor(protected value: number = 0) { }
  public currentValue(): number {
    return this.value;
  }
  public add(operand: number): this {
    this.value += operand;
    return this;
  }
  public multiply(operand: number): this {
    this.value *= operand;
    return this;
  }
  // ...
}
let t = new BasicCalculator(2).multiply(5).add(1).currentValue();
```

#### 索引类型

使用索引类型，编译器能够检查使用了动态属性名的代码，常见的就是从对象中选取属性的子集。

以下通过索引类型查询和索引访问操作符：

```typescript
function pluck<T, K extends keyof T>(o: T, names: K[]): T[K][] {
  return names.map(n => o[n]);
}
interface Person {
  name: string;
  age: number;
}
let person: Person = {
  name: 'owen',
  age: 30
};
let strings: string[] = pluck(person, ['name']);
```



### <a name="module">模块</a>

从ECMAScript 2015开始，JavaScript引入了模块的概念。TypeScript也沿用这个概念。
模块在其自身的作用域里执行，而不是在全局作用域里；这意味着定义在一个模块里的变量，函数，类等等在模块外部是不可见的，除非你明确地使用export形式之一导出它们。 相反，如果想使用其它模块导出的变量，函数，类，接口等的时候，必须要导入它们，可以使用import形式之一。

模块是自声明的；两个模块之间的关系是通过在文件级别上使用imports和exports建立的。

模块使用模块加载器去导入其它的模块。 在运行时，模块加载器的作用是在执行此模块代码前去查找并执行这个模块的所有依赖。 大家最熟知的JavaScript模块加载器是服务于Node.js的CommonJS和服务于Web应用的Require.js。
TypeScript与ECMAScript 2015一样，任何包含顶级import或者export的文件都被当成一个模块。 相反地,如果一个文件不带有顶级的import或者export声明，那么它的内容被视为全局可见的（因此对模块也是可见的）。

#### 导出

任何声明(变量，函数，类，类型别名或接口)，都能通过添加export关键字导出。

```typescript
export interface StringValidator {
  isAccept(s: string): boolean;
}

class codeValidator {
  // ...
}
export { codeValidator };
```

#### 导入

导入与导出一样的操作，采用import形式来导入模块中的内容

```typescript
import {codeValidator} from './codeValidator';

// 将整个模块导入到一个变量，通过它来访问模块的导出部分。
import * as validator from './codeValidator';
```

### <a name="namespace">命名空间</a>

随着更多模块的加入，开发者需要一种手段来组织代码，以便在记录它们类型的同时还不用担心与其它对象产生命名冲突。 因此，把模块包裹到一个命名空间内，而非把它们放在全局命名空间下。

下面的例子里，把所有与验证器相关的类型都放到一个叫做Validation的命名空间里。 因为想让这些接口和类在命名空间之外也是可访问的，所以需要使用export。 相反的，变量lettersRegexp和numberRegexp是实现的细节，不需要导出，因此它们在命名空间外是不能访问的。 在文件末尾的测试代码里，由于是在命名空间之外访问，因此需要限定类型的名称，比如Validation.LettersOnlyValidator。

```typescript
namespace Validation {
  export interface StrValidator {
    isAcceptable(s: string): boolean;
  }
  const lettersRegexp = /^[A-Za-z]+$/;
  const numberRegexp = /^[0-9]+$/;
  export class letterValidator implements StrValidator {
    isAcceptable(s: string) {
      return lettersRegexp.test(s);
    }
  }
  export class codeValidator implements StrValidator {
    isAcceptable(s: string) {
      return s.length === 5 && numberRegexp.test(s);
    }
  }
}

let strings = ["Hello", "98052", "101"];
let validators: { [s: string]: Validation.StrValidator; } = {};
validators["ZIP code"] = new Validation.codeValidator();
validators["Letters only"] = new Validation.letterValidator();

for (let s of strings) {
  for (let name in validators) {
    console.log(`${ s } - ${ name }`);
  }
}
```



### <a name="merge">声明合并</a>

TypeScript中有些独特的概念可以在类型层面上描述JavaScript对象的模型。 这其中尤其独特的一个例子是“声明合并”的概念。 理解了这个概念，将有助于操作现有的JavaScript代码。 同时，也会有助于理解更多高级抽象的概念。

“声明合并”是指编译器将针对同一个名字的两个独立声明合并为单一声明。 合并后的声明同时拥有原先两个声明的特性。 任何数量的声明都可被合并；不局限于两个声明。

#### 合并接口

从根本上说，合并的机制是把双方的成员放到一个同名的接口里。

接口的非函数的成员应该是唯一的。 如果它们不是唯一的，那么它们必须是相同的类型。 如果两个接口中同时声明了同名的非函数成员且它们的类型不同，则编译器会报错。

```typescript
interface Box {
  height: number;
  width: number;
}
interface Box {
  scale: number;
}
let box: Box = {height: 5, width: 10, scale: 15};
```

#### 合并命名空间

与接口相似，同名的命名空间也会合并其成员。 命名空间会创建出命名空间和值，需要知道这两者都是怎么合并的。

对于命名空间的合并，模块导出的同名接口进行合并，构成单一命名空间内含合并后的接口。

对于命名空间里值的合并，如果当前已经存在给定名字的命名空间，那么后来的命名空间的导出成员会被加到已经存在的那个模块里。

```typescript
namespace Animals {
  export class Zebra { }
}
namespace Animals {
  export interface Legged { numberOfLegs: number; }
  export class Dog { }
}
```

#### 合并命名空间和类

```typescript
class Album {
  label: Album。AlbumLabel;
}
namespace Album {
  export class AlbumLabel { }
}
```
