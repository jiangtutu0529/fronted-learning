### TypeScript
##### interface 定义对象的结构，可以被extends，会合并，
##### type 类型别名,可以用作联和，交叉计算，不会合并
#### 高级类型
```
//联合类型
let A = B|C
//类型守卫
function isString(test: any): test is string {
    return typeof test === "string";
}
keyof操作符
interface Person {
  name:string
  age:number
}
type PersonKey = keyof Person   'name'|'age'

// typeof 操作符（类型上下文）
let s = "hello";
let n: typeof s; // string

// 索引访问类型
type NameType = Person["name"]; // string

// 映射类型
type Readonly<T> = {
    readonly [P in keyof T]: T[P];
};
type Optional<T> = {
    [P in keyof T]?: T[P];
};
```

##### ts和js的区别
1、ts是静态类型，js是动态类型
2、ts会被编译成js执行
3、ts类型安全，ts有更好的代码提示
4、ts面向对象特性

##### any,unknown,null,never
any:任意类型，绕过类型检查
unknown:类型安全的any，在使用前必须进行类型判断和断言
never:永远不会出现的值，用于抛出异常或无线循环的函数


#### 实现一个简单的Pick
```
interface Person {
  name:string
  age:number
  sex:string
}
type PickKey<T,P extends keyof T> = {
  [P in T]:T[P]
}
type PickTest = PickKey<Person,'age'|'name'>
```

#### 实现一个Partial
```
interface Person {
  name:string
  age:number
  srx:string
}
type PartialType<T> = {
  [P in keyof T]?:T[P]
}
```

#### tsconfig的主要配置有哪些？
​​compilerOptions​​：编译选项
target：编译目标（ES5, ES6, ES2015等）
module：模块系统（commonjs, amd, es2015等）
strict：严格模式开关
outDir：输出目录
​​include​​/​​exclude​​：包含/排除的文件
​​files​​：指定要编译的文件列表

#### .d.ts文件的作用
声明文件用于描述已有的js库的类型信息，让ts能够理解

##### 什么是TypeScript泛型？为什么要使用它？
答案：泛型允许创建可重用的组件，这些组件可以支持多种类型而不是单一类型。主要好处：
类型安全：在编译时捕获类型错误
代码复用：编写一次，支持多种类型
更好的类型推断：TypeScript可以更好地推断类型

##### 泛型约束的作用是什么？举例说明
答案：泛型约束使用extends关键字限制泛型参数必须满足某些条件
```
// 约束T必须有length属性
interface HasLength {
    length: number;
}

function getLength<T extends HasLength>(arg: T): number {
    return arg.length;
}

getLength("hello"); // 正确
getLength([1, 2, 3]); // 正确
// getLength(123); // 错误：number没有length属性
```

#####  keyof关键字的作用是什么？
答案：keyof操作符获取对象类型的所有键的联合类型
```
interface Person {
    name: string;
    age: number;
}

type PersonKeys = keyof Person; // "name" | "age"

function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
    return obj[key];
}
```