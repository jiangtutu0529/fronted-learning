### React Hooks
react16.8之后引入，在函数中使用state以及其他reac特性，无需写class
class痛点：
1. 状态逻辑难以复用
2. 复杂组件难以理解
3. class复杂

函数组件使用函数式编程

##### useState
useState不会合并对象，可以函数式更新和直接更新
```
import React, { useState } from 'react';

function Example() {
  // 声明一个叫 "count" 的 state 变量，初始值为 0
  // setCount 是更新 count 的函数
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>You clicked {count} times</p>
      {/* 函数式更新：如果新 state 依赖于之前的 state */}
      <button onClick={() => setCount(prevCount => prevCount + 1)}>
        Click me
      </button>
      {/* 直接更新 */}
      <button onClick={() => setCount(100)}>
        Reset to 100
      </button>
    </div>
  );
}
```
##### useEffect
在函数组件中执行副作用，相当于componentDidMount,componentDidUpdate,componentWillUnmount的组合
```
import React, { useState, useEffect } from 'react';

function Example() {
  const [data, setData] = useState(null);

  // 1. 无需清除的 effect（在每次渲染后都会执行）
  useEffect(() => {
    document.title = `You clicked ${count} times`;
  }); // 没有依赖数组，每次更新都执行

  // 2. 需要清除的 effect（如订阅）
  useEffect(() => {
    const subscription = props.source.subscribe();
    // 返回一个清除函数，在组件卸载前和执行下一个 effect 前调用
    return () => {
      subscription.unsubscribe();
    };
  }, [props.source]); // 依赖数组：只有当 props.source 改变时，effect 才会重新执行

  // 3. 只在挂载时执行一次的 effect
  useEffect(() => {
    fetchData().then(setData);
  }, []); // 空依赖数组：effect 不依赖于任何 props 或 state，所以只运行一次

  return <div>{/* ... */}</div>;
}
```
依赖数组（Dependency Array）​​ 是 useEffect的灵魂，它决定了 effect 在何时运行

##### useContext
在组件树中订阅状态，避免多层props传递
```
const ThemeContext = React.createContext('light');

function App() {
  return (
    <ThemeContext.Provider value="dark">
      <Toolbar />
    </ThemeContext.Provider>
  );
}

function Toolbar() {
  // 直接获取 Context 的值
  const theme = useContext(ThemeContext);
  return <div style={{ background: theme === 'dark' ? '#333' : '#FFF' }}>Current theme: {theme}</div>;
}
```
##### useReducer
useState的替代方案，适用复杂的state
##### useCallback
返回一个记忆化的回调函数，避免非必要的重渲染，常用于优化子组件。
##### useMemo
返回一个记忆化的值，避免每次渲染都进行高开销计算。
##### useRef
返回一个可变的 ref 对象，其 .current属性被初始化为传入的参数。常用于直接访问 DOM 节点或存储任意可变值
##### 自定义hook
use开头的js函数，可以提取重复的状态逻辑


#### 为什么不能条件调用hooks
react内部维护单向链表，按照hooks的调用顺序存储每个hook的数据，第一次渲染，将hook按顺序放入链表，后续渲染，严格按照第一次的调用顺序来读取对应的状态，如果使用条件判断，会打乱hook的执行顺序，导致状态错乱

#### useEffect和useLayoutEffect的区别
useEffect：在DOM更新后，浏览器绘制后，异步执行，不会阻塞浏览器渲染，适用数据获取，事件监听
useLayoutEffect：在DOM更新后，浏览器绘制前，同步执行，在浏览器绘制前执行，会阻塞浏览器渲染，适用同步修改DOM，动画起始状态修改，避免闪烁


#### 如何解决闭包陷阱
```
function Counter() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    const timer = setInterval(() => {
      // 闭包问题：始终拿到初始的 count 值
      setCount(count + 1);
    }, 1000);
    return () => clearInterval(timer);
  }, []); // 依赖数组为空
  
  return <div>{count}</div>;
}
```
解决方案：
```
// 方案1：使用函数式更新
setCount(c => c + 1);

// 方案2：正确添加依赖
useEffect(() => {
  const timer = setInterval(() => {
    setCount(count + 1);
  }, 1000);
  return () => clearInterval(timer);
}, [count]); // 添加 count 依赖
```

#### 实现一个防抖hook
```
function useDounce(value,time){
  const [newValue,setValue] = useState()
  useEffect(()=>{
    const timer = setTimeout(()=>{
      setNewValue(value)
    },time)
    return ()=>{
      clearTimeout(timer)
    }
  },[value])
  return newValue
}
```
#### 如何获取上一轮的props和state
useRef存储
```
function usePrevious(value){
  const ref = useRef()
  useEffect(()=>{
    ref.current= value
  },[value])
  return ref.current
}
```

##### context的核心原理
```
// Context创建
const MyContext = React.createContext(defaultValue);

// Context包含两个重要组件
<MyContext.Provider value={/* 某个值 */}>
  <MyContext.Consumer>
    {value => /* 基于上下文值进行渲染*/}
  </MyContext.Consumer>
</MyContext.Provider>
```
  

1. Context的实现原理是什么？
答案：Context通过React内部的Fiber树维护一个值栈。Provider组件将值存储在Context对象中，Consumer组件或useContext Hook通过遍历Fiber树找到最近的Provider来获取值。
2. 为什么Context的value变化会导致所有消费者重新渲染？
答案：当Provider的value变化时，React会遍历所有依赖此Context的组件，并标记它们需要更新。这是通过Fiber树的依赖追踪机制实现的。


##### useEffect直接用async?
不能，因为useEffect期待返回一个清理函数或者undefined,但是async返回了一个promise,违反了类型约定。
解决：
在useEffect内部定义和调用异步函数，
使用立即执行函数