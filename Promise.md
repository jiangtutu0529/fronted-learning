### Promise
用于处理异步操作的对象，代表一个异步操作的最终完成或者失败的返回结果，Promise之前异步操作依赖回调函数，容易形成'回调地狱',Promise更优雅的管理异步。


一个Promise有三种状态：
pending:待定状态，还没有完成
fulfilled:已兑现
rejected:已失败
状态一旦改变，不可逆
pending->fulfilled
pending->rejected

创建Promise:new Promise(),接受一个函数作为参数，函数本身有两个函数参数(resolve,reject)
resolve(value)：成功状态，Promise的状态由pending转变为fulfilled，并且将结果返回到.then的参数
reject(reason)：失败状态，Promise的状态由pending转变为rejected，并且将失败原因返回打.catch的参数或者.then的第二个参数

const myPromise = new Promise((resolve,reject)=>{
  setTimeout(()=>{
    const random = Math.Random()
    if(random<0.5){
      resolve(random)
    }else{
      reject(random)
    }
  },1000)
})
myPromise().then(res=>{
  console.log(res)
}).catch(error=>{
  console.error(error)
}).finally(()=>{
  console.log('finally')
})

Promise链，.then总是返回一个新的Promise，可以让下一步的.then处理
function async1(){
  return new Promise((resolve,reject)=>{
    resolve('1111')
  })
}
function async2(){
  return new Promise((resolve,reject)=>{
    resolve('2222')
  })
}
async1().then(res=>{
  console.log(res) //'111'
  return async2()
}).then(res=>{
  console.log(res) //'222'
  return '333'
}).then(res=>{
  console.log(res) //'333'
})

#### Promise.all
接收一个数组，当所有数组都成功返回，依次将成功的结果放在结果数组里面返回，当有一个失败时，返回失败，失败原因为第一个失败的原因
```
const p1 = Promise.resolve(3);
const p2 = 42; // 非Promise值会被Promise.resolve包装
const p3 = new Promise((resolve) => setTimeout(resolve, 100, 'foo'));

Promise.all([p1, p2, p3])
  .then((values) => console.log(values)) // 约100ms后输出: [3, 42, "foo"]
  .catch((error) => console.error(error));
```

#### Promise.allSettled
接收一个数组，返回数组每一项的结果，如论成功或者失败，不关心成功或者失败，只全部完成
```
const p1 = Promise.resolve(3);
const p2 = Promise.reject('出错了');

Promise.allSettled([p1, p2])
  .then((results) => results.forEach((result) => console.log(result)));
// 输出:
// {status: "fulfilled", value: 3}
// {status: "rejected", reason: "出错了"}
```

#### Promise.race
接收一个集合，返回一个结果，结果是最先返回的结果
```
const fast = new Promise((resolve) => setTimeout(resolve, 100, '快的'));
const slow = new Promise((resolve) => setTimeout(resolve, 500, '慢的'));

Promise.race([fast, slow])
  .then((value) => console.log(value)) // 约100ms后输出: "快的"
  .catch((error) => console.error(error));
```

#### Promise.any
接收一个集合，返回任何一个成功的结果，并返回成功的最快的结果，全部失败才为失败
```
const p1 = Promise.reject('错误1');
const p2 = new Promise((resolve) => setTimeout(resolve, 100, '快的成功'));
const p3 = new Promise((resolve) => setTimeout(resolve, 500, '慢的成功'));

Promise.any([p1, p2, p3])
  .then((value) => console.log(value)) // 约100ms后输出: "快的成功"
  .catch((error) => console.error(error));
```

#### 简单实现一个Promise
const promse = new Promise((resolve,reject)=>{}).then().catch
```
Class MyPromise {
  constructor(executor){
    this.status = 'pending'
    this.onRejectCallback = [] //失败回调函数队列
    this.onResolveCallback = [] //成功回调函数队列
    resolve(value){
      if(this.status === 'pending'){
        this.status = 'fulfilled'
        this.value = value
        this.onResolveCallback.forEach(fn=>fn())
      }
    }
    reject(err){
      if(this.status === 'pending'){
        this.status = 'rejected'
        this.reason = err
        this.onRejectCallback.forEach(fn=>fn())
      }
    }
    try {
      executor(resolve,reject)
    }catch(err){
      reject(err)
    }
  }

  then(onFulfilled,onRejected){
    onFulfilled = typeof onFulfilled === 'function'?onFulfilled:value=>value
    onRejected = typeof onRejected === 'function'?onRejected:reason=>{throw reason}
    const promise2 = new MyPromise((resolve,reject)=>{
      const handleFulfilled = ()=>{
        queueMicroTask(()=>{
          try {
            const x = onFulfilled(this.value)
            resolvePromise(promise2, x, resolve, reject)
          }catch(err){
            reject(err)
          }
        })
      }
       const handleRejected = () => {
        queueMicrotask(() => {
          try {
            const x = onRejected(this.reason);
            resolvePromise(promise2, x, resolve, reject);
          } catch (err) {
            reject(err);
          }
        });
      };
      if(this.status === 'fulfilled'){
        handleFulfilled()
      }
      if(this.status === 'rejected'){
        handleRejected()
      }
      if(this.status === 'pending'){
        this.onResolveCallback.push(handleFulfilled)
        this.onRejectCallback.push(handleRejected)
      }
    })
    return promise2
  }
  catch(onRejected){
    return this.then(null,onRejected)
  }
  static resolve(value){
    return new MyPromise(resolve=>resolve(value))
  }
  static reject(reason){
    return new MyPromise((null,onject)=>onject(reason))
  }
}
// 解析 Promise 的递归函数（符合 Promise A+ 规范）
function resolvePromise(promise2, x, resolve, reject) {
  // 禁止循环引用：promise2 不能与 x 相同
  if (promise2 === x) {
    return reject(new TypeError('Chaining cycle detected for promise'));
  }

  // 如果 x 是 Promise 实例，则让 promise2 接受其状态
  if (x instanceof MyPromise) {
    x.then(
      value => resolve(value),
      reason => reject(reason)
    );
  } else if (x !== null && (typeof x === 'object' || typeof x === 'function')) {
    // 处理 thenable 对象（例如第三方 Promise 实现）
    let then;
    try {
      then = x.then;
    } catch (err) {
      return reject(err);
    }

    // 如果 then 是函数，则调用它并递归解析
    if (typeof then === 'function') {
      let called = false; // 防止重复调用
      try {
        then.call(
          x,
          value => {
            if (!called) {
              called = true;
              resolvePromise(promise2, value, resolve, reject);
            }
          },
          reason => {
            if (!called) {
              called = true;
              reject(reason);
            }
          }
        );
      } catch (err) {
        if (!called) {
          reject(err);
        }
      }
    } else {
      // 普通对象/值，直接 resolve
      resolve(x);
    }
  } else {
    // 基础类型值，直接 resolve
    resolve(x);
  }
}
```

##### Promise.then为什么可以链式调用？
答案：
每个then方法都返回一个新的Promise实例
新Promise的状态由回调函数的返回值决定
通过递归解析返回值，处理Promise、thenable对象和普通值
实现了值的传递和错误的冒泡传播

##### 手写实现promise.all
输入一个可迭代对象
如果输入不是可迭代对象，返回一个已拒绝的promise
等待所有promise完成
如果其中任何一个失败，返回失败的
返回值：一个新的promise
```
// 手写 Promise.all
Promise.myAll = function(promises) {
    // 1. 返回一个新的 Promise
    return new Promise((resolve, reject) => {
        // 2. 检查输入是否为可迭代对象
        // 判断 promises 是否可迭代，最基本的是检查是否有 [Symbol.iterator] 方法
        if (promises == null || typeof promises[Symbol.iterator] !== 'function') {
            return reject(new TypeError(`${promises} is not iterable`));
        }

        // 3. 将可迭代对象转为数组，便于处理
        const promisesArray = Array.from(promises);
        const len = promisesArray.length;

        // 4. 如果传入的是空数组，直接返回一个已解决状态的 Promise，结果为 []
        if (len === 0) {
            return resolve([]);
        }

        // 5. 初始化结果数组和计数器
        const results = new Array(len); // 创建一个长度确定的空数组，用于存放结果
        let completedCount = 0; // 记录已完成的 Promise 数量（成功或失败）

        // 6. 遍历 promises 数组
        promisesArray.forEach((promise, index) => {
            // 6.1 确保当前项是一个 Promise。
            // 如果传入的不是 Promise（比如数字、字符串），用 Promise.resolve() 包装它，使其成为 Promise
            Promise.resolve(promise)
                .then((value) => {
                    // 6.2 当一个 Promise 成功解决时
                    // 关键：将结果存入 results 数组的对应索引位置，保证顺序
                    results[index] = value;
                    completedCount++;

                    // 6.3 检查是否所有 Promise 都已完成（无论成功失败）
                    // 只有当所有都成功完成时，才 resolve 最终结果
                    if (completedCount === len) {
                        resolve(results); // 所有 Promise 都成功，返回结果数组
                    }
                })
                .catch((reason) => {
                    // 6.4 如果其中任何一个 Promise 被拒绝，立即拒绝整个 Promise.all
                    reject(reason);
                });
        });
    });
};
```