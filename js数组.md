#### 一、数组方法
##### 改变原数组方法：
排序：
reverse,sort
长度改变：
splice: array.splice(start[, deleteCount[, item1[, item2[, ...]]]])
,pop,push,shift,unshift
填充：
fill,copyWithin(复制数组一部分到数组的另一个位置)

##### 不改变原数组方法
slice：arr.slice([start[, end]])，不包含end
concat
join
map
filter
reduce
flat
toString
indexOf
lastIndexOf
find
findIndex
some
every
includes
flat
flatMap

#### 二、数组浅复制与深复制
```const original = [1, 2, { name: 'Alice' }]```
##### 浅复制：只复制数据的第一层，如果是复杂数据类型，只复制引用地址，如果对复杂数组类型进行修改，会修改原数组
1、slice:

```const shallowCopy = original.slice();```

2、concat
```const shallowCopy = [].concat(original);```

3、[...]
```const shallowCopy = [...original];```

4、Array.from(arr)
```const shallowCopy = Array.from(original);```

##### 深复制：逐层复制
1、JSON.parse(JSON.stringify(arr))

```const deepCopy = JSON.parse(JSON.stringify(original))```

无法复制函数、undefined、Symbol等特殊类型
循环引用会报错

2、lodash.cloneDeep

```
import _ from 'lodash';
const deepCopy = _.cloneDeep(original);

```
3、递归

```
function deepClone(arr){
  //判断基础数据类型
  if(arr === null || typeof arr !== 'object'){
    return arr
  }
  
  //date
  if(arr instanceof Date){
    return new Date(arr.getTime())
  }
  //数组
  if(Array.isArray(arr)){
    const res = []
    for(const i of arr){
      res.push(deepClone(i))
    }
    return res
  }
  //对象
  if(arr instanceof Object){
    const res = {}
    for(const key in arr){
      if(arr.hasOwnProperty(key)){
        res[key] = deepClone(arr[key])
      }
    }
    return res
  }
  throw new Error("Unable to copy object")
}
```
4、structuredClone浏览器API
浏览器原生支持，兼容性IE不支持

##### 数组扁平化
1、使用 Array.prototype.flat()（ES2019）
```
const arr = [1, [2, [3, [4, 5]], 6]];
const flattened = arr.flat(Infinity);
console.log(flattened); // [1, 2, 3, 4, 5, 6]
```
2、手动实现
```
function flattenDeep(arr) {
  const result = [];
  
  for (const item of arr) {
    if (Array.isArray(item)) {
      // 如果是数组，递归调用，并将结果通过扩展运算符或concat放入result
      result.push(...flattenDeep(item));
      // 或者用： result = result.concat(flattenDeep(item));
    } else {
      result.push(item);
    }
  }
  
  return result;
}

// 测试
const arr = [1, [2, [3, [4, 5]], 6]];
console.log(flattenDeep(arr)); // [1, 2, 3, 4, 5, 6]
```

3、reduce实现
```
function flattenDeep(arr) {
  return arr.reduce((acc, cur) => {
    // 如果当前元素是数组，则递归调用，并将结果合并到累加器
    // 如果不是，则直接将当前元素放入累加器
    return acc.concat(Array.isArray(cur) ? flattenDeep(cur) : cur);
  }, []);
}

// 测试
const arr = [1, [2, [3, [4, 5]], 6]];
console.log(flattenDeep(arr)); // [1, 2, 3, 4, 5, 6]
```
4、指定深度
```
// depth 为扁平化深度，默认为无限深度（用 Infinity 表示）
function flattenDepth(arr, depth = Infinity) {
  // 如果深度为0，直接返回原数组的浅拷贝
  if (depth < 1) return arr.slice();

  return arr.reduce((acc, cur) => {
    // 如果当前元素是数组且深度还未用完，则递归调用，深度减1
    if (Array.isArray(cur) && depth > 0) {
      acc.push(...flattenDepth(cur, depth - 1));
    } else {
      acc.push(cur);
    }
    return acc;
  }, []);
}

// 测试
const arr = [1, [2, [3, [4, 5]], 6]];

console.log(flattenDepth(arr, 1)); // [1, 2, [3, [4, 5]], 6]
console.log(flattenDepth(arr, 2)); // [1, 2, 3, [4, 5], 6]
console.log(flattenDepth(arr, 3)); // [1, 2, 3, 4, 5, 6] (等同于 Infinity)
```

##### 遍历方法
for in和for of
for in适合遍历对象属性，但是也会遍历原型链上的属性，需要用hasOwnProperty过滤掉，遍历结果为属性名，键值
for of遍历可迭代属性，可迭代对象（Array, Map, Set, String等），遍历结果为属性值

如何遍历对象
1、for...in 循环
```
const person = {
    name: 'John',
    age: 30,
    city: 'New York'
};

// 遍历对象自身和继承的可枚举属性
for (let key in person) {
    console.log(key + ': ' + person[key]);
}
// 输出:
// name: John
// age: 30
// city: New York

// 只遍历自身属性（推荐）
for (let key in person) {
    if (person.hasOwnProperty(key)) {
        console.log(key + ': ' + person[key]);
    }
}
```
2、Object.keys() + forEach()
```
const person = { name: 'John', age: 30, city: 'New York' };

Object.keys(person).forEach(key => {
    console.log(key + ': ' + person[key]);
});

// 或者使用数组方法
Object.keys(person).forEach(key => {
    console.log(`${key}: ${person[key]}`);
});
```
3、Object.values() - 遍历值
```
const person = { name: 'John', age: 30, city: 'New York' };

Object.values(person).forEach(value => {
    console.log(value);
});
// 输出:
// John
// 30
// New York
```
4、Object.entries() - 遍历键值对
```
const person = { name: 'John', age: 30, city: 'New York' };

// 使用 for...of
for (let [key, value] of Object.entries(person)) {
    console.log(`${key}: ${value}`);
}

// 使用 forEach
Object.entries(person).forEach(([key, value]) => {
    console.log(`${key}: ${value}`);
});

// 输出:
// name: John
// age: 30
// city: New York
```

5、for of
```
const person = { name: 'John', age: 30, city: 'New York' };

// 使用 for...of 配合 Object.keys()
for (let key of Object.keys(person)) {
    console.log(key + ': ' + person[key]);
}

// 使用 for...of 配合 Object.entries()
for (let [key, value] of Object.entries(person)) {
    console.log(key + ': ' + value);
}
```