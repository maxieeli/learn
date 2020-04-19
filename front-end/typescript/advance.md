# TypeScript进阶以及技巧

在学习了TypeScript的一些基本使用以及概念之后，这篇文章侧重于一些关于TypeScript的高级技巧，以及在基础上的一些扩展。

## 目录

  + [keyof](#keyof)
  + [interface & type](#interface)
  + [高级类型](#generic)
  + [typeof](#typeof)



### <span id="keyof">keyof</span>

keyof与Object.keys() 相似，只不过keyof取interface的键。

```typescript
// 例: 实现一个get函数来获取它的属性值
const data = {
  a: 3,
  hello: 'world'
};
function get(o: object, name: string) {
  return o[name];
}
```

以上的写法有两个缺点：无法确认返回类型，无法对key做约束。
这时可以使用keyof来加强get函数的类型功能：

```typescript
function get(o: T, name: K): T[K] {
  return o[name];
}
```

### <span id="generic">高级类型</span>

#### 交叉类型

必须同时满足接口里面的成员属性

```typescript
interface Dog {
  run(): void;
}
interface Cat {
  jump(): void;
}
const pet: Dog & Cat = {
  run() {},
  jump() {}
}
```

#### 联合类型

可以选择联合中的类型的其中一个。

```typescript
let age1: string | number = '10'; // ok
let age2: string | number = 10; // ok

// 有时不仅需要限定一个变量的类型，而且还要限定变量的值再某一个特定范围内
let limitAge: 10 | 20 | 30 = 10;
```

对象的联合类型在类型不确定的时候，只能访问所有类型的公有成员。

```typescript
interface DogInterface { run(): void; }
interface CatInterface { jump(): void; }
class Dog implements DogInterface {
  run() {};
  eat() {};
}
class Cat implements CatInterface {
  jump() {};
  eat() {};
}
enum Types{Dogs, Cats}
function getAnimal(type: Types) {
  const animal = Types.Dogs === type ? new Dog() : new Cat();
  animal.eat();
  return animal;
}
```

一个类型如果是多个类型的联合类型，并且每个类型之间有一个公共的属性，那么可以凭借这个公共属性，创建不同的类型保护区块。

```typescript
interface SquareInterface {
  types: 'square';
  width: number;
}
interface CircularInterface {
  types: 'circular';
  radius: number;
}
type Shape = SquareInterface | CircularInterface;

function getArea(s: Shape): number {
  switch(s.types) {
    case 'square':
      return s.width * s.width;
    case 'circular':
      return s.radius * 3.14;
    default:
      return ((e: never) => {throw new Error(e)})(s)
  }
}
let result: SquareInterface = { types: 'square', width: 100 };
getArea(result); // 10000
```

#### 索引类型

索引类型的操作符：

<b>keyof T</b> 表示类型T的所有公共属性的自变量的联合类型
<b>T[K]</b> 类型T的属性K代表的类型
<b>T extends U</b> 泛型变量可以通过继承某个类型或者某个属性

```typescript
const obj = {a: 1, b: 2, c: 3}
function getArr(obj: any, keys: string[]) {
  return keys.map(key => obj[key])
}
console.log(getArr(obj, ['b', 'c'])); // [2, 3]
console.log(getArr(obj, ['r', 's'])); // [undefined, undefined]

// 改进：利用索引类型
function getArr<T, K extends keyof T>(obj: T, keys: K[]): T[K][] {
  return keys.map(key => obj[key]);
}
// console.log(getArr(obj, ['r', 's'])); // error
```

#### 映射类型

先看个例子：

```typescript
interface Obj { a: number; b: string; }
let obj1: Obj = { a: 1, b: 'd' }
obj1.a = 3; // ok
```

如何把上面例子的接口变成只读或者可选呢？

```typescript
// 只读
type ReadonlyNew<T> = {
  readonly [P in keyof T]: T[P] // 索引T中的所有元素，设置成readonly
}
type ReadonlyObj = ReadonlyNew<Obj>;
let obj2: ReadonlyObj = {a: 2, b: 'c'};
obj2.a = 3;

// 可选
type PartialNew<T> = {
  readonly [P in keyof T]?: T[P];
}
type PartialObj = PartialNew<Obj>;
let obj3: PartialObj = {a: 2} // ok
let obj4: Obj = {a: 2} // Error
```

在上述的例子里，ReadonlyNew / PartialNew 就是 TS内置的 Readonly / Partial方法。
还有另外一种抽取Object子集的方法Pick:

```typescript
interface Obj1 {
  a: number; b: number;
  c: number; d: number;
}
type PickNew<T, K extends keyof T> = {
  [P in K]: T[P]
}
type PickObj = PickNew<Obj1, 'a' | 'd'>; // 只剩a,d两项
```

以上三种类型 Readonly / Partial / Pick, 官方成为同态，意味着只能做用于object属性，而不会引入新的属性。

```typescript
interface Obj1 {a: number; b: number; c: number; d: number;}
type RecordObj = Record<'x' | 'y', Obj1>;
// 此时的RecordObj是这样的：
RecordObj = { x: Obj1; y: Obj2; }
```

#### 条件类型

<b>T extends U ? X : Y</b> 这个表达式的意思是 如果类型 T 可以被赋值给类型 U，那么就是 X 类型，否则 Y 类型。

```typescript
type TypeName<T> =
  T extends string ? 'string' :
  T extends number ? 'number' :
  T extends boolean ? 'boolean' :
  T extends Function ? 'function' :
  T extends undefined ? 'undefined' :
  'object'

let res1: TypeName<string> = 'string';
type res2 = TypeName<number>; // type res2 = 'number'
```

<b>(A|B) extends U ? X : Y</b> 根据这个特性可以实现一些类型的过滤。

```typescript
type Diff<U, K> = U extends K ? never : U;

type dif1 = Diff<'a' | 'b' | 'c', 'a' | 'e'>; // 'b' | 'c'
type dif2 = Diff<'a', 'a' | 'e'>; // never
type _dif2 = Diff<"b", "a" | "e">; // b
type _dif3 = Diff<"c", "a" | "e">; // c

// diff 扩展，过滤掉某些类型
type NotNull<T> = Diff<T, undefined | null>;
type ts = NotNull<string | number | null | undefined>; // string|number
```

这些官方有内置的方法，对应如下：

  + <b>Exclude<T, U> -- 从T中剔除可以赋值给U的类型。</b>
  + <b>Extract<T, U> -- 提取T中可以赋值给U的类型。</b>
  + <b>NonNullable<T> -- 从T中剔除null和undefined。</b>
  + <b>ReturnType<T> -- 获取函数返回值类型。</b>
  + <b>InstanceType<T> -- 获取构造函数类型的实例类型.</b>

```typescript
type T00 = Exclude<'a' | 'b' | 'c' | 'd', 'a' | 'c' | 'f'>;  // b,d
type T01 = Extract<"a" | "b" | "c" | "d", "a" | "c" | "f">;  // a,c
type T02 = Exclude<string | number | (() => void), Function>;  // string | number
type T03 = Extract<string | number | (() => void), Function>;  // () => void
type T04 = NonNullable<string | number | undefined>;  // string,number
type T05 = NonNullable<(() => string) | string[] | null | undefined>;  // (() => string) | string[]
function f1(s: string) {
  return { a: 1, b: s };
}
class C {
  x = 0;
  y = 0;
}
type T06 = ReturnType<() => string>;  // string
type T07 = ReturnType<(s: string) => void>;  // void
type T08 = ReturnType<(<T>() => T)>;  // {}
type T09 = ReturnType<(<T extends U, U extends number[]>() => T)>;  // number[]
type T10 = ReturnType<typeof f1>;  // { a: number, b: string }
type T11 = ReturnType<any>;  // any
type T12 = ReturnType<never>;  // any
type T13 = ReturnType<string>;  // Error
type T14 = ReturnType<Function>;  // Error
type T15 = InstanceType<typeof C>;  // C
type T16 = InstanceType<any>;  // any
type T17 = InstanceType<never>;  // any
type T18 = InstanceType<string>;  // Error
type T19 = InstanceType<Function>; // Error
```
