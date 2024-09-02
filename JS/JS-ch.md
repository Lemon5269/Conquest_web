<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [深拷贝浅拷贝](#%E6%B7%B1%E6%8B%B7%E8%B4%9D%E6%B5%85%E6%8B%B7%E8%B4%9D)
- [Promise 概述 + 手写 Promise](#promise-%E6%A6%82%E8%BF%B0--%E6%89%8B%E5%86%99-promise)
- [手写 Promise.all 和 Promise.allSettled](#%E6%89%8B%E5%86%99-promiseall-%E5%92%8C-promiseallsettled)
- [Promise 经典考题](#promise-%E7%BB%8F%E5%85%B8%E8%80%83%E9%A2%98)
- [数组常用方法](#%E6%95%B0%E7%BB%84%E5%B8%B8%E7%94%A8%E6%96%B9%E6%B3%95)
- [数组去重常用的方法](#%E6%95%B0%E7%BB%84%E5%8E%BB%E9%87%8D%E5%B8%B8%E7%94%A8%E7%9A%84%E6%96%B9%E6%B3%95)
- [数组扁平化](#%E6%95%B0%E7%BB%84%E6%89%81%E5%B9%B3%E5%8C%96)
- [数组乱序（shuffle）](#%E6%95%B0%E7%BB%84%E4%B9%B1%E5%BA%8Fshuffle)
- [函数柯里化](#%E5%87%BD%E6%95%B0%E6%9F%AF%E9%87%8C%E5%8C%96)
- [instanceof 和 typeof 的原理](#instanceof-%E5%92%8C-typeof-%E7%9A%84%E5%8E%9F%E7%90%86)
- [js 实现继承的 5 种方法](#js-%E5%AE%9E%E7%8E%B0%E7%BB%A7%E6%89%BF%E7%9A%84-5-%E7%A7%8D%E6%96%B9%E6%B3%95)
- [手写 reduce](#%E6%89%8B%E5%86%99-reduce)
- [手写 map](#%E6%89%8B%E5%86%99-map)
- [new 操作符](#new-%E6%93%8D%E4%BD%9C%E7%AC%A6)
- [xhrfetchaxios](#xhrfetchaxios)
- [JS 垃圾回收机制](#js-%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6%E6%9C%BA%E5%88%B6)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## 深拷贝浅拷贝

**实现浅拷贝的常用方法**：只能拷贝第一层，深层次无法实现

- `Object.Assign`
- 解构赋值

```js
let obj = {
  name: "brave",
  friends: {
    name: "beibei",
    age: "13",
  },
};
// 方法一：解构赋值
const newObj2 = { ...obj };
// 方法二：Object.assign
const newObj3 = Object.assign({}, obj);
```

**实现深拷贝的常用方法**

- 使用 JSON 完成

```js
let obj = {
  name: "brave",
  friends: {
    name: "beibei",
    age: "13",
  },
};
let newObj = JSON.parse(JSON.stringfy(obj));
console.log(obj === newObj); // false
```

- 手写深拷贝（解决了循环引用和函数拷贝的问题）

```js
function isObject(value) {
  const valueType = typeof value;
  return value !== null && (valueType === "object" || valueType === "function");
}
function deepClone(originValue, visited = new WeakMap()) {
  if (!isObject(originValue)) {
    return originValue;
  }
  if (typeof originValue === "function") {
    return eval(`(${originValue.toString()})`);
  }
  if (visited.get(originValue)) return originValue;
  let newValue = Array.isArray(origin) ? [] : {};
  visited.set(originValue, newValue);
  for (let key in originValue) {
    newValue[key] = deepClone(originValue[key], visited);
  }
  return newValue;
}
const obj = {
  num: 0, // number
  str: "", // string
  bool: true, // boolean
  unf: undefined, // undefined
  nul: null, // null
  sym: Symbol("sym"), // symbol
  bign: BigInt(1n), // bigint,
  arr: [1, 2, 3, 4],
  test: function () {
    console.log("2312");
  },
};
let obj2 = {
  name: "111",
};
obj.obj2 = obj2;
obj2.obj = obj;
// 开头的测试obj存在循环引用，除去这个条件进行测试
const clonedObj = deepClone(obj);

// 测试
console.log(clonedObj === obj); // false，返回的是一个新对象
console.log(clonedObj.arr === obj.arr); // false，说明拷贝的不是引用
console.log(clonedObj.test === obj.test); // false
```

## Promise 概述 + 手写 Promise

Promise 是 es6 新增的语法，解决了回调地狱的问题。
可以把 Promise 看成一个状态机。初始是`pending`状态，可以通过函数`resolve`和`reject`,将状态转变为 `resolve`或者`rejected`状态，状态一旦改变就不能再次变化。

`.then和.catch`用于处理回调，`.then`中会接收两个函数，第一个函数是成功的回调函数，第二个参数是失败的回调函数。`.catch`中只能接收一个参数就是失败的回调函数。`.catch`一般是放在最后，可以解决一个 Promise 异常穿透的问题。

- 手写 Promise

```js
class MyPromise{
    constructor(executor){
        // 初始化值
        this.initValue();
        // 初始化this指向 this指向永远是当前MyPromise的实例
        this.initBind();
        try{
            // 执行传进来的函数
            executor(this.resolve, this.reject);
        }catch(e){
            // 捕捉到错误执行执行reject
            this.reject(e)
        }
    }
    // 初始化this
    initBind(){
        this.resolve = this.resolve.bind(this);
        this.reject = this.reject.bind(this);
    }
    //初始化值
    initValue(){
        this.PromiseResult = null; // 终值
        this.PromiseState = 'pending'; // 状态
        this.onFulfilledCallbacks = []; // 保存成功回调
        this.onRejectedCallbacks = []; // 保存失败回调
    }

    resolve(value){
        // 状态一旦改变是不可以再次改变的
        if(this.PromiseState !== 'pending') return;
        // 如果执行resolve，状态变为fulfilled
        this.PromiseState = 'fulfilled';
        // 终值为传进来的值
        this.PromiseResult = value;
        // 执行保存的成功回调
        while(this.onFulfilledCallbacks.length){
            this.onFulfilledCallbacks.shift()(this.PromiseResult)
        }
    }
    reject(reason){
        if(this.PromiseState !== 'pending') return;
        // 如果执行reject，状态变为fulfilled
        this.PromiseState = 'rejected';
        // 终值为传进来的值
        this.PromiseResult = reason;
        // 执行保存的失败回调
        while(this.onRejectedCallbacks.length){
            this.onRejectedCallbacks.shift()(this.PromiseResult)

    }
    then(onFulfilled, onRejected){
        // then接收两个回调，一个是成功的回调，一个是失败的回调
        // 判断参数是否是函数
        onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : val=>val;
        onRejected = typeof onRejected === 'function' ? onRejected : reason => { throw reason }

        // then的链式调用
        var thenPromise = new MyPromise((resolve, reject)=>{
            const resolvePromise = cb => {
                // 这里还需要让resolvePromise是一个异步任务
                setTiemout(()=>{
                    try{
                        const x = cb(this.PromiseResult);
                        if(x===thenPromise){
                            // 不能返回自身
                            throw new Error("不能返回自身");
                        }
                        if(x instanceof MyPromise){
                            // 如果返回值是Promise
                            // 如果返回值是promise对象，返回值为成功，新promise就是成功
                            // 如果返回值是promise对象，返回值为失败，新promise就是失败
                            // 谁知道返回的promise是失败成功？只有then知到
                            x.then(resolve, reject);
                        }else{
                            // 非Promise就直接成功
                            resolve(x)
                        }
                    }catch(err){
                        reject(err)
                    }
                })

            }
            if(this.PromiseState === 'fulfilled'){
                // 如果当前状态为成功，则执行第一个回调
                resolvePromise(onFulfilled);
            }else if(this.PromiseState === 'rejected'){
                // 如果当前状态为失败执行失败的回调
                resolvePromise(onRejected);
            }else if(this.PromiseState === 'pending'){
                // 如果状态为待定状态，暂时保存两个回调
                this.onFulfilledCallbacks.push(resolvePromise(this, onFulfilled));
                this.onRejectedCallbacks.push(resolvePromise(this, onRejected));
            }
        })

    }
    // catch方法
    catch(fn){
        return this.then(null, fn);
    }
    // all方法
    static all(it){
        // 对传入的数据做一个浅拷贝，确保有遍历器
        const promises = Array.from(it);
        let data = [];
        let index = 0;
        let len = promises.length;
        return new MyPromise((resolve, reject)=>{
            for(let i in promises){
                promises[i].then(res=>{
                    data[i] = res;
                    if(++index===len){
                        resolve(data);
                    }
                }).catch(err=>{
                    reject(err);
                })
            }
        })
    }

    // race方法
    static race(it){
        const promises = Array.from(it);
        return new MyPromise((resolve, reject)=>{
            promises.forEach(promise=>{
                // 又做了一层检查
                if(promise instanceof MyPromise){
                    promise.then(res=>{
                        resolve(res);
                    }, err=>{
                        reject(err);
                    })
                } else{
                    resolve(promise)
                }
            })
        })
    }

    // any：与all相反
    static any(it){
        const promises = Array.from(it);
        let index = 0;
        let len = promises.length;
        return new Promise((resolve, reject)=>{
            for(let i in promises){
                promises[i].then(res=>{
                    resolve(res);
                }).catch(err=>{
                    if(++index==len){
                        reject(new Error('All promises were rejected'));
                    }
                })
            }
        })
    }

    finally(fn){
        return this.then((res)=>{
            fn()
            return res;
        }).catch(err=>{
            fn();
            return err;
        })
    }
}
```

## 手写 Promise.all 和 Promise.allSettled

`Promise.all`：同时执行多个 Promise 对象（以数组的形式传入）。两种情况，当所有的 Promise 状态为 fulfilled 时，新的 Promise 状态为 fullfilled，并且将所有的 Promise 的返回值组成一个数字；当有一个 Promise 状态为 reject 时，新的 Promise 状态为 reject，并且会将第一个 reject 的返回值作为参数。

**缺点**：当有一个 Promise 变为 reject 状态时，新的 Promise 就会立即变成对应的 reject 状态，对于 resolved 以及依然处于 pending 状态的 Promise，就获取不到对应的结果。
手写 Promise.all

```js
const MyPromiseAll = (iterator) => {
  // 对传入的数据做一个浅拷贝，确保有遍历器
  const promises = Array.from(iterator);
  const len = promises.length;
  let data = []; // 用来存储返回的数据数组
  let index = 0; // 记录data的长度
  return new Promise((resolve, reject) => {
    for (let i in promises) {
      promises[i]
        .then((res) => {
          data[i] = res;
          if (++index === len) {
            resolve(data);
          }
        })
        .catch((err) => {
          reject(err);
        });
    }
  });
};
const promise1 = Promise.resolve("promise1");
const promise2 = new Promise(function (resolve, reject) {
  setTimeout(resolve, 2000, "promise2");
});
const promise3 = new Promise(function (resolve, reject) {
  setTimeout(resolve, 1000, "promise3");
});
PromiseAll([promise1, promise2, promise3]).then(function (values) {
  console.log(values);
});
```

`Promise.allSettled`：该方法会在所有的 Promise 都有结果（无论是 fullfilled 还是 reject）才会有最终状态，并且这个 Promise 的结果一定是 fullfilled。

```js
const myPromiseAllSettled = function (it) {
  // 浅拷贝，确保有迭代器
  const promises = Array.from(it);
  const len = promises.length;
  let data = []; // 返回值
  let index = 0; // 统计
  return new Promises((resolve, reject) => {
    promises[i].then(
      (res) => {
        data[i] = {
          status: "fullfilled",
          value: res,
        };
        if (++index === len) {
          resolve(data);
        }
      },
      (reason) => {
        data[i] = {
          status: "rejected",
          reason,
        };
        if (++index === len) {
          resolve(data);
        }
      }
    );
  });
};
const p = new Promise((resolve, reject) => {
  setTimeout(() => {
    reject(3);
  }, 1000);
});
console.log(
  myPromiseAllSettled([p, Promise.resolve(1), Promise.reject(2)]).then(
    (res) => {
      console.log(res);
    }
  )
);
```

## Promise 经典考题

- 一
  ```js
  new Promise((res, rej) => {
    console.log(1);
    throw new Error("abc");
    res(2);
    console.log(3);
  })
    .catch((e) => {
      console.log("catch");
    })
    .then((t) => {
      console.log(t);
    });
  console.log("end");
  ```
- 二
  ```js
  new Promise((res, rej) => {
    console.log(1);
    res(2);
    throw new Error("abc");
    console.log(3);
  })
    .catch((e) => {
      console.log("catch");
    })
    .then((t) => {
      console.log(t);
    });
  console.log("end");
  ```

## 数组常用方法

- 可以改变原数组的方法
  - push()
  - pop()
  - shift()：移除数组中第一个元素，返回所移除的元素
  - unshift()：向队头插入元素，并返回数组长度
  - splice()
  ```js
  /*使用该方法会影响原数组
  参数1：表示开始位置的索引
  参数2：表示删除的数量
  参数3：传递的新元素
  该方法返回被删除元素组成的数组，并会改变原数组
  splice(i,j, xxx, xxx);
  */
  console.log([1, 2, 3, 4, 5].splice(1, 2, 3, 4, 5));
  // 从位置1开始删除两个元素，并在开始位置后添加元素 3 4 5
  // 2 3？
  ```
  - sort()
  - reverse()
  - fill()
- 不能改变数组的相关方法
  - concat()
  - slice()
  ```js
  /*start 可选，整数，指定从哪里开始选择，使用负数从数组的末尾进行选择，如果省略，则类似 0
  end 可选 整数 指定结束选择的位置，如果省略将选择从开始位置到数组末尾的所有元素，使用负数从数组末尾进行选择，-1就是最后一个元素
  注意包含开始，不包含结束
  该方法返回被提取出来的元素组成的数组，但不会改变原数组
  */
  console.log([1, 2, 3, 4, 5].slice(1, 2, 3, 4, 5));
  // 后面的三个参数是无效的，从位置1开始到位置2前一个元素
  // 2
  ```
  - join()
  - indexOf()
  - includes()
  - filter()
  - map()
  - reduce()
  - forEach()
  - some()
  - every()
  - find()
  - findIndex()

## 数组去重常用的方法

- 方法一：使用 set

```js
const arr = [1, 1, 2, 2, 3, 4];
function unique(arr) {
  return Array.from(new Set(arr));
}
```

- 方法二：使用 reduce + indexOf

```js
const arr = [1, 1, 2, 2, 3, 4];
function unique(arr) {
  const res = arr.reduce((prev, curr) => {
    if (prev.indexOf(curr) === -1) {
      prev.push(curr);
    }
  }, []);
  return res;
}
```

- 方法三：使用 filter + indexOf

```js
const arr = [1, 1, 2, 2, 3, 4];
function unique(arr) {
  const res = arr.filter((item, index) => arr.indexOf(item) === index);
  return res;
}
```

- 方法四：使用 map

```js
const arr = [1, 1, 2, 2, 3, 4];
function unique(arr) {
  let map = new Map();
  let res = [];
  arr.forEach((i) => {
    if (!map.has(i)) {
      map.set(i, 1);
      res.push(i);
    }
  });
  return res;
}
```

## 数组扁平化

将数组拉平的过程

- 方法一：使用递归+concat

```js
const a = [1, [2, [3, [4, 5]]]];
const flatten = (arr) => {
  let result = [];
  for (let i = 0; i < arr.length; i++) {
    if (Array.isArray(arr[i])) {
      result = result.concat(flatten(arr[i]));
    } else {
      result.push(arr[i]);
    }
  }
  return result;
};
console.log(flatten(a));
```

- 方法二：使用扩展运算符

```js
const arrNum = [1, [2, 3], [4, [5, 6, 7]]];
function flatten(arr) {
  // 循环，直到arr中没有数组才跳出循环
  while (arr.some((item) => Array.isArray(item))) {
    arr = [].concat(...arr);
  }
  return arr;
}
console.log(flatten(arrNum)); //  [1, 2, 3, 4，5, 6, 7]
```

## 数组乱序（shuffle）

- Math.floor() 向下取整 2.7 -> 2
- Math.ceil() 向上取整 2.7 -> 3
- Math.round() 四舍五入
- Math.random() 返回介于 0（包括）-1（不包括）之间的随机数

```js
// 思路：从最后一个元素开始，从数组中随机选出一个位置，交换，直到第一个元素
function shuffle(arr) {
  const len = arr.length;
  let curr = len - 1;
  let random;
  while (curr >= 0) {
    random = Math.floor(len * Math.random());
    [arr[curr], arr[random]] = [arr[random], arr[curr]];
    curr--;
  }
  return arr;
}
```

## 函数柯里化

通俗理解：用闭包把参数保存起来，当参数的数量足够执行函数了，就开始执行函数。

- 实现
  - 判断当前函数传入的参数是否大于或等于`fn`需要参数的数量，如果是，直接执行 fn
  - 如果传入参数数量不够，返回一个闭包，暂存传入的参数，并重新返回`currying`函数

```js
function currying(fn, ...args) {
  if (args.length >= fn.length) {
    return fn(...args);
  } else {
    return (...args2) => currying(fn, ...args, ...args2);
  }
}
// 简单验证
function fun(a, b, c) {
  console.log(a, b, c);
}
const curryingFun = currying(fun);
curryingFun(1)(2)(3); // 1 2 3
// curryingFun(1, 2)(3);  // 1 2 3
// curryingFun(1, 2, 3);  // 1 2 3
```

## instanceof 和 typeof 的原理

- instanceof：测试一个对象在其原型链中是否存在一个构造函数的 prototype 属性，简单来说就是通过 **原型链** 去查找。
- typeof：js 在底层存储变量的时候，会在变量的机器码的低位 1-3 位存储其类型信息 - 000 对象 - 010 浮点数 - 100 字符串 - 110 布尔 - 111 整数 - null 所有机器码均为 0 - undefined 用-2^30 整数来表示
  所以 typeof 在判断 null 的时候就出现问题，由于 null 的所有机器码均为 0，因此直接被当作了对象来看。
  **手写函数** 传入任意变量，可准确获取类型，如 number、string、boolean、object、array、map、regexp

```js
function getType(x) {
  const originType = Object.prototype.toString.call(x);
  const spaceIndex = originType.indexOf(" ");
  const type = originType.slice(spaceIndex + 1, -1);
  return type.toLowerCase();
}
```

## js 实现继承的 5 种方法

**父类**

```js
function Father(name) {
  this.name = name || "father";
  // 实例方法
  this.sayName = function () {
    console.log(this.name);
  };
  this.color = ["red", "blue"];
  // 原型方法
  Father.prototype.age = 18;
  Father.prototype.sayAge = function () {
    console.log(this.age);
  };
}
```

- 方法一：原型链继承：将父类的实例作为子类的原型

  ```js
  function Son(name) {
    this.name = name || "son";
  }
  Son.prototype = new Father();
  let s1 = new Son("s1");
  let s2 = new Son("s2");
  s1.color.push("black");
  console.log(s1.name); //s1
  console.log(s1.color); //['red','blue','black']
  console.log(s1.age); //18
  s1.sayAge(); //18
  console.log(s2.name); //s2
  console.log(s2.color); //['red','blue','black']
  ```

  优点：简单、易于实现；父类新增原型方法、属性，子类都能访问到

  缺点：

        - 创建子类实例时，无法向父类构造函数传参

        - 无法实现多继承，因为原型一次只能被一个实例更改

        - 来自原型对象的所有属性被所有实例共享

- 方法二：借助构造函数实现继承，复制父类的实例属性给子类
  ```js
  function Son(name, hobby){
  Father.call(this, name)
      this.hobby = hobby
  }
  let s = new Son('brave', 'play')
  console.log(s.name); // brave
  //s.sayAge(); // 抛出错误（无法继承父类原型方法）
  s.sayName(); // brave
  console.log(s.age); // undefined （无法继承父类原型属性）
  console.log(s instanceof Father); // false
  console.log(s instanceof Son); // true
  //--------------------------------------
  优点：
  1. 解决了原型链继承中子类实例无法向父类构造函数传参的问题
  2. 可以实现多继承 call多个父类对象
  缺点：
  1. 只能继承父类实例的属性和方法，不能继承其原型上的属性和方法
  2. 实例并不是父类的实例而是子类的实例
  ```
- 方法三：组合继承，组合继承：将原型链和借用构造函数的技术组合到一起，使用原型链实现对原型属性和方法的继承，通过构造函数来实现对实例属性的继承
  ```js
  function Son(name, hobby){
      Father.call(this, name)
  }
  Son.prototype = new Father()
  let s = new Son("son");
  console.log(s.name); // son
  s.sayAge(); // 18
  s.sayName(); // son
  console.log(s.age); // 18
  console.log(s instanceof Father); // true
  console.log(s instanceof Son); // true
  console.log(s.constructor === Father); // true
  console.log(s.constructor === Son); // false
  //---------------------------------------
  优点：
  1. 既是子类的实例，也是父类的实例
  2. 可以继承实例的属性和方法，也可以继承原型的属性和方法
  3. 可以向父类传递参数
  缺点：
  1. 调用了两次父类构造函数，生成了两份实例
  2. constructor指向问题
  ```
- 方法四：寄生式组合继承，通过寄生方式，砍掉父类的实例属性，避免了组合继承生成两份实例的缺点

  ```js
  // 创建对象的过程
  // 手动创建一个中间类，或者也可以借用Object.create()方法
  function createObject(o){
      function F(){}
      F.prototype = o
      return new F()
  }

  // 将Subtype和Supertype联系在一起
  // 寄生式函数
  function inherit(Subtype, Supertype){
      //其实不用这么麻烦，可以直接使用Object.create
      Subtype.prototype = createObject(Supertype.prototype)
      Object.defineProperty(Subtype.prototype, "constructor", {
          enumerable: true,
          configurable: true,
          writable: true,
          value: Subtype
      })
      //===================================
      // 上述代码的另一种写法
      Subtype.protptype = Object.create(Supertype.prototype, {
      constructor: {
          value: Subtype,
          enumerable: false,
          writable: true,
          configurable: true
      }
      })
  }
  //-----------------------------------------
  优点
  1. 比较完美(js实现继承的首选方式)
  缺点：
  实现起来比较复杂
  ```

- 方法五：es6——class 继承：使用 extends 表面继承自哪个父类，并且在子类的构造函数中必须调用 super

  ```js
  class Son extends Father {
    constructor(name) {
      super(name);
      this.name = name || "son";
    }
  }

  let s = new Son("son");
  console.log(s.name); // son
  s.sayAge(); // 18
  s.sayName(); // son
  console.log(s.age); // 18
  console.log(s instanceof Father); // true
  console.log(s instanceof Son); // true
  console.log(s.constructor === Father); // false
  console.log(s.constructor === Son); // true
  ```

## 手写 reduce

```js
Array.prototype.MyReduce = function (fn, init) {
  let arr = this;
  let prev;
  if (typeof fn !== "function") {
    throw new Error("fn must be a function");
  }
  if (!init) {
    for (let i = 0; i < arr.length; i++) {
      prev = fn(prev, arr[i], i, arr);
    }
  } else {
    prev = init;
    for (let i = 0; i < arr.length; i++) {
      prev = fn(prev, arr[i], i, arr);
    }
  }
  return prev;
};
// 注意：此处不能用箭头函数！！
```

## 手写 map

```js
Array.prototype.MyMap = function (fn) {
  if (typeof fn !== "function") {
    throw new Error("fn must be function");
  }
  let arr = this;
  for (let i = 0; i < arr.length; i++) {
    arr[i] = fn(arr[i], i, arr);
  }
  return arr;
};
// 注意：此处不能用箭头函数！！
```

## new 操作符

- 创建一个空对象
- 将空对象赋值给 this
- 将函数的显式原型赋值给这个对象作为它的隐原型
- 执行函数体中的代码
- 将该对象默认返回
  `代码：手写new操作符`

```js
//Fun为构造函数, args表示传参
function myNew(Fun, ...args) {
  // 1.在内存中创建一个新对象
  let obj = {};

  // 2.把新对象的原型指针指向构造函数的原型属性
  obj.__proto__ = Fun.prototype;

  // 3.改变this指向，并且执行构造函数内部的代码（传参）
  let res = Fun.apply(obj, args);

  // 4.判断函数执行结果的类型
  if (res instanceof Object) {
    return res;
  } else {
    return obj;
  }
}

let obj = myNew(One, "XiaoMing", "18");
console.log("newObj:", obj);
```

## xhrfetchaxios

xhr：原生的一种与服务端进行数据交换的请求方式，通过创建 xhr 对象，设置请求参数，注册回调等多步来完成网络请求。
好处：不重新加载页面的情况下更新网页 在页面已加载后从服务器请求/接收数据 在后台向服务器发送数据
缺点：使用起来比较繁琐，需要设置很多值

```js
const xhr = new XMLHttpRequest();
xhr.open('POST', url, true);
xhr.send(data)
xhr.onreadystatechange = function(){
	....
}
```

fetch：也是一种原生的请求方式，基于标准 Promise 实现，支持 async/await，可以通过.then()或者 await 来获取响应结果；更加底层，提供丰富的 API（request，response）。
二者的一个对比：

- 语法和用法：XHR 使用的是回调函数的方式进行异步操作，通过创建 XHR 对象、设置请求参数、注册回调函数等步骤来完成网络请求。而 fetch 使用的是 Promise 和 async/await 的方式进行异步操作，通过调用 fetch 函数并使用.then()或 await 来获取响应结果。
- 请求和响应对象：XHR 在请求和响应的过程中使用的是 XHR 对象，可以通过该对象访问请求和响应的相关信息，如请求头、响应头等。而 fetch 使用的是 Request 和 Response 对象，这两个对象提供了更多的功能和属性，如请求方法、请求体、响应类型等。
- 跨域请求处理：XHR 在发送跨域请求时，需要通过设置相应的请求头（如设置 Access-Control-Allow-Origin 头）或使用代理服务器等方式来解决跨域问题。而 fetch 在发送跨域请求时，默认是不发送 cookie 和身份验证信息的，需要手动设置 credentials 属性为"include"来启用跨域携带 cookie。
- 错误处理：XHR 的错误处理需要在回调函数中通过判断状态码来确定请求是否成功，并进行相应的处理。而 fetch 在请求遇到网络错误时会认为请求失败，返回一个 rejected 状态的 Promise，需要通过.catch()方法来捕获错误。

## JS 垃圾回收机制

关于 JS 垃圾回收机制可以参考：https://juejin.cn/post/6981588276356317214

- 内存溢出：内存溢出是一种程序运行出现的错误；当程序运行需要的内存超过了剩余的内存时，就会爆出内存溢出的错误。
- 内存泄漏：一些变量占用的内存没有及时释放；通常内存泄漏积累多了就会造成内存溢出；常见的内存泄漏就是一些变量、定时器或者回调函数、闭包这些。

**JS 垃圾回收机制：** JS 的解释器可以检测到什么时候程序不再使用这个对象了，就会把它所占用的内存释放掉。常用的垃圾回收机制有两种：标记清除（现代）、引用计数（之前）

- 标记清除：该方法分为两个阶段，标记阶段和清除阶段。标记阶段即为所有活动对象坐上标记，运行时将内存中的所有变量标记为 0，然后从各个根对象开始遍历，将非垃圾遍历标记为 1；清除阶段则把所有标记为 0 的变量内存释放（也就是非活动对象）销毁。最后把内存中对象标记修改为 0， 等待一下轮垃圾回收
  - 优点：实现比较简单，打标记也无非打与不打两种情况，这是的一位二进制位（0 和 1）就可以为其标记，非常简单。
  - 缺点——内存碎片化：通过标记清除之后，剩下没有被释放的对象在内存中的位置是不变的，这就回导致空闲内存是不连续的。解决方法（Mark-Compact）：标记之后使用标记整理算法，将活着的对象向内存一端移动，然后清理掉边界的内存。
- 引用计数：把对象是否不再需要简化定义为对象有没有其他对象引用到它。如果没有引用指向该对象（引用计数为 0），对象被垃圾回收机制回收。会产生如下问题：
  - 计数器：需要一个计数器，所占内存空间大，因为不知道具体的引用梳理
  - 循环引用无法回收的问题

**关于 V8 的垃圾回收机制：**

- 分代式垃圾回收
  上述垃圾清理算法在每次垃圾回收时都要检查内存中所有的对象，这样对于一些大、老、存活时间长的对象来说同新、小、存活时间短的对象一个频率的检查不是很好，因为前者需要时间长并且不需要频繁进行清理，后者恰好相反，所以引出了分代式
  V8 中将堆内存分为新生代和老生代两区域，采用不同的垃圾回收器（不同的策略）管理垃圾回收
- 新老生代
  - 新生代：其中对象为存活时间较短的对象。通过支持 1-8M 的容量（新生代垃圾回收器）
  - 老生代：其中为存活时间较长或常驻内存的对象（老生代垃圾回收器）
- 新生代垃圾回收器
  其采用一种叫 **Scavenge 的算法** 进行垃圾回收，具体内容是将堆内存一分为二，一个是处于使用状态的空间称之为使用区，一个是处于闲置状态的空间成为空间区，新加入的对象都会放到使用区，当使用区快被写满时，就需要执行一次垃圾清理操作。当开始进行垃圾回收时，新生代垃圾回收器会对使用区中的活动对象做标记，标记完成之后将使用区的活动对象复制进空闲区并进行排序，随后进入垃圾清理阶段，即将非活动对象占用的空间清理掉。最后进行角色互换，把原来的使用区变成空闲区，把原来的空闲区变成使用区，当一个对象经过多次复制后依然存活，它将会被认为是生命周期较长的对象，随后会被移动到老生代中，采用老生代的垃圾回收策略进行管理
- 老生代垃圾回收器
  其整个流程采用标记清除算法，首先是标记阶段，从一组根元素开始，递归遍历这组根元素，遍历过程中能到达的元素称为活动对象，没有到达的元素就可以判断为非活动对象。清除阶段老生代垃圾回收器会直接将非活动对象，也就是数据清理掉，也是基于标记清除算法，不过对其做了一些优化。主要就是为了减少全停顿的时间，全停顿主要就是因为垃圾回收靠主线程的话，就会阻塞后序 js 脚本的执行 - 针对新生代采用并行回收，垃圾回收器在主线程上执行的过程中，开启多个辅助线程，同时执行同样的回收工作（并行回收依然会阻塞主线程） - 针对老生代采用增量标记与惰性回收 - 增量标记：就是将一次 GC 标记的过程，分成了很多小步，每执行完一小步就让应用逻辑执行一会儿，这样交替多次后完成一轮 GC 标记 - 惰性回收：当增量标记完成后，假如当前的可用内存足以让我们快速的执行代码，其实我们是没必要立即清理内存的，可以将清理过程稍微延迟一下，让 JavaScript 脚本代码先执行，也无需一次性清理完所有非活动对象内存，可以按需逐一进行清理直到所有的非活动对象内存都清理完毕，后面再接着执行增量标记 - 使用二者的优点：主线程的停顿时间大大减少让用户与浏览器交互的过程变得更加流畅。但是由于每个小的增量标记之间执行了 JavaScript 代码，堆中的对象指针可能发生了变化，需要使用写屏障技术来记录这些引用关系的变化，所以增量标记缺点也很明显：首先是并没有减少主线程的总暂停的时间，甚至会略微增加，其次由于写屏障机制的成本，增量标记可能会降低应用程序的吞吐量（吞吐量是啥总不用说了吧）
