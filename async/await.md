##### async/await
async/await实际上是generator+promise的语法糖
```
// async/await 实际上是 Generator + Promise 的语法糖
async function example() {
  const result = await someAsyncFunction();
  console.log(result);
}

// 相当于：
function example() {
  return new Promise((resolve, reject) => {
    someAsyncFunction().then(result => {
      console.log(result);
      resolve();
    }).catch(reject);
  });
}

// 更精确的转换：
async function complexExample() {
  console.log('开始');
  const a = await getValueA();
  console.log('得到a:', a);
  const b = await getValueB(a);
  console.log('得到b:', b);
  return a + b;
}

// 转换为Promise链：
function complexExampleTransformed() {
  return new Promise((resolve, reject) => {
    console.log('开始');
    
    getValueA().then(a => {
      console.log('得到a:', a);
      return getValueB(a);
    }).then(b => {
      console.log('得到b:', b);
      resolve(a + b); // 注意：这里的a需要闭包处理
    }).catch(reject);
  });
}
```

##### await和promise.then哪个先执行？
答案：
执行顺序取决于它们在微任务队列中的位置
先添加到微任务队列的回调先执行
await后面的代码本质上也是promise.then回调
在同一个Promise上，先注册的.then先于await后面的代码执行

##### async/await 
Promise的语法糖，同步的代码风格，异步的执行效果
async的本质返回一个promise

async function test(){
  creturn 42
}
function test(){
  return new Promise((resolve,reject)=>{
    resolve(42)
  })
}

```
// await 的伪代码实现
function _await(value, then) {
  // 如果 value 是 Promise，等待其解决
  if (value && typeof value.then === "function") {
    return value.then(then);
  }
  // 否则直接返回值
  return then(value);
}

// async 函数被转换为生成器 + 自动执行器
```