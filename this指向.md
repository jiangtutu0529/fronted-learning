##### this的指向问题
this指向在函数被调用时确定，不在定义时确定
1、独立调用，this指向window
2、函数作为对象的方法调用时，this指向对象
3、显式绑定，指向绑定的对象
4、new 绑定函数，this指向新创建的实例
5、箭头函数，没有自己的this，this继承外层作用域

优先级：
new 绑定 > 显式绑定 > 隐式绑定 > 默认绑定

```
const obj = {
  name: 'Alice',
  sayHello: function() {
    console.log(`Hello, I'm ${this.name}`);
  }
};

obj.sayHello(); // "Hello, I'm Alice" - this指向obj

// 隐式丢失问题
const hello = obj.sayHello;
hello(); // "Hello, I'm undefined" - this指向全局对象

function introduce(age, hobby) {
  console.log(`I'm ${this.name}, ${age} years old, like ${hobby}`);
}

const person1 = { name: 'Alice' };
const person2 = { name: 'Bob' };

// call - 立即执行，参数逐个传递
introduce.call(person1, 25, 'reading'); 
// "I'm Alice, 25 years old, like reading"

// apply - 立即执行，参数数组传递
introduce.apply(person2, [30, 'swimming']);
// "I'm Bob, 30 years old, like swimming"

// bind - 返回新函数，延迟执行
const bobIntro = introduce.bind(person2, 30, 'swimming');
bobIntro(); // "I'm Bob, 30 years old, like swimming"


function Person(name) {
  this.name = name;
  this.sayHello = function() {
    console.log(`Hello from ${this.name}`);
  };
}

const alice = new Person('Alice');
alice.sayHello(); // "Hello from Alice" - this指向alice实例


const obj = {
  name: 'Alice',
  regularFunc: function() {
    console.log('Regular:', this.name); // this指向obj
  },
  arrowFunc: () => {
    console.log('Arrow:', this.name); // this指向外层作用域（全局）
  },
  nested: function() {
    const innerArrow = () => {
      console.log('Nested Arrow:', this.name); // this继承自nested函数
    };
    innerArrow();
  }
};

obj.regularFunc(); // "Regular: Alice"
obj.arrowFunc();   // "Arrow: undefined" (浏览器中可能是空字符串)
obj.nested();      // "Nested Arrow: Alice"


function test() {
  console.log(this.name);
}

const obj1 = { name: 'Alice', test: test };
const obj2 = { name: 'Bob' };

// 优先级演示
obj1.test(); // "Alice" - 隐式绑定

const boundTest = test.bind(obj2);
boundTest(); // "Bob" - 显式绑定优先于隐式绑定

obj1.test.call(obj2); // "Bob" - 显式绑定优先
```

```
name = 'w'
var obj = {
    name:'a',
    fn:()=>{
        console.log(this.name)
    },
    fn1:function(){
        console.log(this.name)
        console.log(name)
        var name = 'c'
    },
    fn2:function(){
        return function(){
            console.log(name)
        }
    }
}
obj.fn()
obj.fn1()
const fn3 = obj.fn2()
fn3()
```

##### setTimeout中的this
setTimeout的回调函数是独立调用的，this指向全局对象。
```
var name = 'Global';

const obj = {
  name: 'Alice',
  sayName: function() {
    setTimeout(function() {
      console.log(this.name);
    }, 100);
  }
};

obj.sayName(); // 输出什么？
```
window，因为setTimeout是另外调用的，所以指向window

解决方案：
```
好的，JavaScript 中的 this是一个非常重要且容易让人困惑的概念。我来为你详细解析 this的指向规则和相关面试题。
一、this的五大绑定规则
this的指向在函数被调用时确定，而不是在函数定义时确定。主要有以下五大规则：
1. 默认绑定（独立函数调用）
函数独立调用时，this指向全局对象（浏览器中为 window，Node.js 中为 global）。
function showThis() {
  console.log(this);
}

showThis(); // 浏览器中: Window, Node.js中: global

// 严格模式下为 undefined
function strictThis() {
  'use strict';
  console.log(this);
}
strictThis(); // undefined
2. 隐式绑定（方法调用）
函数作为对象的方法调用时，this指向调用该方法的对象。
const obj = {
  name: 'Alice',
  sayHello: function() {
    console.log(`Hello, I'm ${this.name}`);
  }
};

obj.sayHello(); // "Hello, I'm Alice" - this指向obj

// 隐式丢失问题
const hello = obj.sayHello;
hello(); // "Hello, I'm undefined" - this指向全局对象
3. 显式绑定（call/apply/bind）
使用 call、apply或 bind方法显式指定 this。
function introduce(age, hobby) {
  console.log(`I'm ${this.name}, ${age} years old, like ${hobby}`);
}

const person1 = { name: 'Alice' };
const person2 = { name: 'Bob' };

// call - 立即执行，参数逐个传递
introduce.call(person1, 25, 'reading'); 
// "I'm Alice, 25 years old, like reading"

// apply - 立即执行，参数数组传递
introduce.apply(person2, [30, 'swimming']);
// "I'm Bob, 30 years old, like swimming"

// bind - 返回新函数，延迟执行
const bobIntro = introduce.bind(person2, 30, 'swimming');
bobIntro(); // "I'm Bob, 30 years old, like swimming"
4. new 绑定（构造函数）
使用 new关键字调用构造函数时，this指向新创建的对象实例。
function Person(name) {
  this.name = name;
  this.sayHello = function() {
    console.log(`Hello from ${this.name}`);
  };
}

const alice = new Person('Alice');
alice.sayHello(); // "Hello from Alice" - this指向alice实例
5. 箭头函数
箭头函数没有自己的 this，它的 this继承自外层作用域。
const obj = {
  name: 'Alice',
  regularFunc: function() {
    console.log('Regular:', this.name); // this指向obj
  },
  arrowFunc: () => {
    console.log('Arrow:', this.name); // this指向外层作用域（全局）
  },
  nested: function() {
    const innerArrow = () => {
      console.log('Nested Arrow:', this.name); // this继承自nested函数
    };
    innerArrow();
  }
};

obj.regularFunc(); // "Regular: Alice"
obj.arrowFunc();   // "Arrow: undefined" (浏览器中可能是空字符串)
obj.nested();      // "Nested Arrow: Alice"
二、优先级规则
new 绑定 > 显式绑定 > 隐式绑定 > 默认绑定
function test() {
  console.log(this.name);
}

const obj1 = { name: 'Alice', test: test };
const obj2 = { name: 'Bob' };

// 优先级演示
obj1.test(); // "Alice" - 隐式绑定

const boundTest = test.bind(obj2);
boundTest(); // "Bob" - 显式绑定优先于隐式绑定

obj1.test.call(obj2); // "Bob" - 显式绑定优先
三、经典面试题解析
面试题 1：隐式绑定丢失
var name = 'Global';

const obj = {
  name: 'Alice',
  sayName: function() {
    console.log(this.name);
  }
};

// 常见陷阱
const sayName = obj.sayName;
sayName(); // 输出什么？
答案："Global"
解析：虽然 sayName方法定义在 obj中，但赋值给变量后独立调用，this指向全局对象。
面试题 2：setTimeout 中的 this
var name = 'Global';

const obj = {
  name: 'Alice',
  sayName: function() {
    setTimeout(function() {
      console.log(this.name);
    }, 100);
  }
};

obj.sayName(); // 输出什么？
答案："Global"
解析：setTimeout的回调函数是独立调用的，this指向全局对象。
解决方案：
// 方案1：使用箭头函数
sayName: function() {
  setTimeout(() => {
    console.log(this.name); // "Alice"
  }, 100);
}

// 方案2：使用bind
sayName: function() {
  setTimeout(function() {
    console.log(this.name); // "Alice"
  }.bind(this), 100);
}

// 方案3：保存this引用
sayName: function() {
  const self = this;
  setTimeout(function() {
    console.log(self.name); // "Alice"
  }, 100);
}
```

##### DOM事件处理中的this指向
```
<button id="btn">Click me</button>

<script>
const obj = {
  name: 'Alice',
  handleClick: function() {
    console.log(this); // 点击按钮时输出什么？
  }
};

document.getElementById('btn').addEventListener('click', obj.handleClick);
</script>
```
事件处理函数中的 this指向绑定事件的 DOM 元素

解决方案：
```
// 使用bind
document.getElementById('btn').addEventListener('click', obj.handleClick.bind(obj));

// 使用箭头函数包装
document.getElementById('btn').addEventListener('click', () => {
  obj.handleClick(); // this正确指向obj
});
```

##### this的链式调用
每次方法调用都返回 this，使得可以链式调用，this始终指向 obj
```
const obj = {
  value: 1,
  add: function(num) {
    this.value += num;
    return this; // 返回this实现链式调用
  },
  multiply: function(num) {
    this.value *= num;
    return this;
  },
  getValue: function() {
    return this.value;
  }
};

const result = obj.add(5).multiply(2).add(3).getValue();
console.log(result); // 输出什么？
```
15

##### 构造函数中的this
构造函数返回对象时，new表达式返回该对象而不是新创建的 this
构造函数返回非对象值时，忽略返回值，仍然返回新创建的 this
```
function Person(name) {
  this.name = name;
  return { name: 'Bob' }; // 返回一个对象
}

function Animal(type) {
  this.type = type;
  return 'cat'; // 返回非对象值
}

const person = new Person('Alice');
const animal = new Animal('dog');

console.log(person.name); // 输出什么？
console.log(animal.type); // 输出什么？
```
"Bob"、"dog"

##### class类中的this
```
class Counter {
  constructor() {
    this.count = 0;
  }
  
  increment() {
    this.count++;
    console.log(this.count);
  }
  
  delayedIncrement() {
    setTimeout(function() {
      this.increment(); // 这里会报错！
    }, 1000);
  }
}

const counter = new Counter();
counter.delayedIncrement(); // 会发生什么？
```
报错 TypeError: this.increment is not a function
setTimeout回调中的 this指向全局对象

解决方案：
```
// 方案1：使用箭头函数
delayedIncrement() {
  setTimeout(() => {
    this.increment(); // 正确执行
  }, 1000);
}

// 方案2：使用bind
delayedIncrement() {
  setTimeout(function() {
    this.increment();
  }.bind(this), 1000);
}

// 方案3：类字段语法（实验性）
increment = () => {
  this.count++;
  console.log(this.count);
}
```

##### 严格模式下，独立调用this为undefined

严格模式下，独立函数调用中的 this为 undefined。
```
'use strict';

var name = 'Global';

function test() {
  console.log(this); // 输出什么？
}

test();
```
##### 总结
箭头函数解决回调问题：在定时器、事件监听器等回调中优先使用箭头函数
bind 方法固定 this：在需要明确指定 this 的场景使用 bind
避免隐式丢失：不要轻易将对象方法赋值给变量
类方法绑定：在构造函数中使用 bind 或使用类字段语法
严格模式：注意严格模式对 this 的影响
理解 this的关键在于记住：this 的指向取决于函数的调用方式，而不是定义方式。掌握了这个原则，就能应对各种复杂的 this指向问题。