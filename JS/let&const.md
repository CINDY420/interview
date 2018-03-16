# `let` & `const`

- [`let` & `const`](#let-const)
  - [`let`](#let)
    - [**作用域**：只在**当前代码块**有效](#%E4%BD%9C%E7%94%A8%E5%9F%9F%EF%BC%9A%E5%8F%AA%E5%9C%A8%E5%BD%93%E5%89%8D%E4%BB%A3%E7%A0%81%E5%9D%97%E6%9C%89%E6%95%88)
      - [关于 `for` 循环](#%E5%85%B3%E4%BA%8E-for-%E5%BE%AA%E7%8E%AF)
    - [**不存在变量提升**](#%E4%B8%8D%E5%AD%98%E5%9C%A8%E5%8F%98%E9%87%8F%E6%8F%90%E5%8D%87)
    - [**不允许重复声明**](#%E4%B8%8D%E5%85%81%E8%AE%B8%E9%87%8D%E5%A4%8D%E5%A3%B0%E6%98%8E)
    - [**`let` 实际上为 JavaScript 新增了块级作用域**](#let-%E5%AE%9E%E9%99%85%E4%B8%8A%E4%B8%BA-javascript-%E6%96%B0%E5%A2%9E%E4%BA%86%E5%9D%97%E7%BA%A7%E4%BD%9C%E7%94%A8%E5%9F%9F)
  - [`const`](#const)
  - [暂时性死区](#%E6%9A%82%E6%97%B6%E6%80%A7%E6%AD%BB%E5%8C%BA)
  - [参考](#%E5%8F%82%E8%80%83)

---

## `let`

 ### **作用域**：只在**当前代码块**有效

  ``` javascript

  for (let i = 0; i < 10; i++) {
    // ...
  }

  console.log(i) // // ReferenceError: i is not defined
  
  ```
  计数器`i`只在`for`循环体内有效，在循环体外引用就会报错.

  所以 `for` 循环的计数器，就很合适使用 `let` 命令。

  ``` javascript

  // 使用 var

  var a = [];
  for (var i = 0; i < 10; i++) {
    a[i] = function () {
      console.log(i);
    };
  }
  a[6](); // 10

  // 使用 let
  
  var a = [];
  for (let i = 0; i < 10; i++) {
    a[i] = function () {
      console.log(i);
    };
  }
  a[6](); // 6
  
  ```
  变量`i`是`let`声明的，当前的`i`只在**本轮循环**有效，所以每一次循环的`i`其实都是一个“新的变量”，所以最后输出的是6。
  
  那既然每一轮循环的变量i都是重新声明的，那它怎么知道上一轮循环的值，从而计算出本轮循环的值？这是因为 JavaScript 引擎内部会记住上一轮循环的值，初始化本轮的变量 `i` 时，就在上一轮循环的基础上进行计算。

  #### 关于 `for` 循环

  ``` javascript
  for (let i = 0; i < 3; i++) {
    let i = 'abc';
    console.log(i);
  }
  // 打印 3个 abc
  ```

  `for` 循环一个特别之处，就是设置循环变量的那部分是一个**父作用域**，而循环体内部是一个单独的 **子作用域**

 ### **不存在变量提升**

  ``` javascript
  // var 的情况
  console.log(foo); // 输出undefined
  var foo = 2;

  // let 的情况
  console.log(bar); // 报错ReferenceError
  let bar = 2;
  ```

 ### **不允许重复声明**

  ``` javascript

  // 报错
  function func() {
    let a = 10;
    var a = 1;
  }

  // 报错
  function func() {
    let a = 10;
    let a = 1;
  }

  // 报错, 不能在函数内部重新声明参数。
  function func(arg) {
    let arg; 
  }
  // 不报错
  function func(arg) {
    {
      let arg; 
    }
  }
  ```

 ### **`let` 实际上为 JavaScript 新增了块级作用域**

  ``` javascript
  // IIFE 写法
  (function () {
    var tmp = ...
  }());

  // 块级作用域写法
  {
    let tmp = ...
  }
  ```

  块级作用域下的函数声明, **行为类似于 `var` 声明的变量**

  ``` javascript
  // 浏览器的 ES6 环境
  function f() { console.log('outside'); }

  (function () {
    if (false) {
      // 重复声明一次函数 f
      function f() { console.log('inside'); }
    }

    f();
  }());

  // Uncaught TypeError: f is not a function

  // 在上面的立即执行函数中，相当于：

  var f = undefined;    // 即 f 被提升
  if (false) {
    function f() { console.log('I am inside!'); }
  }

  f();
  ```


---

## `const`

  > `const` 声明一个**只读的常量**。一旦声明，**常量的值就不能改变**

  但是!!!!😕

  `const` 实际上保证的，并不是变量的值不得改动，而是**变量指向的那个内存地址不得改动**

  - 简单类型的数据（数值、字符串、布尔值），值就保存在变量指向的那个内存地址，因此等同于常量

  - 对于复合类型的数据（主要是对象和数组），变量指向的**内存地址**，保存的只是一个指针， **const** 只能保证这个指针是固定的，至于它指向的数据结构是不是可变的，就完全不能控制了

  ``` javascript

  const foo = {};

  // 为 foo 添加一个属性，可以成功
  foo.prop = 123;
  foo.prop // 123

  // 将 foo 指向另一个对象，就会报错
  foo = {}; // TypeError: "foo" is read-only

  const a = [];
  a.push('Hello');  // 可执行
  a.length = 0;     // 可执行
  a = ['Dave'];     // 报错

  ```

  至于如何冻结一个对象，往这儿看 👉[`Object.freeze` VS `Object.seal`](./freezeVSseal.md)

---

## 暂时性死区

  > ES6 明确规定，如果区块中存在 `let` 和 `const` 命令，这个区块对这些命令声明的变量，从一开始就形成了**封闭作用域**。**凡是在声明之前就使用这些变量，就会报错**。这在语法上，称为“**暂时性死区**”（temporal dead zone，简称 TDZ）

  ``` javascript

  if (true) {
    // TDZ开始
    tmp = 'abc'; // ReferenceError
    console.log(tmp); // ReferenceError

    let tmp; // TDZ结束
    console.log(tmp); // undefined
  }
  
  ```

  **“暂时性死区”意味着 `typeof` 不再是一个百分之百安全的操作**。

  ``` javascript

  typeof undeclared_variable  // "undefined"
  
  typeof x                   // ReferenceError
  let x

  ```

  **默认参数的“死区”**

  ``` javascript

  function bar(x = y, y = 2) {
    return [x, y];
  }

  bar(); // 报错
  
  ```

  参数 `x` 默认值等于另一个参数 `y` ，而**此时 `y` 还没有声明**，属于 死区

  ``` javascript
  function bar(x = 2, y = x) {
    return [x, y];
  }
  bar(); // [2, 2]
  ```

  这个情况，`y` 默认值是 `x`, 而此时 `x` 已经声明，是完全O文明K的

---

## 参考

[let 和 const 命令](http://es6.ruanyifeng.com/#docs/let)