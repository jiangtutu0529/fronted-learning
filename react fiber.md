### Fiber
fiber本质上是一个js对象，代表react的一个工作单元，包含了组件的相关信息，一个简化的fiber结构：
```
{
  type: 'h1',  // 组件类型
  key: null,   // React key
  props: { ... }, // 输入的props
  state: { ... }, // 组件的state (如果是class组件或带有state的function组件)
  child: Fiber | null,  // 第一个子元素的Fiber
  sibling: Fiber | null,  // 下一个兄弟元素的Fiber
  return: Fiber | null,  // 父元素的Fiber
  // ...其他属性
}

```
当react工作时，会沿着Fiber树结构进行，对于每个fiber工作单元，会进行比较新旧props，确定是否需要更新组件等工作，当主线程有优先级高的任务，如响应用户输入，fiber会中断，当前工作，返回执行高优先级工作。
因此，fiber也是react的调度和更新机制的核心。

react16之前，以递归方式处理组件树更新，成为堆栈调和，一旦开始不能中断，直到整个树遍历完，这种机制在处理大量数据或者复杂视图时会阻塞主线程，导致无法及时响应用户输入而引起卡顿

react16之后引入fiber,是react定义的带有链接关系的DOM树，child、sibling 和 return 字段构成了Fiber之间的链接关系，使React能够遍历组件树并知道从哪里开始、继续或停止工作。

#### Fiber工作原理
1、单元工作：每个Fiber代表一个工作单元，所有Fiber节点组成一个Fiber链表树，有链接属性有树的结构，这种结构让react可以细粒度的控制节点行为
2、链接属性：child,sibing,return构成链接关系，使得react可以遍历树结构并且直接从哪里开始，继续或者停止工作
3、双缓冲技术：react更新时，有一个在页面上的树current tree,会根据现有的fiber树在缓存中创建一个新的缓存冲worlInProgress tree，WIP tree包含了更新受影响的最高节点和子节点，WIP tree通过diff更新为每个需要更新的节点打上标记，比如placement-插入，update-更新，deletion-删除。WIP更新完成后复制好其他节点，最终替换掉current tree。
4、state 和props：在fiber上记录了属性如：memoizedProps，memoizedState,pendingProps 字段，让react更新时知道组件的上一个状态和要更新的状态，通过比较，得出是否需要更新，从而避免不必呀的渲染。
5、副作用：flags 和 subtreeFlags 字段标识Fiber及其子树中需要执行的副作用，例如DOM更新、生命周期方法调用等。React会积累这些副作用，然后在Commit阶段一次性执行，从而提高效率。


#### Fiber工作流
##### 一阶段：Reconciliation
确定哪些节点需要更新

工作内容​​：React 会“虚拟地”构建一棵新的 Fiber 树（称为 WorkInProgress Tree），通过与当前的 Fiber 树（称为 Current Tree）进行 Diff 比较，找出所有需要更新的组件，并为每个需要更新的 Fiber 节点打上“副作用（Effect）”标记（比如 Placement-插入，Update-更新，Deletion-删除）。
​​关键特性​​：​​这个阶段的工作是异步的、可中断的​​。React 将整个树的遍历过程分解成一个个以 Fiber 节点为单位的“工作单元”。它处理完一个单元后，会检查主线程是否有更高优先级的任务（如用户输入）。如果有，就​​暂停​​当前工作，让出主线程。等浏览器空闲时，再恢复工作。这就是 ​​合作式调度（Cooperative Scheduling）​​。

##### 二阶段：Commit
​​工作内容​​：React 会​​同步地、一气呵成地​​遍历在阶段一中被标记了副作用的 Fiber 节点链表，并将更新应用到真实的 DOM 上。这个阶段会执行 componentDidMount、componentDidUpdate等生命周期函数。
​​关键特性​​：​​这个阶段的工作是同步的、不可中断的​​。因为 DOM 的变更必须连续完成，否则会给用户展示不一致的UI。

##### 如何实现可中断和恢复
1、​​链表数据结构​​：Fiber 树不再使用传统的树形递归结构，而是通过 child（第一个子节点）、sibling（下一个兄弟节点）、return（父节点）三个指针连接成的​​链表树​​。这使得 React 可以不用递归而使用循环来遍历树，从而可以在任何一个节点处暂停，并保留当前的遍历进度（即“游标”位置）。


2、​requestIdleCallback​​：Fiber 的调度理念是利用浏览器的空闲期。React 自己实现了一个功能更强大的 requestIdleCallbackpolyfill（称为 Scheduler），它会在浏览器空闲时分配时间片给 React 进行协调工作。如果时间片用尽，React 就会暂停工作，将控制权交还浏览器


将不可中断的同步递归更新，重构为可中断的异步增量更新。通过将任务分片、设置任务优先级和合作式调度，解决了主线程阻塞问题，为 Concurrent Mode（并发模式）打下了基础

#### react实现中断和恢复
依赖于Fiber，链表数据结构，优先级调度


react 通过 Fiber 架构将渲染任务拆解为多个小工作单元（每个 Fiber 节点是一个单元），
利用链表遍历代替递归，使得任务可以在处理完任意节点后中断。
中断时，React 保存当前处理的 Fiber 节点（游标）和已构建的部分树（workInProgress）。
通过调度器（Scheduler）和优先级机制，在浏览器空闲时恢复任务，从上次中断的节点继续处理。
整个过程依赖双缓存技术保证状态一致性，且仅在协调阶段允许中断，提交阶段必须同步完成以确保 UI 一致性.

React 在更新时实现​​中断和恢复​​的核心机制依赖于 ​​Fiber 架构​​、​​链表数据结构​​和​​优先级调度​​。以下是x详细的实现原理：
一、如何实现中断？
React 的协调过程（Reconciliation）被拆分为多个​​可中断的工作单元​​，每个单元对应一个 Fiber 节点。中断的实现依赖于以下关键设计：
1. ​​链表遍历代替递归​​
​​传统递归的问题​​：递归调用栈是内存中的连续调用帧，一旦开始就无法中途暂停（必须一口气执行完整个调用栈）。
​​Fiber 的解决方案​​：将组件树转换为一个​​单向链表结构​​（通过 child、sibling、return指针连接），通过循环（while）而非递归遍历链表。
每次处理一个 Fiber 节点后，React 会检查剩余时间片（通过 requestIdleCallback或 Scheduler 实现）。
如果时间不足，React 保存当前遍历到的节点（即“游标”），并主动中断循环。
2. ​​浏览器空闲时间调度​​
React 使用调度器（Scheduler）模拟 requestIdleCallback，在浏览器空闲时执行任务：
// 伪代码：React 的任务调度逻辑
function workLoop(deadline) {
  while (currentFiber && deadline.timeRemaining() > 0) {
    currentFiber = performUnitOfWork(currentFiber); // 处理一个Fiber节点
  }
  if (currentFiber) {
    // 时间不够，中断并请求下次调度
    requestIdleCallback(workLoop);
  }
}
​​中断条件​​：当 deadline.timeRemaining()表示当前帧剩余时间不足时，React 暂停工作。
3. ​​优先级机制​​
React 为不同更新分配优先级（如用户交互 > 动画 > 数据加载）。高优先级任务可以中断低优先级任务：
当高优先级任务到达时，React 会中断当前低优先级任务，先执行高优先级任务。
被中断的任务会被标记为未完成，后续重新调度。
二、如何实现恢复？
中断后恢复的关键是​​保存进度​​和​​重新调度​​：
1. ​​保存当前进度​​
Fiber 节点中保留了遍历所需的上下文信息：
currentFiber：当前处理到的 Fiber 节点（相当于“游标”）。
workInProgressTree：已构建的部分新 Fiber 树。
即使任务被中断，React 也能知道“接下来该处理哪个节点”。
2. ​​重新调度​​
被中断的任务会被放入一个​​任务队列​​中。
当浏览器空闲时（或高优先级任务完成后），React 重新从上次中断的 Fiber 节点继续处理。
3. ​​双缓存技术​​
React 始终维护两棵 Fiber 树：
current：当前渲染的树（对应屏幕上显示的UI）。
workInProgress：正在构建的新树（在内存中）。
中断时，workInProgress树会保留部分完成的更新，恢复时直接继续构建。