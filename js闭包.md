### js闭包
闭包只函数可以记住并访问定义时的作用域，即使函数在外部执行时，依然可以看到定义时的变量。本质上是函数和和函数引用的变量的组合

当一个函数执行时，会创建一个上下文，存储相关的变量，函数参数，以及作用域链，作用域链是指当前执行上下文中所有变量的引用链，它允许函数访问外部的变量。

当我们在一个函数内部定义一个函数时，内部函数会有对于外部函数作用域的引用，即使外部函数已经执行完毕，内部函数依然能访问外部函数的局部变量。

```
function outer() {
    let counter = 0;  // outer函数中的局部变量

    function inner() {
        counter++;  // inner函数访问外部函数的变量
        console.log(counter);
    }

    return inner;  // 返回inner函数
}

const closureFunc = outer();  // 执行outer函数，返回inner函数
closureFunc();  // 输出: 1
closureFunc();  // 输出: 2
closureFunc();  // 输出: 3

```

##### 闭包的应用
1、数据封装和私有变量
闭包可以模拟私有变量，防止外部直接访问和修改部内的变量的值。在js中，函数内部的变量时私有的，只有通过返回的函数才能访问。
```
function createCounter() {
 let count = 0;  // 私有变量

 return {
     increment: function() {
         count++;
         return count;
     },
     decrement: function() {
         count--;
         return count;
     },
     getCount: function() {
         return count;
     }
 };
}
const counter = createCounter();
console.log(counter.increment());  // 输出: 1
console.log(counter.increment());  // 输出: 2
console.log(counter.getCount());   // 输出: 2
console.log(counter.decrement());  // 输出: 1
console.log(counter.count);        // 输出: undefined (无法直接访问 count)

```
在这个例子中，count 是一个私有变量，无法直接从外部访问。通过闭包返回的 increment、decrement 和 getCount 函数，外部可以间接访问和修改 count 的值，但无法直接访问或修改它

2、函数柯里化
闭包可以帮助我们实现柯里化，柯里化是指将一个多参数的函数转变成多个单一参数的函数。每个返回的函数可以继续接收参数，知道接收到所有需要的参数，然后再执行原有的函数。
```
function multiply(a) {
    return function(b) {
        return a * b;
    };
}

const multiplyBy2 = multiply(2);
console.log(multiplyBy2(5));  // 输出: 10
console.log(multiplyBy2(3));  // 输出: 6

```
在这个例子中，multiply 函数返回一个闭包，闭包函数可以记住并访问 a 变量，从而实现了柯里化功能。multiplyBy2 就是一个已经设置了参数 a=2 的柯里化函数，它接收 b 参数并返回 a * b 的值


3、事件处理和回调函数
闭包在事件处理和回调函数中也有广泛的应用。由于闭包能够“记住”外部函数的局部变量，它特别适用于事件处理器或异步操作中的数据存储和状态保持
```
function buttonHandler(buttonId) {
    let count = 0;

    document.getElementById(buttonId).addEventListener('click', function() {
        count++;
        console.log(`Button clicked ${count} times`);
    });
}

buttonHandler('myButton');  // 假设页面中有一个 id 为 'myButton' 的按钮

```
在这个例子中，每当按钮被点击时，事件监听器会触发回调函数。通过闭包，回调函数能够访问并修改 count 变量，记录按钮被点击的次数


4、定时器与延时操作
闭包还常常用于定时器（如 setTimeout 或 setInterval）中的延迟操作。闭包能够保持函数作用域中的变量，即使异步操作的回调函数稍后才执行

```
  function countdown(start) {
    let counter = start;

    setInterval(function() {
        if (counter > 0) {
            console.log(counter);
            counter--;
        } else {
            console.log('Time\'s up!');
            clearInterval(this);
        }
    }, 1000);
}

countdown(5);  // 输出: 5, 4, 3, 2, 1, Time's up!

```
在这个例子中，setInterval 定时器的回调函数可以通过闭包访问到 counter 变量，并每秒递减。即使 countdown 函数早已执行完毕，闭包仍然可以保持 counter 的状态。



#### 闭包注意事项
内存泄漏：由于闭包能够持有外部函数的变量，可能导致这些变量在不再需要时不能被垃圾回收，从而造成内存泄漏。因此，在使用闭包时需要小心，避免不必要的引用。
调试困难：闭包中的变量状态较为隐蔽，可能会增加代码的调试难度，特别是在多个闭包嵌套的情况下。