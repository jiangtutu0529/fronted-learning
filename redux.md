### Redux
##### redux
react:ui视图
redux:状态管理
react-redux：链接react和redux的桥梁，让react组件方便的读取store中的状态并向store分发action来更新状态。


核心原理是通过context API提供全局的Store, 并使用高阶组件或者hooks让组件和Store交互

1. Provider
   <Provider>是一个react组件，接收Redux Store作为prop,使用context API创建一个上下文，并挂在顶层节点，使得子组件可以获取到Store
2. Connect高阶组件
   connect是一个高阶函数，返回一个高阶组件，将Redux状态的Action绑定在React组件上
   原理：
   I. connect订阅Store状态
   II.Store变化时，通过mapStateToProps计算出组件需要用到的状态
   III.如果计算出的状态和上一次的不同，浅比较，就会重新渲染包裹的组件
   IV. 将dispatch方法通过mapDispatchToProps通过props注入到组件中
3. Hooks
   useSelector：接收一个selector函数，当Store的状态改变，useSelector会用新的状态重新执行这个函数，如果得出的结果与上一次的不同，严格比较，就会触发组件的重新渲染
   useDispatch：返回对于Redux Store的dispatch函数的引用

##### 如何使用useSelector避免重复渲染
1. 精细化选择：
   只选择组件真正需要的最小状态单元，不要返回整个状态树
2. 使用记忆化selector： 
   使用reselect库创建记忆化的selector函数，只有当依赖的输入状态变化时，才会去重新计算
3. 使用shallowEquall:
   如果 selector 必须返回一个新对象，可以将 react-redux中的 shallowEqual函数作为 useSelector的第二个参数，进行浅层比较。
#### redux中间件的原理
redux中间件通过对dispatch方法进行拦截和增强，允许在动作派发和到达reducer之间执行自定义逻辑
```
const loggerMiddleware = (store) => (next) => (action) => {
  console.log('Action:', action);
  let result = next(action); // 执行下一个中间件或 Reducer
  console.log('New State:', store.getState());
  return result;
};

```
使用中间件的流程:
1、创建中间件
2、使用applyMiddleware 将中间件添加到Store
3、中间件在每次dispatch时被调用
```
import { createStore, applyMiddleware } from 'redux';

const logger = (store) => (next) => (action) => {
  console.log('Dispatching:', action);
  let result = next(action);
  console.log('State after dispatch:', store.getState());
  return result;
};

const store = createStore(counterReducer, applyMiddleware(logger));

```

#### redux-saga执行原理
redux-saga是一个基于Generator的redux中间件，用来处理复杂的异步逻辑，通过take,call,put等来控制
核心流程：
1、通过take监听对应的action
2、使用call执行异步任务
3、使用put触发新的action
```
import { call, put, takeEvery } from 'redux-saga/effects';
import axios from 'axios';

function* fetchData(action) {
  try {
    const data = yield call(axios.get, '/api/data');
    yield put({ type: 'FETCH_SUCCESS', payload: data });
  } catch (e) {
    yield put({ type: 'FETCH_ERROR', message: e.message });
  }
}

function* mySaga() {
  yield takeEvery('FETCH_REQUEST', fetchData);
}

```

#### redux的核心概念
##### 核心概念
1、Store：存储应用状态
2、Action：描述状态变化的事件
3、Reducer：定义如何根据action来变化状态
4、Middleware：扩展dispatch的功能
5、单项数据流：状态流转的方向是固定的

设计思想：
状态管理的核心是单项数据流和数据不可变状态
状态只通过dispatch(action)触发reducer来更新

Reducer必须是纯函数，不应有副作用(触发异步操作或触发action)
