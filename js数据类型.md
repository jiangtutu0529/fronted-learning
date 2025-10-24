#### 数据类型
string
number
bigInt
boolean
null
undefined
object
function
symbol
#### 判断数据类型
##### 一、typeof
typeof '23' string
typeof 23 number
typeof undefined undefined
typeof function(){}  function
typeof []  object
typeof {} object
typeof null object
##### 二、instanceof  不怎么好使
判断复杂数据类型的原型
[] instanceof Array true
{} instanceof Object true
function(){} instanceof Function true

##### 三、Object.prototype.toString.call
console.log(Object.prototype.toString.call(num));  // "[object Number]"
console.log(Object.prototype.toString.call(str));  // "[object String]"
console.log(Object.prototype.toString.call(bool)); // "[object Boolean]"
console.log(Object.prototype.toString.call(und));  // "[object Undefined]"
console.log(Object.prototype.toString.call(obj));  // "[object Object]"
console.log(Object.prototype.toString.call(arr));  // "[object Array]"
console.log(Object.prototype.toString.call(func)); // "[object Function]"
console.log(Object.prototype.toString.call(nul));  // "[object Null]" (注意：这里用的是 call 方法)


#### es6新方法
1、let const
let 块级作用域，可重新赋值，变量提升
const 块级作用域，不可重新赋值，
var函数作用域，变量提升，可重新赋值

2、箭头函数
const add = (a,b)=>a+b

this函数绑定
const person = {
  name:'aa',
  hello:()=>{
    console.log(this.name)
  }
}
person.hello()  //undefined

const person = {
  name:'aa',
  hello:function(){
    console.log(this.name)
  }
}
person.hello()  //aa

3、模板字符串
``
4、解构赋值
```
const [a,b] = [b,a]

```
5、扩展运算符...  
6、Promise
7、Map
const map = new Map();
map.set('name', 'Alice');
map.set(1, 'number one');

console.log(map.get('name')); // 'Alice'
console.log(map.size); // 2

// 迭代
for (let [key, value] of map) {
  console.log(key, value);
}
8、Set
const set = new Set([1, 2, 2, 3, 3]);
console.log(set); // Set {1, 2, 3}（自动去重）

set.add(4);
set.has(2); // true
set.delete(1);
9、symbol
const sym1 = Symbol('description');
const sym2 = Symbol('description');
console.log(sym1 === sym2); // false（每个 Symbol 都是唯一的）

// 作为对象键名
const obj = {
  [sym1]: 'value'
};

