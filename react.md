### react
react是用于构建界面的js库，提供了UI层面的解决方案，使用虚拟DOM来高效的操作DOM,遵循从高阶组件到低阶组件的单项数据流，将每个页面组成单独的小块，组件之间可以组合，嵌套，组装成整体页面。
特性：
JSX语法
单项数据绑定
虚拟DOM
声明式编程
组件化(Component)


##### JSX
javascript XML, 是一种定义带属性树结构的语法，是js描述UI结构的语法扩展，可以实现在js中描述html，并被编译器(如babel)最终通过React.createElement()转换成标准的js代码。
```
// JSX
<div id="root">Click</div>

// 编译后
React.createElement("div", { id: "root" }, "Click");
```
特性：
1、组件名首字母大写
2、可嵌套，在render函数的返回中只能有一个根节点
3、驼峰写法的属性，如className,onClick
4、求值表达式，在遇到html<>用html解析，在遇到{}用js解析
5、jsx在模板插入js变量，如果变量是一个数组，则会展开这个数组的所有成员
6、ReactDom在渲染前，会转义JSX里面的值，渲染前，会将所有的值都转化成字符串，可以预防xss攻击

##### 虚拟DOM
Real DOM:
1、真实DOM，文档对象模型，结构化文本，页面上每一个真实渲染出来的结点都是一个真实的DOM结构
2、更新缓慢
3、可以直接更新HTML
4、DOM操作代价高，DOM有更新直接创建新DOM
Virtual DOM 
真实DOM的抽象表达，轻量级的js对象，减少直接操作真实 DOM 的次数来提升性能
1、以js对象结构的形式存储对DOM的描述，虚拟DOM对象的节点和真实DOM的属性一一对应
2、更新更快
3、无法直接更新html
4、DOM操作简单，元素更新，更新JSX

##### 虚拟DOM的工作流：
初次渲染：
JSX->虚拟DOM->生成真实DOM
状态更新：
重新生成新的虚拟DOM->对比新旧虚拟DOM(diff)->找出差异(Reconciliation)->将差异部分更新到真实DOM(commit)

##### Diff算法
diff通过最小化DOM操作更新界面，不直接重新渲染，基于两个假设：
1、相同类型的节点有相似的结构，不同类型的节点生成不同的DOM结构
2、同一层级节点通过唯一的key值进行区分，从而优化列表渲染
3、分层比较
I、不跨层比对，只比较同一个层级的节点
II、如果节点类型不同，直接销毁旧节点以及子树，创建新节点
```
// 旧虚拟 DOM
<div>
  <Counter />
</div>

// 新虚拟 DOM
<span>
  <Counter />
</span>
```
III、组件类型判断
组件类型相同，会保留组件实例，仅仅更新props出发更新渲染，组件类型不同，卸载旧组件，挂载新组件
```
// 旧虚拟 DOM
<Counter count={1} />

// 新虚拟 DOM
<Counter count={2} />  // 仅更新 props，不重新挂载
```
IV、列表用key值优化渲染
渲染动态列表时，需要一个唯一的值来判定哪些元素可以复用，而不是全部重新渲染
key应该时稳定的唯一的
如果key相同，尝试复用DOM节点，更新变化的部分
key不同，销毁旧节点，创建新节点
```
// 旧列表
<ul>
  <li key="1">A</li>
  <li key="2">B</li>
</ul>

// 新列表
<ul>
  <li key="2">B</li>  // key="2" 相同，仅移动位置
  <li key="1">A</li>  // key="1" 相同，仅移动位置
</ul>
```

React 会复用 li元素，仅调整它们的顺序，而不是重新创建​​

注意：如果用index做key值，
对于交换位置的变化，会认为key值变了而重新渲染
缺点：
1、无法在不同层级之间移动节点
2、列表渲染依赖key值
3、无法完全避免DOM操作，只能减少，无法完全消除

##### 批量更新
一种优化更新的机制，将多个状态的更新合并为一次渲染，减少不必要的计算和更新，从而提升性能
将多个状态更新合并成一次更新周期渲染一次
1、在React合成事件中，如onClick,onChange，所有状态更新会被自动批量处理
```
function App() {
  const [count, setCount] = useState(0);
  const [flag, setFlag] = useState(false);

  const handleClick = () => {
    setCount(c => c + 1); // 更新 1
    setFlag(f => !f);     // 更新 2
    // React 会将两次更新合并，只触发一次渲染
  };

  return <button onClick={handleClick}>Click</button>;
}
```
2、在setTimeout,Promise和原生事件或者异步上下文中，React17之前的版本不会自动批量更新，React18后改进
```
const handleClick = () => {
  setTimeout(() => {
    setCount(c => c + 1); // 更新 1
    setFlag(f => !f);     // 更新 2
    // React 17 及之前：触发两次渲染
    // React 18：自动批量更新（一次渲染）
  }, 1000);
};
```
3、React18之后并发模式自动批量更新
React 18 引入了 ​​并发模式（Concurrent Mode）​​，对所有场景（包括异步代码）默认启用批量更新：
```
// React 18 中，以下代码只会触发一次渲染：
fetchData().then(() => {
  setCount(c => c + 1);
  setFlag(f => !f);
});
```
4、强制立即更新
​​ReactDOM.flushSync()​​（React 18）
```
import { flushSync } from 'react-dom';

flushSync(() => {
  setCount(1); // 立即更新
});
```
5、批量更新的底层：更新队列
I、收集更新：setState时更新放入队列
II、调度渲染：React在合适的时机，统一处理更新队列中的所有更新
III、合并更新：对同一状态的多次更新合并成一次，多次setCount只保留最终结果

##### 类组件和函数组件的区别
是两种不同的代码编写方式
###### 类组件 class Component
1、基于es6的class语法，己成React.Component
2、必须有一个render方法，返回JSX
3、使用this来访问props和state
4、使用this.state来访问状态，并且用this.setState()来设置状态
```
Class Counter extends React.Component {
  constructor(props){
    super(props)
    this.state = {
      count = 0
    }
  }
  add(){
    this.setState({count:this.state.count+1})
  }
  render(){
    return <div>{this.state.count}</div>
  }
}
```
5、生命周期
componentDidMount()
componentDidUpdate()
componentWillUnMount()
```
class Example extends React.Component {
  componentDidMount() {
    console.log('组件已挂载');
  }
  componentWillUnmount() {
    console.log('组件即将卸载');
  }
  render() { return <div>Lifecycle Example</div>; }
}
```
6、使用React.PureComponent和shouldComponentUpdate来避免不必要的渲染
###### 函数组件 Function Component
1、纯js函数，直接返回JSX
2、通过接收一个参数props
3、更简介，适合无状态UI
4、使用hooks如useState来管理状态
```
function Counter(){
  const [count,setCount] = useState(0)
  const onClick = ()=>{
    setCount(count+1)
  }
  return <div onClick={onClick}>{count}</div>
}
```
5、使用hooks的useEffect来替代生命周期
```
import { useEffect } from 'react';

function Example() {
  useEffect(() => {
    console.log('组件已挂载');
    return () => console.log('组件即将卸载');
  }, []); // 空依赖数组表示仅在挂载/卸载时执行
  return <div>Lifecycle Example</div>;
}
```
6、使用React.memo来包裹，必要重复渲染

##### react为什么是单项数据流
单向数据流：数据从父组件通过props流向子组件
1、所有数据来源都源于父组件，避免子组件随意修改数据导致混乱，单项数据流更好追踪数据变化
2、子组件不可以修改父组件状态，子组件只需要负责渲染和更新事件，子组件是无状态的可以更好的复用
3、React 通过虚拟 DOM 和 Diff 算法优化渲染，单向数据流能更精准定位需要更新的组件，可以精确知道哪些组件需要更新
4、纯函数组件更好测试

##### 子组件如何修改父组件数据
1、父组件将修改的函数当作props传递给子组件，子组件调用修改函数
2、使用context，将状态用context管理，子组件可以修改状态
3、将状态提升到共同的父组件，然后将修改数据传递

##### react事件机制
React基于浏览器的事件机制自己实现了一套事件机制，包括事件注册，事件合成，事件冒泡，事件派发等，React称这套机制为React合成事件
合成事件是React模拟原生浏览器DOM事件所有能力的一个事件对象，即浏览器原生事件的包装器，react17前将事件处理器绑定在document上，在react18之后，绑定到rootDOM容器上
合成事件的执行顺序：
React所有事件都挂在document上
当真实DOM事件触发时，会冒泡到document对象上，此时处理React事件

合成事件与原生事件区别：
1、驼峰命名
2、阻止默认行为：必须显式调用e.preventDefault()，不能返回false
3、停止传播：使用e.stopPropagation()，针对react事件的

##### 受控组件和非受控组件
受控组件：表单数据由react组件管理，表单输入的值由react的state控制
1、表单数据存储在组件的state中
2、通过value值绑定到表单元素
3、通过onChange事件处理器更新state
4、react成为表单数据的“单一数据源”
非受控组件：表单数据由DOM自身管理，React不控制表单的值
1、表单数据存储在DOM中
2、使用ref来获取表单的值
3、接近传统html表单行为
4、适合简单表单或集成非React代码

##### ref
Ref是react直接操作DOM的方式，直接访问元素或者DOM实例
```
import { useRef } from 'react';

function MyComponent() {
  const inputRef = useRef(null);

  const focusInput = () => {
    inputRef.current.focus();
  };

  return (
    <div>
      <input ref={inputRef} type="text" />
      <button onClick={focusInput}>Focus Input</button>
    </div>
  );
}
```
特点：
1、持久性：在整个组件生命周期中useRef返回的对象保持不变
2、不会触发重新渲染：修改ref.current不会触发组件重新渲染
3、访问DOM节点：直接访问DOM节点
4、存储可变值：可以用来存储任何可变值，类似于类组件的实例属性

在父组件需要访问子组件的节点，父节点传递ref给子组件，子组件使用React.forwardRef包裹，承接ref参数


##### react18新特性
1. 并发特性，自动批处理
   ```
   // React 17 及之前：只在React事件处理函数中批处理
function handleClick() {
  setCount(c => c + 1);     // 重新渲染
  setFlag(f => !f);         // 重新渲染
  // 结果：两次渲染
}

// React 18：在所有函数中自动批处理
function handleClick() {
  setCount(c => c + 1);     // 不立即渲染
  setFlag(f => !f);         // 不立即渲染
  // 结果：一次批处理渲染
}

// 异步操作中也批处理
setTimeout(() => {
  setCount(c => c + 1);
  setFlag(f => !f);
  // React 18：批处理，只渲染一次
}, 1000);

// 如果需要立即更新，使用flushSync
import { flushSync } from 'react-dom';

flushSync(() => {
  setCount(c => c + 1);     // 立即渲染
});
flushSync(() => {
  setFlag(f => !f);         // 立即渲染
});
   ```

2. 新的根API
```
// React 17 及之前
import ReactDOM from 'react-dom';
import App from './App';

ReactDOM.render(<App />, document.getElementById('root'));

// React 18 新API
import { createRoot } from 'react-dom/client';
import App from './App';

const container = document.getElementById('root');
const root = createRoot(container);
root.render(<App />);

// 支持并发特性
root.render(
  <React.unstable_ConcurrentMode>
    <App />
  </React.unstable_ConcurrentMode>
);
```

3. 新hooks
useTransition - 非阻塞UI更新
```
import { useState, useTransition, Suspense } from 'react';

function SearchResults({ query }) {
  const [isPending, startTransition] = useTransition();
  const [results, setResults] = useState([]);
  
  function handleSearch(query) {
    startTransition(() => {
      // 非紧急更新：可以被打断
      setResults(fetchResults(query));
    });
  }
  
  return (
    <div>
      <input 
        onChange={(e) => handleSearch(e.target.value)}
        placeholder="搜索..."
      />
      
      {/* 显示加载状态 */}
      {isPending && <div>加载中...</div>}
      
      <Suspense fallback={<div>加载结果...</div>}>
        <ResultsList results={results} />
      </Suspense>
    </div>
  );
}
```

4. 服务端渲染新的API
```
import { Suspense } from 'react';
import { createFromFetch } from 'react-server';
```


##### react和react-dom的区别
```
// React包只包含核心逻辑，不包含DOM操作
import React from 'react';

// React核心功能：
// 1. 组件定义
class MyComponent extends React.Component {}
function MyFunctionalComponent() {}

// 2. Hooks系统
const [state, setState] = React.useState();
React.useEffect(() => {});

// 3. 虚拟DOM创建
const element = React.createElement('div', { className: 'test' }, 'Hello');
const jsxElement = <div className="test">Hello</div>;
```

```
// React DOM负责将React组件渲染到浏览器DOM
import ReactDOM from 'react-dom/client';

// React DOM主要功能：
// 1. DOM渲染
const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(<App />);

// 2. DOM事件处理
// 3. DOM属性更新
// 4. 服务端渲染
```

1. requestAnimationFrame
作用
requestAnimationFrame要求浏览器在下一次重绘之前执行指定的回调函数。这是实现流畅动画和连续更新的标准方式。
执行时机
一帧的生命周期：
[处理输入事件] → [执行 rAF 回调] → [计算样式/布局/绘制] → [页面渲染]
                      ↑
              在这里执行，适合视觉更新

2. requestIdleCallback
作用
requestIdleCallback在浏览器空闲时期执行回调函数，允许开发者在主事件循环上执行后台和低优先级工作，而不会影响延迟关键事件。
执行时机
一帧的生命周期：
[处理任务] → [渲染工作] → [有空闲时间？] → [执行 rIC 回调]
                                          ↑
                                   只有这时才执行

##### React和requestIdleCallback
React 并没有直接使用或重写 requestIdleCallback，而是基于相似的空闲时间调度理念，自己实现了一个功能更强大的调度器。"
"主要原因有四点：
更精细的优先级控制：React 需要多级优先级，而原生 API 只有'空闲'和'非空闲'两种状态
更稳定的执行频率：原生 API 的执行频率不足以支持流畅的60fps动画
更好的超时控制：React 需要确保高优先级任务准时执行
一致的跨浏览器行为：避免不同浏览器的实现差异"
"这个自定义调度器是 React 并发模式（Concurrent Mode）的基石，使得特性如 useTransition、Suspense等能够实现'可中断的渲染'，从根本上改善了大型应用的用户体验。"