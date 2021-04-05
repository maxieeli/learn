# React Hooks详解

# React Hook

## 一、什么是Hooks

+ React一直都是提倡使用函数组件，但是有时需要使用state或其他一些功能时，只能使用类组件。因为函数组件没有实例，没有生命周期函数，只有类组件才有
+ Hooks是React 16.8后新增的特性，它可以在不编写class的情况下使用state以及其他的React特性。
+ 如果在编写函数组件并意识到需要向其添加一些state，以前的做法是必须将其转化为class。现在可以直接在现有的函数组件中使用Hooks

## 二、Hooks解决的问题

### 1. 类组件的不足

+ <b>状态逻辑难复用</b>
    + 在组件之间复用状态逻辑很难，可能要用到 render props(渲染属性)或 HOC(高阶组件)，但无论是哪种方式，都会在原先的组件外包裹一层父容器，导致层级冗余。
+ <b>趋向复杂难以维护</b>
    + 在生命周期函数中混杂不相干的逻辑, 如在`componentDidMount`中注册时间以及其他的逻辑，在`componentWillUnmount`中卸载时间，这样分散不集中的写法。
    + 类组件中导出都是对状态的访问和处理，导致组件难以拆分成更小的组件、
+ <b>this指向问题</b>
    + 父组件给子组件传递函数时，必须绑定this
    + react中的组件四种绑定this方法的区别

```javascript
class App extends React.Component<any, any> {
  handleClick2;
  constructor(props) {
    super(props)
    this.state = {
      num: 1,
      title: 'react hooks'
    }
    this.handleClick2 = this.handleClick1.bind(this)
  }
  handleClick1() {
    this.setState({ num: this.state.num + 1 })
  }
  handleClick3 = () => {
    this.setState({ num: this.state.num + 1 })
  }
  render() {
    return (
      <div>
        <span>{this.state.num}</span>
        <button onClick={this.handleClick2}>btn1</button>
        <button onClick={this.handleClick1.bind(this)}>btn2</button>
        <button onClick={() => this.handleClick1()}>btn3</button>
        <button onClick={this.handleClick3}>btn4</button>
      </div>
    )
  }
}
```

前提：子组件内部做了性能优化(React.PureComponent)

+ 第一种是在构造函数中绑定this：那么每次父组件刷新时，如果传递给子组件其他的props值不变，那么子组件不会刷新。
+ 第二种是在render()函数里面绑定this：因为<b>bind函数会返回一个新的函数</b>，所以每次父组件刷新时，都会重新生成一个函数，即使父组件传递给子组件其他的props值不变，子组件每次都会刷新。
+ 第三种是使用箭头函数：父组件刷新的时候，即使两个箭头函数的函数体是一样的，都会生成一个新的箭头函数，所以子组件每次都会刷新
+ 第四种是使用类的静态属性：原理与第一种相似，但是更简洁


### 2. Hooks优势

+ 能优化类组件的三大问题
+ 能在无需修改组件结构的情况下复用状态逻辑(自定义hooks)
+ 能将组件中相互关联的部分拆分成更小的函数(比如设置订阅或请求数)
+ 副作用：指的是那些没有发生在数据向视图转换过程中的逻辑，如ajax请求，访问原生DOM元素、本地持久化缓存、绑定/解绑事件，添加订阅、设置定时器、记录日志等，以往这些副作用都是写在类组件的生命周期函数中，而 `useEffect`在全部渲染完毕后才会执行，`useLayoutEffect`会在浏览器layout之后，painting之前执行。



## 三、使用注意事项
+ 只能在函数内部的最外层调用Hook，不能在循环，条件判断以及子函数中调用。
+ 只能在React的函数组件中调用hooks



## 四、useState & useMemo & useCallback

+ 当多次调用useState的时候，要确保每次渲染时它们的调用顺序是不变的。
+ 通过在函数组件里调用它来给组件添加一个内部state，React会在重复渲染时保留这个state
+ useState唯一的参数就是初始state
+ useState会返回一个数组，一个state，一个更新state的函数
    + 在初始化渲染期间，返回的state与传入的第一个参数(initialState)值相同
    + 可以在事件处理函数中或其他地方调用这个函数，它类似class组件的this.setState，但是它不会把新的state和旧的state进行合并，而是直接替换

```javascript
//这里可以任意命名，因为返回的是数组，数组解构
const [state, setState] = useState(initialState)
```

### 4.1 使用例子

```javascript
import React, {useState} from 'react'
import ReactDOM from 'react-dom'

function Child1(props) {
  const {num, handleClick} = props
  return (
    <div onClick={() => {handleClick(num + 1)}}>
      child
    </div>
  )
}

function Child2(props) {
  const {text, handleClick} props
  return (
    <div>
      <ReactComp text={text} handleClick={handleClick} />
    </div>
  )
}

function ReactComp(props) {
  const {text, handleClick} = props
  return (
    <div onClick={() => {handleClick(text + 1) }}>ReactComp</div>
  )
}

function Parent() {
  let [num, setNum] = useState(0)
  let [text, setText] = useState(1)
  return (
    <div>
      <Child1 num={num} handleClick={setNum} />
      <Child2 text={text} handleClick={setText} />
    </div>
  )
}

const rootElement = document.getElementById('root')
ReactDOM.render(<Parent />, rootElement)
```

### 4.2 每次渲染都是独立的闭包

+ 每一次渲染都有它自己的props和state
+ 每一次渲染都有它自己的事件处理函数
+ 当点击更新状态时，函数组件都会重新被调用，那么每次渲染都是独立的，取到的值不会受后面操作的影响。

```javascript
function Counter2() {
  let [number, setNumber] = useState(0)
  function alertNumber() {
    setTimeout(() => {
      alert(number)
    }, 3000)
  }
  return (
    <>
      <p>{number}</p>
      <button onClick={() => setNumber(number + 1)}>+</button>
      <button onClick={alertNumber}>alertNumber</button>
    </>
  )
}
```

### 4.3 函数式更新

+ 如果新的state需要通过使用先前state计算得出，那么可以将回调函数当做参数传递给setState,该回调函数将接收先前的state，并返回一个更新后的值。

```javascript
function Counter() {
  let [number, setNumber] = useState(0)
  function lazy() {
    setTimeout(() => {
      // 这样每次执行时都会去获取一遍state，而不是使用点击触发时的那个state
      setNumber(number => number + 1)
    })
  }
  return (
    <>
      <p>{number}</p>
      <button onClick={() => setNumber(number + 1)}>+</button>
      <button onClick={lazy}>lazy</button>
    </>
  )
}
```

### 4.4 惰性初始化state

+ initialState参数只会在组件的初始化渲染中起作用，后续渲染时会被忽略
+ 如果初始state需要通过复杂计算获得，则可以传入一个函数，在函数中计算并返回初始的state，此函数只在初始渲染时被调用

```javascript
function Counter (props) {
  function getInitState() {
    return { number: props.number }
  }
  let [counter, setCounter] = useState(getInitState)
  return(
    <>
      <p>{counter.number}</p>
      <button onClick={() => 
        setCounter({number: counter.number + 1})
      }>+</button>
      <button onClick={() => setCounter(counter)}>setCounter</button>
    </>
  )
}
```

### 4.5 性能优化

#### 4.5.1 Object.is(浅比较)

+ Hooks内部使用 Object.is来比较新旧state是否相等。
+ 与class组件中的setState方法不同，如果你修改状态时，传的状态值没有变化，则不重新渲染。
+ 与class组件中的setState方法不同。useState不会自动合并更新对象，可以用函数式的setState结合展开运算符来达到合并更新对象的效果

```javascript
function Counter() {
  const [counter, setCounter] = useState({name: 'react', number: 0})
  return (
    <>
      <p>{counter.name}:{counter.number}</p>
      <button onClick{()=>
        setCounter({...counter,number:counter.number+1})
      }>+</button>
      <button onClick={()=>setCounter(counter)}>++</button>
    </>
  )
}
```

#### 4.5.2 减少渲染次数

+ 默认情况，只要父组件状态变了(不管子组件依不依赖该状态)，子组件也会重新渲染
+ 一般的优化：
    + 1.类组件：可以使用PureComponent
    + 2.函数组件：使用 React.memo，将函数组件传递给memo之后，就会返回一个新的组件，新组件的功能：如果接受到的属性不变，则不重新渲染函数。

但是怎么保证属性不会变呢？这里使用useState，每次更新都是独立的，也就是说每次都会生成一个新的值(哪怕这个值没有变化)，即使使用了React.memo，也还是会重新渲染

```javascript
import React, {useState, memo, useMemo, useCallback} from 'react'
function SubCounter({onClick, data}) {
  return (
    <button onClick={onClick}>{data.number}</button>
  )
}

SubCounter = memo(SubCounter)
export default function Counter() {
  const [name, setName] = useState('react')
  const [number, setNumber] = useState(0)
  const data = { number }
  const addClick = () => {
    setNumber(number+1)
  };
  return (
    <>
      <input type="text" value={name}
        onChange={(e) => setName(e.target.value)} />
      <SubCounter data={data} onClick={addClick} />
    </>
  )
}
```

+ 更深入的优化

    + useCallback：接收一个内联回调函数参数和一个依赖项数组，useCallback会返回该回调函数的memoized版本，该回调函数仅在某个依赖项改变时才会更新。
    + useMemo：把创建函数和依赖项数组作为参数传入useMemo，它仅会在某个依赖项改变时才重新计算memoized值，这种优化有助于避免在每次渲染时都进行高开销的计算

```javascript
import React, {useState, memo, useMemo, useCallback} from 'react'
function SubCounter({onClick, data}) {
  return (
    <button onClick={onClick}>{data.number}</button>
  )
}
SubCounter = memo(SubCounter)
let oldData, oldAddClick
export default function Counter() {
  const [name, setName] = useState('react')
  const [number, setNumber] = useState(0)
  const data = useMemo(() => ({number}), [number])
  oldData = data;
  const addClick = useCallback(() => {
    setNumber(number+1);
  }, [number])
  oldAddClick = addClick
  return (
    <>
      <input type="text" value={name}
        onChange={(e) => setName(e.target.value)} />
      <SubCounter data={data} onClick={addClick} />
    </>
  )
}
```



## 五、useReducer

+ useReducer 和 redux中的reducer很像
+ useState 的替代方案，它接收一个形如 (state, action) => newState 的 reducer，并返回当前的 state 以及与其配套的 dispatch 方法
+ 在某些场景下，useReducer 会比 useState 更适用，例如 state 逻辑较复杂且包含多个子值，或者下一个 state 依赖于之前的 state 等

```javascript
let initialState = 0
const [state, dispatch] = useReducer(reducer, initialState, init)
```

```javascript
const initialState = 0
function reducer(state, action) {
  switch(action.type) {
    case 'increment':
      return {number: state.number + 1};
    case 'decrement':
      return {number: state.number - 1};
    default:
      return state
  }
}

function init(initialState) {
  return {number: initialState}
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, initialState, init);
  return (
     <>
       Count: {state.number}
      <button onClick={() => dispatch({type: 'increment'})}>+</button>
      <button onClick={() => dispatch({type: 'decrement'})}>-</button>
    </>
  )
}
```



## 六、useContext

+ 接收一个context对象(React.createContext的返回值) 并返回该context的当前值
+ 当前context值由上层组件中距离当前组件最近的`<MyContext.Provider>`的value prop决定
+ 当组件上层最近的`<MyContext.Provider>`更新时，该Hook会触发重新渲染，并使用最近传递给MyContext provider的context value值
+ useContext(MyContext)相当于class组件中的`static contextType = MyContext` 或 `<MyContext.Consumer>`
+ `useContext(MyContext)`只是能够读取到context的值以及订阅context的变化，仍然需要在上层组件树种使用`<MyContext.Provider>`来为下层组件提供context

```javascript
import React, {
  useState, memo, useMemo, useCallback,
  useReducer, createContext, useContext
} from 'react'
import ReactDOM from 'react-dom'

const initialState = 0
function reducer(state=initialState, action) {
  switch(action.type) {
    case 'ADD':
      return {number: state.number + 1}
    default:
      break
  }
}

const CounterContext = createContext()

// first version
function SubCounter() {
  return (
    <CounterContext.Provider>
    { value => (
      <>
        <p>{value.state.number}</p>
        <button onClick={() => dispatch({type: 'ADD'})}>+</button>
      </>
    )}
    </CounterContext.Provider>
  )
}

// two version
function SubCounter() {
  const {state, dispatch} = useContext(CounterContext);
  return (
    <>
      <p>{state.number}</p>
      <button onClick={()=>dispatch({type: 'ADD'})}>+</button>
    </>
  )
}

function Counter() {
  const [state, dispatch] = useReducer((reducer), initialState,
    () => ({number: initialState}));
  return (
    <CounterContext.Provider value={{state, dispatch}}>
      <SubCounter />
    </CounterContext.Provider>
  )
}
ReactDOM.render(<Counter />, document.getElementById('root'))
```



## 七、useEffect

+ effect(副作用)：指那些没有发生在数据向视图转换过程中的逻辑，如 ajax 请求、访问原生dom 元素、本地持久化缓存、绑定/解绑事件、添加订阅、设置定时器、记录日志等。
+ 副作用操作可以分两类：需要清除的和不需要清除的
+ 原先在函数组件内改变dom，发送ajax请求以及执行其他包含副作用的操作是不被允许的，因为这可能破坏UI的一致性
+ useEffect接收一个函数，该函数会在组件渲染到屏幕之后才执行，该函数有要求：要么返回一个能清除副作用的函数，要么不返回任何内容
+ 与 `componentDidMount`和 `componentDidUpdate`不同，使用useEffect调度的effect不会阻塞浏览器更新屏幕，effect不需要同步的执行。

### 7.1 使用class组件实现修改标题

+ 在这个 class 中，我们需要在两个生命周期函数中编写重复的代码，这是因为很多情况下，我们希望在组件加载和更新时执行同样的操作。我们希望它在每次渲染之后执行，但 React 的 class 组件没有提供这样的方法。即使我们提取出一个方法，我们还是要在两个地方调用它。而 useEffect 会在第一次渲染之后和每次更新之后都会执行

```javascript
class Counter extends React.Component {
  componentDidMount() {
    this.changeTitle();
  }
  componentDidUpdate() {
    this.changeTitle();
  }
  changeTitle = () => {
    document.title = 'click title';
  }
  render() {
    return(
      <>...</>
    )
  }
}
```

### 7.2 使用useEffect实现修改标题

```javascript
import React, {useEffect} from 'react'

function Counter() {
  useEffect(() => {
    document.title = 'click title';
  })
  return (
    <></>
  )
}
```

### 7.3 清除副作用

+ 副作用函数还可以通过返回一个函数来指定如何清除副作用，为防止内存泄漏，清除函数会在组件卸载前执行。如果组件多次渲染，则在执行下一个effect之前，上一个effect就已经被清空

```javascript
function Counter() {
  let [number, setNumber] = useState(0)
  let [text, setText] = useState('')
  // 相当于 componentDidMount 和 componentDidUpdate
  useEffect(() => {
    let $timer = setInterval(() => {
      setNumber(number => number + 1);
    }, 1000)
  })
  return (
    <>
      <input value={text}
        onChange={(e) => setText(e.target.value)} />
      <p>{number}</p>
    </>
  )
}
```

### 7.4 跳过effect进行性能优化

+ 依赖项数组控制着useEffect的执行
+ 如果某些特定值在两次重渲染之间没有发生变化，可以通知React跳过对effect的调用，只要传递数组作为useEffect的第二个可选参数即可。
+ 如果想执行只运行一次的effect(仅在组件挂载和卸载时执行)，可以传递一个空数组作为第二个参数，这就告诉React你的effect不依赖于props或state中的任何值，所以它永远都不需要重复执行。

```javascript
function Counter() {
  let [number, setNumber] = useState(0)
  let [text, setText] = useS5tate('')
  useEffect(() => {
    let $timer = setInterval(() => {
      setNumber(number => number + 1)
    }, 1000)
  }, [text])
  return(
    <>
      <input value={text} onChange={(e) => setText(e.target.value)} />
      <p>{number}</p>
    </>
  )
}
```

### 7.4 使用多个Effect实现关注点分离

+ 使用Hook其中一个目的就是要解决class中生命周期函数经常包含不相关的逻辑，但又把相关逻辑分离到几个不同方法中的问题。

```javascript
class FriendStatusWithCounter extends React.Component {
  constructor(props) {
    super(props);
    this.state = { count: 0 };
  }
  componentDidMount() {
    document.title = `You clicked ${this.state.count} times`;
  }
  componentDidUpdate() {
    document.title = `You clicked ${this.state.count} times`;
  }
}
```

+ Hooks允许按照代码的用途分离它们，而不是像生命周期那样，react将按照effect声明的顺序依次调用组件中的每一个effect

```javascript
function FriendStatusWithCounter(props) {
  const [count, setCount] = useState(0);
  useEffect(() => {
    document.title = `You clicked ${count} times`;
  });
}
```



## 八、useLayoutEffect

+ useEffect在全部渲染完毕后才会执行
+ useLayoutEffect会在浏览器layout之后，painting之前执行。
+ 其函数签名与useEffect相同，当它会在所有DOM变更之后同步调用effect
+ 可以使用它来读取DOM布局并同步触发重渲染。
+ 在浏览器执行绘制之前useLayoutEffect内部的更新计划将同步刷新
+ 尽可能使用标准的useEffect以避免阻塞视图更新



## 九、useRef & useImperativeHandle

### 9.1 useRef

+ 类组件、React元素用React.createRef，函数组件用useRef
+ useRef返回一个可变的ref对象，其current属性被初始化为传入的参数(initalValue)
+ useRef 返回的 ref 对象在组件的整个生命周期内保持不变，也就是说每次重新渲染函数组件时，返回的 ref 对象都是同一个(使用React.createRef，每次重新渲染组件都会重新创建 ref)

```javascript
import React, {useState, useEffect, useRef} from 'react'
function Parent() {
  let [number, setNumber] = useState(0)
  return (
    <>
      <Child />
      <button onClick={() => setNumber({number: number + 1})}>+</button>
    </>
  )
}
function Child() {
  const inputRef = useRef()
  let input = inputRef;
  function getFocus() {
    inputRef.current.focus()
  }
  return (
    <>
      <input type="text" ref={inputRef} />
      <button onClick={getFocus}>get Focus</button>
    </>
  )
}
```

### 9.2 forwardRef

+ 因为函数组件没有实例，所以函数组件无法像类组件一样可以接收ref属性
+ forwardRef可以在父组件中操作子组件的ref对象
+ forwardRef可以将父组件中的ref对象转发到子组件中的DOM元素上
+ 子组件接受 props 和 ref 作为参数

```javascript
function Child(props, ref) {
  return (
    <input type="text" ref={ref} />
  )
}
Child = React.forwardRef(Child)

function Parent() {
  let [number, setNumber] = useState(0);
  const inputRef = useRef();
  function getFocus() {
    inputRef.current.value = 'focus';
    inputRef.current.focus();
  }
  return (
    <>
      <Child ref={inputRef} />
      <button onClick={()=>setNumber({number:number+1})}>+</button>
    </>
  )
}
```

### 9.3 useImperativeHandle

+ useImperativeHandle 可以在使用 ref 时，自定义暴露给父组件的实例值
+ 大多数情况下，应当避免使用ref这样的命令式代码，useImperativeHandle应当与forwardRef 一起使用
+ 父组件可以使用操作子组件中的多个ref

```javascript
import React, {
  useState, useEffect, createRef, useRef
  forwardRef, useImperativeHandle
} from 'react'

function Child(props, parentRef) {
  let focusRef = useRef()
  let inputRef = useRef()
  useImperativeHandle(parentRef, () => (
    return {
      focusRef, inputRef, name: 'react',
      focus() { focusRef.current.focus() },
      changeText(text) {
        inputRef.current.value = text
      }
    }
  ))
  return(
    <>
      <input ref={focusRef} />
      <input ref={inputRef} />
    </>
  )
}
Child = forwardRef(Child);

function Parent(){
  const parentRef = useRef();
  function getFocus(){
    parentRef.current.focus();
  }
  return (
    <>
      <Child ref={parentRef}/>
      <button onClick={getFocus}>获得焦点</button>
    </>
  )
}
```



## 十、自定义Hook

+ 有时会想在组件之间重用一些状态逻辑，之前采用的是render props，或者 高阶组件，或者 redux
+ 自定义Hook可以在不增加组件的情况下达成以上目的
+ Hook是一种复用状态逻辑的方式，它不复用state本身
+ Hook的每次调用都有一个完全独立的state

```javascript
import React, {useLayoutEffect, useEffect, useState} from 'react'
import ReactDOM from 'react-dom'

function useNumber() {
  let [number, setNumber] = useState(0)
  useEffect(() => {
    setInterval(() => {
      setNumber(number => number + 1)
    }, 1000)
  }, [])
  return [number, setNumber]
}

function Counter() {
  let [number, setNumber] = useNumber()
  return (
    <div>
      <button onClick={()=>{ setNumber(number+1)}}>
        {number}
      </button>
    </div>
  )
}
```



## 十一、问题补充

### 1. 使用 `eslint-plugin-react-hooks`来检查代码错误并提示

```
{
  'plugins': ['react-hooks'],
  'rules': {
    'react-hooks/rules-of-hooks': 'error', // 检查 Hook 的规则
    'react-hooks/exhaustive-deps': 'warn' // 检查 effect 的依赖
  }
}
```

### 2. 为什么必须在组件的顶层使用Hook & 在单个组件中使用多个 State Hook 或 Effect Hook

+ React依赖于 Hook的调用顺序，如果能确保Hook在每一次渲染中都按照同样的顺序被调用，那么React能够在多次的useState 和 useEffect调用之间保持hook状态的正确性

### 3. 在一个组件中多次调用 useState 或者 useEffect，每次调用 Hook，它都会获取独立的 state，是完全独立的。

### 4. useEffect 中调用用函数时，要把该函数在 useEffect 中申明，不能放到外部申明，然后再在 useEffect 中调用

### 5. 在Hooks中使用 fetch data

```javascript
import React, {useState, useEffect} from 'react'
import axios from 'axios'
function App () {
  const [data, setData] = useState({ hits: [] })
  useEffect(async () => {
    const result = await axios(apiUrl)
    setData(result.data)
  })
  return(
    <ul>
      {data.hits.map(item => (
        <li key={item.objectID}>
          <a href={item.url}>{item.title}</a>
        </li>
      ))}
    </ul>
  )
}
export default App;
```

### 6. 不要过度依赖useMemo

+ useMemo 本身也有开销，useMemo会记住一些值，同时在后续render时，将依赖数组中的值取出来和上一次记录的值进行比较，如果不相等才会重新执行回调函数，否则直接返回记住的值，这个过程本身就会消耗一定的内存和计算资源。


### 7. useEffect不能接收async作为回调函数

+ useEffect接收的函数，要么返回一个能清除副作用的函数，要么不返回任何内容，async返回的是promise