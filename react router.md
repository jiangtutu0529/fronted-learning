### react router
react router在spa单页面中管理导航和渲染不同的组件，使得应用看起来像一个多页面，但无需每次跳转页面都向服务端请求html，从而提供更加流畅的页面体验。

react router v5和v6版本，目前基于v6

BrowserRouter：使用historyAPI保持UI和url同步
HashRouter：使用URL的hash部分实现路由，常用于不支持historyAPI的静态文件服务

Routes: 取代V5的Switch，是所有子路由的父容器，负责遍历子Route元素，找到与当前URL匹配最好的进行渲染
Route：定义子路由，path路径模式，element取代V5的component和render，为当前路径渲染的组件

NavLink：一种特殊的Link,当to属性与当前url匹配，可以为其应用自定义样式，适合导航菜单
Link：相当于一个不会引起页面刷新的a标签，用于跳转到新位置，to属性为跳转路径


##### hooks
useParams：获取动态参数
useLocation:返回当前location对象，  包含pathname,search,hash,state等信息
useSearchParams:用户读取和修改URL中的查询字符串
useNavigate:编程式导航
useMatch：返回给定路径模式相对于当前 URL 的匹配信息。


##### 原理：基于客户端路由的条件渲染
1、监听路由变化：路由器比如BrowserRouter会在挂载时监听history的popstate事件，当用户点击浏览器前进或者后退，或者通过history.pushState，history.replaceState改变URL时，触发事件
2、路径匹配：URL改变后，<Routes>组件将内部所有的<Route>的path字段和window.locatipn.pathname进行匹配
3、状态更新与渲染：一旦匹配到对应的Route，React Router更新内部的状态，包括当前的location信息，通过contextApi传递，
4、条件渲染：匹配到的<Route>渲染element属性指定的组件

##### 关键实现
history：react router内部使用history库，该库封装了不同环境下的历史记录管理，提供统一的API
Context API:react router在根部挂在context API即BrowserRouter向下传递路由状态(当前的location和history对象)


BrowserRouter:
```
const RouterContext = React.createContext()
function BrowserRoter((children)){
  const [location,setLocation] = useState(window.history.location)

  useEffect(()=>{
    const handlePopState = ()=>{
      setLocation(window.location.pathname)
    }
    window.addEventListener('popstate',handlePopState)  
    return ()=>{
      window.removeEventListener('popstate',handlePopState)
    }
  },[])
  const history = {
    push:(path)=>{
      window.history.push({},'',path)
      setLocation(path)
    }
  }
  const value={
    location,
    history
  }
  return <RouterContext.Provider value={value}>
  {children}
  </RouterContext.Provider>
}

function Routes({children}){
  const routerContext = useContext(RouterContext)
  let match = null
  React.children.forEach(children,(child)=>{
    if(!match&&React.isVaildElement(child)&&child.type === Route){
      const {path} = child
      if(path === location){
        match = child
      }
    }
  })
  return match?React.cloneElement(match,{componentMatch:match}):null
}

function Route({path,element}{
  return element
})

function Link({to,children}){
  const {history} = useContext(RouterContext)
  const handleClick = (e)=>{
    e.preventDefault()
    history.push(to)
  }
  return <a href={to} onClick={handleClick}>{children}</a>
}
```

##### react router如何实现路由守卫
在子路由组件<route>外层封装一层组件，用来判断，然后将封装的组件传递给element
```
function PrivateRoute({ children }) {
  const isAuthenticated = // ... 你的认证逻辑
  const location = useLocation();

  return isAuthenticated ? (
    children
  ) : (
    <Navigate to="/login" state={{ from: location }} replace />
  );
}

// 使用
<Route
  path="/dashboard"
  element={
    <PrivateRoute>
      <Dashboard />
    </PrivateRoute>
  }
/>
```

##### ​​React Router 是如何实现路由切换不刷新页面的？​
react router通过拦截浏览器的默认导航行为，使用html history的API来修改URL，并通过react的状态更新来触发组件的重新渲染，从而实现页面无刷新的视图切换。

详细原理分解（三步走）
我们可以将整个过程分解为三个关键环节：
第1步：拦截导航 - 阻止默认刷新行为
当你在应用中点击一个 <Link>组件时，背后其实是一个普通的 <a>标签。
// 你写的代码
<Link to="/about">关于我们</Link>

// 渲染到DOM后大致相当于
<a href="/about">关于我们</a>
如果没有干预，点击这个链接会向服务器请求 /about页面，导致整个页面刷新。
​​React Router 做的事情是：​​
<Link>组件为这个 <a>标签添加了一个 onClick事件监听器。
当你点击链接时，这个监听器会：
​​调用 event.preventDefault()​​：阻止浏览器默认的跳转和刷新行为。
​​执行自己的导航逻辑​​：也就是接下来的第2步。
第2步：改变URL - 使用 History API 无刷新更新地址栏
阻止默认行为后，需要一种方法在不刷新页面的情况下改变浏览器地址栏的 URL。这就是 ​​HTML5 History API​​ 的用武之地。
History API 提供了两个关键方法：
window.history.pushState(state, title, url): 向浏览器历史记录堆栈​​添加​​一个新记录，同时改变当前 URL。
window.history.replaceState(state, title, url): ​​替换​​当前的历史记录，不添加新记录。
​​React Router 的执行过程：​​
当点击 <Link>或调用 navigate(‘/about’)时，React Router 在内部调用 history.pushState（或 replaceState）。
// 内部类似这样的调用
window.history.pushState({}, "", "/about");
这行代码执行后，​​浏览器的地址栏会立刻变成 www.yoursite.com/about，但页面完全不会刷新​​。
第3步：触发渲染 - 通知 React 更新视图
仅仅改变 URL 是不够的，还需要让 React 组件知道该显示什么。这是最精妙的一步。
​​1. 监听 URL 变化：​​
React Router 的根组件（如 <BrowserRouter>）在初始化时，会监听 popstate事件。
popstate事件会在用户点击浏览器​​前进/后退​​按钮时触发。
​​注意​​：直接调用 history.pushState​​不会​​触发 popstate事件。所以，当通过 <Link>或 navigate导航时，需要手动通知路由系统。
​​2. 内部状态管理（核心）：​​
React Router 在它的顶层维护着一个内部状态（例如 location对象），这个状态保存了当前的路径信息。
当导航发生时（无论是通过 pushState还是点击了前进/后退），React Router 会​​更新这个内部的 location状态​​。
​​3. 上下文（Context）传递与重新渲染：​​
这个 location状态是通过 React 的 ​​Context API​​ 从根组件向下传递的。
当 location状态发生变化时，所有消费了这个 Context 的组件（例如 <Routes>, useLocation等）都会收到通知。
​​<Routes>组件的工作​​：它会遍历其所有的子 <Route>元素，将它们的 path属性与新的 location.pathname进行匹配。
​​匹配与渲染​​：一旦找到匹配的 <Route>，<Routes>就会渲染其 element属性所对应的 React 组件。不匹配的组件则不会被渲染。


##### hash模式和history模式区别
监听hash值的变化
监听popstate的变化