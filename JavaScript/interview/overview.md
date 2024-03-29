![overview](https://raw.githubusercontent.com/aboutcroon/Notes/main/JavaScript/interview/assets/js%20overview.png)

## 1. 数据类型

## 2. ES6

### 🔴 ES6 有哪些新特性

- let 和 const
- 变量（数组，对象，字符串）的解构赋值
- 正则表达式的扩展
- String 的扩展，字符串的新增方法（matchAll, normalize），模版字符串等
- Number 的扩展（Number.isInteger, Number.isNaN, Number.parseInt, Number.parseFloat），BigInt 数据类型
- 函数的扩展，箭头函数，参数默认值



## 3. JavaScript 基础

### 🔴 new 操作符做了什么事情

执行过程：

- 创建了一个新的空对象
- 为新对象添加属性 `__proto__`，并将该属性指向构造函数的 prototype 属性
- 将构造函数的 this 指向新对象，并执行构造函数内部的代码（为这个新对象添加属性）
- 如果该构造函数没有返回对象，则返回 this





## 4. 原型与原型链

### 🔴 什么是原型与原型链

在 JavaScript 中是使用构造函数来新建一个对象的，每一个构造函数的内部都有一个 prototype 属性，它的属性值是一个对象，这个对象包含了可以由该构造函数的所有实例共享的属性和方法。**当使用构造函数新建一个对象后，在这个对象的内部将包含一个指针，这个指针指向构造函数的 prototype 属性对应的值，在 ES5 中这个指针被称为对象的原型**。

> 现在浏览器中都实现了 `__proto__` 属性来访问这个对象的原型，但是最好不要使用这个属性，因为它不是规范中规定的。ES5 中新增了一个 `Object.getPrototypeOf()` 方法，可以通过这个方法来获取对象的原型。



当访问一个对象的属性时，如果这个对象内部不存在这个属性，那么它就会去它的原型对象里找这个属性，这个原型对象又会有自己的原型，于是就这样一直找下去，也就是原型链的概念。**原型链的尽头一般来说都是 Object.prototype**。

### 原型链的指向

```js
p.__proto__  // Person.prototype
Person.prototype.__proto__  // Object.prototype
p.__proto__.__proto__ // Object.prototype
p.__proto__.constructor.prototype.__proto__ // Object.prototype
Person.prototype.constructor.prototype.__proto__ // Object.prototype
p1.__proto__.constructor // Person
Person.prototype.constructor  // Person
```

### 如何获得对象非原型链上的属性

使用 `hasOwnProperty()` 方法可以判断属性是属于实例本身的属性还是属于原型链上的属性：

```js
function iterate(obj){
   const res = [];
   for(let key in obj){
        if(obj.hasOwnProperty(key)) {
          res.push(key+': '+obj[key]);
        }   
   }
   return res;
} 
```



## 5. 闭包

### 什么是闭包

**闭包(closure)是一个函数以及其捆绑的周边环境状态（词法作用域环境）的引用的组合**

换而言之，**闭包让开发者可以从内部函数访问外部函数的作用域**

在 JavaScript 中，**闭包会随着函数的创建而被创建**

### 闭包示例

```js
function makeFunc() {
    const name = "Mozilla";
    function displayName() {
      	// displayName() 没有自己的局部变量。然而，因为它可以访问到外部函数的变量，所以它可以使用父函数中声明的变量 name
        alert(name);
    }
    return displayName;
}

const myFunc = makeFunc();
myFunc();
```

在一些编程语言中，一个函数中的局部变量仅存在于此函数的执行期间。一旦 `makeFunc()` 执行完毕，你可能会认为 `name` 变量将不能再被访问。然而，在 JavaScript 中情况显然与此不同。

JavaScript 中的函数会形成闭包。**闭包是由函数以及声明该函数的词法环境组合而成的。该环境包含了这个闭包创建时作用域内的任何局部变量**。在本例子中，`myFunc` 是执行 `makeFunc` 时创建的 `displayName` 函数实例的引用。`displayName` 的实例维持了一个对它的词法环境（变量 `name` 存在于其中）的引用。因此，当 `myFunc` 被调用时，变量 `name` 仍然可用，其值就被传递到`alert`中。

### 项目中遇到的闭包

tabA 中请求表格数据，tabB中请求表格数据，但是 A 中数据量较多，在 A 中请求还未返回过来的时候切换到 B，B 的请求已经返回过来了，随后 A 的请求数据返回了并替换掉了 B 的数据。

解决该问题就是利用闭包，在每次请求发出时，保存请求发出时的 tab，并在请求返回时，将闭包保存的先前 tab 与当前 tab 做对比，如果 tab 不一样则丢弃返回的数据。

### 闭包常见用途

- 可以通过在函数外部调用闭包函数，从而在函数外部访问到函数内部的变量

  > 可以使用这种方法来创建私有变量

- 使已经运行结束的函数上下文中的变量对象继续留在内存中，因为闭包函数保留了这个变量对象的引用，所以这个变量对象不会被垃圾回收。

## 6. 执行上下文，作用域链

### 什么是作用域

**全局作用域**

- 最外层函数和最外层函数外面定义的变量
- 所有未定义直接赋值的变量
- 所有 window 对象的属性

过多的全局作用域变量会污染全局命名空间，容易引起命名冲突。



**函数作用域**：声明在函数内部的变量，一般只有固定的代码片段可以访问到



**块级作用域**：let 和 const 指令可以声明块级作用域，块级作用域可以在函数中创建，也可以在一个代码块中创建（由 { } 包裹的代码片段）

在循环中比较适合绑定块级作用域，这样就可以把声明的计数器变量限制在循环内部。

### 什么是作用域链

在当前作用域中查找所需变量，但是该作用域没有这个变量，那这个变量就是自由变量。
如果在自己作用域找不到该变量就去父级作用域查找，依次向上级作用域查找，直到访问到window 对象就被终止，这一层层的关系就是作用域链。

**作用域链的本质是一个指向变量对象的指针列表**。
变量对象是一个包含了执行环境中所有变量和函数的对象。
作用域链的前端始终都是当前执行上下文的变量对象。
**全局执行上下文的变量对象始终是作用域链的最后一个对象**。
**当查找一个变量时，如果当前执行环境中没有找到，可以沿着作用域链向后查找**。

### 作用域链的作用

**保证对执行环境有权访问的所有变量和函数的有序访问**，通过作用域链，可以访问到外层环境的变量和函数。

### 什么是执行上下文

**全局执行上下文**

任何不在函数内部的都是全局执行上下文，它首先会创建一个全局的 window 对象，并且设置this 的值等于这个全局对象，一个程序中只有一个全局执行上下文。

**函数执行上下文**

当一个函数被调用时，就会为该函数创建一个新的执行上下文，函数的上下文可以有任意多个。

### 执行上下文的阶段

**创建阶段**：进行 this 绑定，创建词法环境，创建变量环境

**执行阶段**：完成对变量的分配，最后执行完代码



## 7. this

### this 的概念

this 是执行上下文的一个属性，它指向最后一次调用这个方法的对象

### 🔴 有哪些方法改变 this 指向

优先级按顺序

1. **箭头函数**
   创建箭头函数时，就已经确定了它的 this 指向，箭头函数内的 this 指向外层的 this

2. **new**
   一个函数用 new 调用时，函数执行前会创建一个新对象，this 指向这个新对象。
   （箭头函数不能当做构造函数，所以不能与 new 一起执行）

3. **bind、apply 和 call**
   这三个方法可以显式的指定调用函数的 this 指向。（如果使用 new 运算符构造绑定函数，则忽略传入的 this，例如：boundFunc = func.bind(1); new boundFunc()

   > 为什么需要改变 this 指向？
   > 因为我们在使用 setTimeout 的时候，回调函数的 this 会指向 window 对象（在浏览器中），因为其在定时器中是作为回调函数来执行的，因此回到主栈执行时是在全局执行上下文的环境中执行的。这时我们就需要手动修改 this 的指向。

4. **方法调用**（obj.xx）
   一个函数作为一个对象的方法来调用时，this 指向这个对象。

5. **函数调用** fn()
   直接作为函数来调用时，this 指向全局对象。
   在浏览器环境中全局对象是 Window，在 Node.js 环境中是 Global。

6. **不在函数里**
   在 `<script />` 标签里，this 指向 Window。
   在 Node.js 的模块文件里，this 指向 Module 的默认导出对象，也就是 module.exports。

7. **在非严格模式下，如果得出 this 指向是 undefined 或 null，那么 this 都指向全局对象**



## 8. call/apply/bind

### 🔴 call 和 apply 的区别

第一个参数都是指定函数体内 this 对象的指向，第二个参数都是作为参数传递给被调用的函数。

使用 apply 和 call 方法改变 this 指向后原函数都会立即执行，且只是临时改变 this 指向一次。

apply 的第二个参数是数组或者类数组，call 从第二个参数开始往后，每个参数被依次传入函数。

### 🔴 bind 有什么特殊的点需要我们注意

- bind 方法会创建一个新的函数，所以并不像 call, apply 那样函数会立即执行
- 在 bind 被调用时，这个新函数的 this 被指定为 bind 的第一个参数，而其余参数将作为新函数的参数。
- bind 的传参方式和 call 很相似，**但 bind 的参数列表可以分多次传入，最后函数运行时会把所有参数连接起来一起放入函数运行。**

**示例**

`bind()` 最简单的用法是创建一个函数，不论怎么调用，这个函数都有同样的 **`this`** 值。JavaScript 新手经常犯的一个错误是将一个方法从对象中拿出来，然后再调用，期望方法中的 `this` 是原来的对象（比如在回调中传入这个方法）。如果不做特殊处理的话，一般会丢失原来的对象。基于这个函数，用原始的对象创建一个绑定函数，巧妙地解决了这个问题：

```js
this.x = 9;    // 在浏览器中，this 指向全局的 "window" 对象
const module = {
  x: 81,
  getX: function() { return this.x; }
};

module.getX(); // 81

const retrieveX = module.getX;
retrieveX(); // 返回 9 - 因为函数是在全局作用域中调用的

// 创建一个新函数，把 'this' 绑定到 module 对象
// 新手可能会将全局变量 x 与 module 的属性 x 混淆
const boundGetX = retrieveX.bind(module);
boundGetX(); // 81
```

在你想要为一个需要特定的 **`this`** 值的函数创建一个捷径（shortcut）的时候，`bind()` 也很好用。

你可以用 [`Array.prototype.slice`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/slice) 来将一个类似于数组的对象（array-like object）转换成一个真正的数组，就拿它来举例子吧。你可以简单地这样写：

```js
const slice = Array.prototype.slice;

// ...

slice.apply(arguments);
```

用 `bind()`可以使这个过程变得简单。在下面这段代码里面，`slice` 是 [`Function.prototype`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/prototype) 的 [`apply()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/apply) 方法的绑定函数，并且将 `Array.prototype` 的 [`slice()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/slice) 方法作为 **`this`** 的值。这意味着我们压根儿用不着上面那个 `apply()`调用了。

```js
// 与前一段代码的 "slice" 效果相同
const unboundSlice = Array.prototype.slice;
const slice = Function.prototype.apply.bind(unboundSlice);

// ...

slice(arguments);
```

### 🔴 函数柯里化相关

函数柯里化的定义：在数学和计算机科学中，柯里化是一种将使用多个参数的一个函数转换成一系列使用一个参数的函数的技术。

通过上面的定义可以看出，柯里化是一个函数返回另一个函数，这是一个典型的**闭包**，它封装了一部分不变的内容，然后去处理其他可变的数据

例如我们在使用 ajax 的时候 url 是不变的，但是传递的参数不同将返回不同的数据，那么我们就可以把 url 封装到一个函数里然后返回一个带参数的函数，通过返回的函数去处理不同参数的情况。

通用 curry 函数的实现：

```js
function add(a, b) {
  return a + b;
}
// 执行 add 函数，需要每次都传入两个参数
add(1, 2) // 3
add(1, 3) // 4
add(1, 4) // 5

// 假设有一个 curry 函数可以做到柯里化
function curry(fn, args) {
  const length = fn.length;
  args = args || [];

  return function () {
    const _args = args.slice(0), arg, i;

    for (i = 0; i < arguments.length; i++) {
      arg = arguments[i];
      _args.push(arg);
    }
    if (_args.length < length) {
      return curry.call(this, fn, _args);
    } else {
      return fn.apply(this, _args);
    }
  };
}

const addCurry = curry(add);
const add1 = addCurry(1)
// 只需传入一个参数
add1(2)
add1(3)
add1(4)
```



### 🔴 进行多次.bind之后的结果

进行多次.bind之后的结果，会指向第一个bind的地址。

详细原因看一下 bind 的实现原理：

```js
Function.prototype.bind = function (context) {
  if (typeof this !== "function") {
    throw new Error("Function.prototype.bind - what is trying to be bound is not callable");
  }

  var self = this;
  var bindArgs = Array.prototype.slice.call(arguments, 1); // 从第二个参数开始作为 bind 函数的参数 bindArgs
  var fNOP = function () {};

  var fbound = function () {
    var args = bindArgs.concat(Array.prototype.slice.call(arguments)); // 后续添加的参数也会作为 bindArgs
    self.apply(this instanceof self ? this : context, args);
  }

  fNOP.prototype = this.prototype;
  fbound.prototype = new fNOP();

  return fbound;
}
```

如果清楚这部分实现，bind 两次的结果已经很明显了，在第一次 bind 完 this 就已经确定了，返回了一个函数出去，这个函数体内不存在 this 问题，后续无论 bind 多少次，this 都指向第一次 bind 传入的 context，但是后面 bind 再传入的其他参数会生效。

### 手写实现 call、apply 及 bind



## 9. 异步编程

### 异步编程的实现方式

JavaScript中的异步机制可以分为以下几种：

1. 回调函数，多个回调函数嵌套的时候会造成回调函数地狱，上下两层的回调函数间的代码耦合度太高，不利于代码的可维护。
2. Promise，可将嵌套的回调函数作为链式调用。但是有时会造成多个 then 的链式调用，可能会造成代码的语义不够明确。
3. generator
4. async 函数，可以将异步逻辑，转化为同步的顺序来书写，并且这个函数可以自动执行。

### setTimeout、Promise、Async/Await 的区别

`setTimeout` 是**宏任务**，Promise 里面的 then 方法是**微任务**，async/await 中 await 语法后面紧跟的表达式是同步的，但 await 下面的代码是异步的，属于**微任务**。



setTimeout 是异步执行函数，当 js 主线程执行到此函数时，不会等待 setTimeout 中的回调函数，会直接进行 setTimeout 下面的语句的执行，当执行完当前事件循环的时候，setTimeout 中的回调会在下一次宏任务的事件循环中被执行。

示例：

```js
console.log('setTimeout start');
setTimeout(function(){
    console.log('setTimeout execute');
})
console.log('setTimeout end');

//输出顺序
1. setTimeout start 
2. setTimeout end 
3. setTimeout execute
```



Promise 本身是同步的立即执行函数，但在执行体中执行 resolve 或者 reject 的时候是异步的，即 **then方法是异步的**。

等同步代码完成后，才会去执行 resolve / reject 中的方法。then() 里面的回调就是一个 task，如果是 resolve 或者 reject 出一个结果了，那么就会把这个 task 放入微任务队列中，等待执行；如果一直是 pending，这个 task 就会放入事件循环的未来的某个(可能下一个)回合的微任务队列中。

setTimeout 的回调也是个 task ，它会被放入宏任务队列，即使是 0ms 的情况。

示例：

```js
console.log('script start')
let promise1 = new Promise(function (resolve) {
    console.log('promise1')
    resolve()
    console.log('promise1 end')
}).then(function () {
    console.log('promise2')
})
setTimeout(function(){
    console.log('settimeout')
})
console.log('script end')

// 输出顺序
1.script start
2.promise1
3.promise1 end
4.script end
5.promise2
6.setimeout
```



async 函数返回的是一个 Promise 对象，在遇到 await 之前是同步执行的，在遇到 await 的时候，会让出主线程，阻塞后面代码的执行。

> 可以理解为 await 之前的代码相当于 Promise 中 resolve 之前的同步代码， await 之后的代码相当于 Promise.then() 里面的回调

async 函数需要等待 await 后的函数执行完成并且有了返回结果（Promise 对象）之后，才能继续执行下面的代码。await 通过返回一个 Promise 对象来实现同步的效果。

来看 async/await，promise，setTimeout 组合起来的执行顺序：

```js
async function async1() {
    console.log('async1 start');
    await async2();
    console.log('async1 end');
}
async function async2() {
    console.log('async2');
}

console.log('start');
setTimeout(() => {
    console.log('setTimeout');
}, 0);
async1();
new Promise(resolve => {
    console.log('promise1');
    resolve();
}).then(() => {
    console.log('promise2');
})
console.log('end');

// 输出顺序
1. start
2. async1 start
3. async2
4. promise1
5. end
6. async1 end
7. promise2
8. setTimeout
```

### 对 Promise 的理解

**Promise 是异步编程的一种解决方案**，最早由社区提出。Promise 是一个构造函数，接收一个回调函数作为参数，返回一个 Promise 实例。

一个 Promise 实例有三种状态

- Pending（进行中）
- Resolved（已完成）
- Rejected（已拒绝）

当把一件事情交给 Promise 时，它的状态就是 Pending，任务完成了状态就变成了 Resolved，没有完成失败了就变成了 Rejected。

Promise 实例有两个过程

- pending -> fulfilled : Resolved（已完成）
- pending -> rejected：Rejected（已拒绝）

实例的状态只能由 pending 转变 resolved 或者 rejected 状态，并且状态一经改变，就凝固了，无法再被改变了。

状态的改变是通过 resolve() 和 reject() 函数来实现的，可以在异步操作结束后调用这两个函数改变 Promise 实例的状态，它的原型上定义了一个 then 方法，使用这个 then 方法可以为两个状态的改变注册回调函数。这个回调函数属于微任务，会在本轮事件循环的末尾执行。

> **注意：** 在构造 `Promise` 的时候，构造函数内部的代码是立即执行的

### Promise 的基本用法

Promise 对象可以使用 `new Promise` 的形式创建，也可以使用 `Promise.resolve(value)` 或 `Promise.reject(value)` 的形式创建。

> `Promise.resolve(x)` 可以看作是 `new Promise(resolve => resolve(x))` 的简写，可以用于快速封装字面量对象或其他对象，将其封装成 Promise 实例。

Promise 有五个常用的方法：then()、catch()、finally、Promise.all()、Promise.race()。

catch 方法相当于 `then` 方法的第二个参数，指向 `reject` 的回调函数。不过`catch`方法还有一个作用，就是在执行`resolve`回调函数时，如果出现错误，抛出异常，不会停止运行，而是进入`catch`方法中。

Promise.all 可以完成并行任务， 它接收一个数组，数组的每一项都是一个 `promise` 对象。当数组中所有的 `promise` 状态都达到 `resolved` 的时候，Promise.all 方法的状态就会变成 `resolved`，如果有一个状态变成了 `rejected`，那么 Promise.all 方法的状态就会变成 `rejected`。

Promise.race 与 all 方法一样，只不过是当最先执行完的事件执行完之后，就直接返回该 `promise` 对象的值。如果第一个 Promise 对象状态变成 `resolved`，那自身的状态变成了 `resolved`；反之第一个 Promise 变成 `rejected`，那自身状态就会变成 `rejected`。

### Promise解决了什么问题

如果后一个请求需要依赖于前一个请求成功后，将数据往下传递，那么会导致多个 ajax 请求嵌套的情况，代码不够直观。

Promise 的链式调用，解决了回调地狱的问题。

### 对 async/await 的理解

async/await 其实是 `Generator` 的语法糖，它能实现的效果都能用 then 的链式调用来实现，它是为优化 then 的链式调用而开发出来的。从字面上来看，async 是“异步”的简写，await 则为等待，所以很好理解 async 用于申明一个 function 是异步的，而 await 用于等待一个异步方法执行完成。当然语法上强制规定 await 只能出现在 asnyc 函数中。

### await 到底在等待什么

await 表达式的运算结果取决于它等待的是什么。

- 如果它等待的不是一个 Promise 对象而是一个任意表达式，那 await 表达式的运算结果就是它等待的这个表达式的返回结果。
- 如果它等待的是一个 Promise 对象，那 await 就忙起来了，它会阻塞后面的代码，等着 Promise 对象 resolve，然后得到 resolve 的值，作为 await 表达式的运算结果。

看一个示例：

```javascript
function testAsy(x){
   return new Promise(resolve => { setTimeout(() => {
         resolve(x);
       }, 3000)
     }
   )
}
async function testAwt(){    
  let result = await testAsy('hello world');
  console.log(result);
  console.log('cuger')
}
testAwt();
console.log('cug')

// 立即输出 cug
// 3秒钟之后出现 hello world
// 3秒钟之后出现 cuger
```

await 暂停当前 async 的执行，所以 cug 最先输出，hello world 和 cuger 是3秒钟后同时出现的。

这就是 await 必须用在 async 函数中的原因。**因为 async 是异步函数，它的调用不会造成主线程同步代码的阻塞**，它内部所有的阻塞都被封装在一个 Promise 对象中异步执行。

### async/await 相对于 Promise 的优势

- 阅读友好；代码同步执行，比起多个链式调用阅读起来更友好
- 传递中间值友好；多个链式调用时，Promise 传递中间值比较麻烦，而 async/await 是同步的写法，非常优雅
- 错误处理友好，async/await 可以使用成熟的 try catch，promise 的错误捕获会导致多个链式调用，很冗余
- 调试友好；调试器只能跟踪同步代码的每一步，如果在 .then 处使用调试器的步进（step-over）功能，调试器并不会进入后续的 .then 代码块



## 10. 面向对象

### 对象创建的方式有哪些

1. 工厂模式
2. 构造函数模式
3. 原型模式
4. 组合使用构造函数和原型模式
5. 动态原型模式
6. 寄生构造函数模式

### 🔴 对象继承的方式有哪些

1. 原型链的方式

   ```js
   // 父类
   function Person (name) {
     this.name = name
     this.sum = () => { alert(this.name) }
   }
   Person.prototype.age = 10 // 原型属性
   
   function Per () {}
   Per.prototype = new Person() // 重点：让新实例的原型指向父类的实例
   ```

   > 缺点：
   >
   > - 在包含有引用类型的数据时，会被所有的实例对象所共享
   > - 在创建子类型的时候不能向父类构造函数传递参数

2. 借用构造函数的方式

   ```js
   // 父类
   function Person (name) {
     this.name = name
     this.sum = () => { alert(this.name) }
   }
   Person.prototype.age = 10 // 原型属性
   
   function Con () {
     Person.call(this. 'jer') // 重点：在子类中通过.call()或.apply()调用父类的构造函数来实现，将父类构造函数绑定在子类构造函数上
   }
   
   const con1 = new Con()
   console.log(con1.name) // jer
   console.log(con1.age) // 12
   ```

   > 优点：
   >
   > - 不会共享引用类型的属性
   > - 可以向父类构造函数传递参数
   >
   > 缺点：
   >
   > - 只能继承父类构造函数的属性，不能继承父类原型上的属性
   > - 无法实现父类构造函数的复用，相同的函数在每个实例中都有一份

3. 组合继承（将原型链与构造函数组合起来）

   ```js
   // 父类
   function Person (name) {
     this.name = name
     this.sum = () => { alert(this.name) }
   }
   Person.prototype.age = 10 // 原型属性
   
   // 重点：结合了两种模式的优点，传参和复用
   function SubType (name) {
     Person.call(this, name) // 借用构造函数模式
   }
   SubType.prototype = new Person() // 借用原型链模式
   SubType.prototype.constructor = SubType
   
   const sub = new SubType('gar')
   console.log(sub.name) // 'gar'
   console.log(sub.age) // 10
   ```

   > 优点：
   >
   > - 不会共享引用类型的属性
   > - 可以向父类构造函数传递参数
   > - 可以复用父类构造函数
   >
   > 缺点：
   >
   > - 需要调用两次父类构造函数（耗内存）
   > - 子类的构造函数会代替原型上的那个父类构造函数，造成了子类型的原型中多了很多不必要的属性

4. 原型式继承

   ```js
   // 父类
   function Person (name) {
     this.name = name
     this.sum = () => { alert(this.name) }
   }
   Person.prototype.age = 10 // 原型属性
   
   // 用一个函数包装一个对象，然后返回这个函数的调用，这个函数就变成了个可以随意增添属性的实例或对象。object.create()就是这个原理。
   function object (obj) {
     function F () {}
     F.prototype = obj
     return new F()
   }
   
   const sup = new Person()
   const sup1 = object(sup) // Object.create(sup)
   console.log(sup1.age) // 10
   ```

   > 缺点：
   >
   > - 与原型链方式相同

   在 ECMAScript5 中新增了 Object.create() 方法，该方法规范了原型式继承。这个方法接收两个参数：一个用作新对象原型的对象，另一个是可选的，用于新对象定义额外的属性的对象。在只传入一个参数的情况下，Object.create() 与上面的 object() 方法行为相同。

5. 寄生式继承

   ```js
   // 父类
   function Person (name) {
     this.name = name
     this.sum = () => { alert(this.name) }
   }
   Person.prototype.age = 10 // 原型属性
   
   function object (o) {
     function F () {}
     F.prototype = o
     return new F()
   }
   
   function createAnother (original) {
     const clone = object(original) // 通过调用函数创建一个新对象
     clone.name = 'clone'
     return clone // 返回这个对象
   }
   
   const sup = new Person()
   const sup2 = createAnother(sup)
   console.log(sup2.name) // clone
   ```

   > 优点：
   >
   > - 对一个简单对象实现继承，如果这个对象不是自定义类型时
   >
   > 缺点：
   >
   > - 未用到原型，做不到函数复用

6. 寄生式组合继承

   ```js
   function Parent (name) {
     this.name = name
     this.colors = ['red', 'blue', 'green']
   }
   Parent.prototype.getName = function () {
     console.log(this.name)
   }
   
   function Child (name, age) {
     Parent.call(this, name) // 继承了父类构造函数的属性
     this.age = age
   }
   
   // 重点
   // function object (obj) {
   //   function F () {}
   //   F.prototype = obj
   //   return new F()
   // }
   // Child.prototype = object(Parent.prototype)
   // 等价于
   Child.prototype = Object.create(Parent.prototype)
   Child.prototype.constructor = Child
   
   const child1 = new Child('kevin', '18')
   
   console.log(child1)
   ```

   > 优点：
   >
   > - 使用父类型的原型的副本作为子类型的原型，这样就避免了创建不必要的属性。

## 11. 垃圾回收与内存泄漏

### 垃圾回收的概念

#### 垃圾回收

JavaScript 代码运行时，需要分配内存空间来储存变量和值。当变量不再参与运行时，就需要系统收回被占用的内存空间，这就是垃圾回收。

#### 回收机制

JS 引擎中有一个后台进程称为垃圾回收器，它监视所有对象，观察对象是否可被访问，然后按照固定的时间间隔周期性的删除掉那些不可访问的对象即可。



### 垃圾回收的方式

现在各大浏览器通常用采用的垃圾回收方式有两种：

#### 引用计数

最早最简单的垃圾回收机制，就是给一个占用物理空间的对象附加一个引用计数器，当有其它对象引用这个对象时，这个对象的引用计数加 1，反之解除时就减 1，当该对象引用计数为 0 时就会被回收。

该方式很简单，但会引起内存泄漏：

```js
// 循环引用的问题
function temp () {
    const obj1 = {}
    const obj2 = {}
    obj1.a = obj2 // obj1 引用 obj2
    obj2.a = obj1 // obj2 引用 obj1
}
```

这种情况下每次调用 `temp` 函数，`obj1` 和 `obj2` 的引用计数都是 `2` ，会使这部分内存永远不会被释放，即内存泄漏。现在已经很少使用了，只有低版本的 IE 使用这种方式。

> 只能手动释放变量占用的内存：
>
> ```js
> obj1.a = null
> obj2.a = null
> ```

#### 标记清除

V8 引擎中主垃圾回收器就采用标记清除法进行垃圾回收。主要流程如下：

1. 在变量进入执行上下文时打上“进入”标记
2. 同时在变量离开执行上下文时也打上“离开”标记
3. 标记为“离开”的变量，从此以后是无法访问这个变量的，在下一次垃圾回收时该变量占用的内存将会被释放

> 所以我们需要尽可能的使用 const 和 let，因为 const 和 let 使 JS 有了块级作用域，当块级作用域比函数作用域更早结束时，垃圾回收器可以更早介入。

![mark and sweep](https://raw.githubusercontent.com/aboutcroon/Notes/main/JavaScript/interview/assets/mark%20and%20sweep.webp)

> 拓展
>
> 如果一个对象被多次引用时，例如作为另一对象的键、值或子元素时，将该对象引用设置为 `null` 时，该对象是不会被回收的，依然存在。例如：
>
> ```javascript
> let a = {}
> let arr = [a]
> 
> a = null;
> console.log(arr)
> // [{}]
> ```
>
> 如果作为 `Map` 的键呢？则也是如此，ES6 考虑到了这一点，推出了： `WeakMap` 。它对于值的引用都是不计入垃圾回收机制的，所以名字里面才会有一个"Weak"，表示这是弱引用（对对象的弱引用是指当该对象应该被GC回收时不会阻止GC的回收行为）。
>
> `Map` 相对于 `WeakMap` ：
>
> - `Map` 的键可以是任意类型，`WeakMap` 只接受对象作为键（null除外），不接受其他类型的值作为键。
> - `Map` 的键实际上是跟内存地址绑定的，只要内存地址不一样，就视为两个键； `WeakMap` 的键是弱引用，键所指向的对象可以被垃圾回收，**此时键是无效的**。
> - `Map` 可以被遍历， `WeakMap` 不能被遍历。

### 如何减少垃圾回收

当代码比较复杂时，浏览器进行垃圾回收的代价会变得比较大，所以应该尽量减少垃圾回收。

- 数组优化：在清空一个数组时，我们可以尽量将数组长度设为 0，`arr.length = 0`，而不是 `arr = []`，这样会创建一个新的空对象，对垃圾回收不友好。
- 对象优化：对象尽量复用，对于不再使用的对象，将其设置为 null 使之尽快被回收。
- 函数优化：在循环中的函数表达式，如果有可以复用的部分，尽量放在函数的外面。

### 哪些情况会导致内存泄漏

1. **意外的全局变量**
2. **被遗忘的计时器或回调函数**：设置了 setInterval 定时器，而忘记取消它，如果定时器的回调函数有对外部变量的引用的话，那么这个变量会被一直留在内存中，而无法被回收。
3. **脱离 DOM 的引用**：获取一个 DOM 元素的引用，而后这个 DOM 元素被删除，那么由于一直保留了对这个元素的引用，那么它也将无法被回收。
4. **不合理的闭包**：使用闭包的方式不合理时，将导致某些变量一直被引用从而一直无法被回收。



