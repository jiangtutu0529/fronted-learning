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