##### axios取消请求
AbortControl
AbortController是 Web API 标准，提供 abort()方法和 signal属性
signal会传递给 fetch 或 XMLHttpRequest
调用 abort()时，浏览器原生支持请求中止
```
const control = new AbortControl()
const signal = control.signal
axios.get(url,{
  signal:signal
})
control.aboer()
```

```
// 方法1: 使用CancelToken（传统方式）
const source = axios.CancelToken.source();

axios.get('/api/data', {
  cancelToken: source.token
});

// 取消请求
source.cancel('Operation canceled by user.');

// 方法2: 使用AbortController（推荐）
const controller = new AbortController();

axios.get('/api/data', {
  signal: controller.signal
});

// 取消请求
controller.abort();
```

##### 实现请求防抖，取消重复请求
```
class RequestAbort{
  constructor(){
    this.pendingKeys = new Map()
  }
  resquest(key,realRequest){
    if(this.pendingKeys.has(key)){
      this.pendingKeys.get(key).abort()
    }
    const control = new AbortControl()
    this.pendingKeys.set(key,control)
    try{
      const response = await realRequest(control.signal)
      this.pendingKeys.delete(key)
      return response
    }catch(error){
      if(error.name === 'CanceledError'){
        this.pendingKeys.delete(key)
      }
      throw error
    }
  }
}

const fetch = new requestAbort()
fetch.request('1111111',(signal)=>{
  return axios.get(url,{signal})
})
```
##### 实现超时自动取消请求
```
function requestTimeout(url,time){
  const control = new AbortControl()

  let timer = setTimeout(()=>{
    control.abort()
  },[time])
  
  return axios.get(url,{
    signal:control.signal
  }).finally(()=>{
    clearTimeout(timer)
  })
}
```

##### 函数柯里化
将多参数函数转化为一系列单函数
```
// 普通多参数函数
function add(a, b, c) {
    return a + b + c;
}

// 柯里化后的函数
function curriedAdd(a) {
    return function(b) {
        return function(c) {
            return a + b + c;
        };
    };
}

// 使用方式
add(1, 2, 3);        // 6
curriedAdd(1)(2)(3); // 6
```

```
function curry(fn) {
    return function curried(...args) {
        if (args.length >= fn.length) {
            return fn.apply(this, args);
        } else {
            return function(...args2) {
                return curried.apply(this, args.concat(args2));
            };
        }
    };
}

// 测试
function sum(a, b, c, d) {
    return a + b + c + d;
}

const curriedSum = curry(sum);
console.log(curriedSum(1)(2)(3)(4));     // 10
console.log(curriedSum(1, 2)(3, 4));     // 10
console.log(curriedSum(1)(2, 3, 4));     // 10
```