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

