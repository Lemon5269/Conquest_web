<!-- vscode-markdown-toc -->
* 1. [深拷贝浅拷贝](#)
* 2. [Promise概述 + 手写Promise](#PromisePromise)
* 3. [手写Promise.all和Promise.allSettled](#Promise.allPromise.allSettled)
* 4. [Promise经典考题](#Promise)
* 5. [数组常用方法](#-1)
* 6. [数组去重常用的方法](#-1)
* 7. [数组扁平化](#-1)
* 8. [数组乱序（shuffle）](#shuffle)
* 9. [函数柯里化](#-1)
* 10. [instanceof和typeof的原理](#instanceoftypeof)
* 11. [js实现继承的5种方法](#js5)

<!-- vscode-markdown-toc-config
	numbering=true
	autoSave=true
	/vscode-markdown-toc-config -->
<!-- /vscode-markdown-toc -->
##  1. <a name=''></a>深拷贝浅拷贝
__实现浅拷贝的常用方法__：只能拷贝第一层，深层次无法实现
- `Object.Assign`
- 解构赋值
```js
let obj = {
    name: 'brave',
    friends: {
        name: 'beibei',
        age: '13'
    }
}
// 方法一：解构赋值
const newObj2 = {...obj};
// 方法二：Object.assign
const newObj3 = Object.assign({}, obj);
```
__实现深拷贝的常用方法__
- 使用JSON完成
```js
let obj = {
    name: 'brave',
    friends: {
        name: 'beibei',
        age: '13'
    }
}
let newObj = JSON.parse(JSON.stringfy(obj))
console.log(obj === newObj); // false
```
- 手写深拷贝（解决了循环引用和函数拷贝的问题）
```js
function isObject(value) {
  const valueType = typeof value;
  return (value !== null) && (valueType === "object" || valueType === "function");
}
function deepClone(originValue, visited = new WeakMap()) {
  if(!isObject(originValue)) {
      return originValue;
  };
  if(typeof originValue === 'function'){
      return eval(`(${originValue.toString()})`)
  }
  if(visited.get(originValue)) return originValue;
  let newValue = Array.isArray(origin) ? []:{};
  visited.set(originValue, newValue);
  for(let key in originValue){
      newValue[key] = deepClone(originValue[key], visited);
  }
  return newValue
}

// 测试的obj对象
const obj = {
  // =========== 1.基础数据类型 ===========
  num: 0, // number
  str: '', // string
  bool: true, // boolean
  unf: undefined, // undefined
  nul: null, // null
  sym: Symbol('sym'), // symbol
  bign: BigInt(1n), // bigint,
  arr: [1, 2, 3, 4],
  test: function(){
      console.log('2312')
  }
};
let obj2 = {
  name: '111'
}
obj.obj2 = obj2;
obj2.obj = obj;
// 开头的测试obj存在循环引用，除去这个条件进行测试
const clonedObj = deepClone(obj)

// 测试
console.log(clonedObj === obj)  // false，返回的是一个新对象
console.log(clonedObj.arr === obj.arr)  // false，说明拷贝的不是引用
console.log(clonedObj.test===obj.test) // false
```
##  2. <a name='PromisePromise'></a>Promise概述 + 手写Promise
Promise是es6新增的语法，解决了回调地狱的问题。
可以把Promise看成一个状态机。初始是`pending`状态，可以通过函数`resolve`和`reject`,将状态转变为 `resolve`或者`rejected`状态，状态一旦改变就不能再次变化。

`.then和.catch`用于处理回调，`.then`中会接收两个函数，第一个函数是成功的回调函数，第二个参数是失败的回调函数。`.catch`中只能接收一个参数就是失败的回调函数。`.catch`一般是放在最后，可以解决一个Promise异常穿透的问题。
- 手写Promise
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
##  3. <a name='Promise.allPromise.allSettled'></a>手写Promise.all和Promise.allSettled
`Promise.all`：同时执行多个Promise对象（以数组的形式传入）。两种情况，当所有的Promise状态为fulfilled时，新的Promise状态为fullfilled，并且将所有的Promise的返回值组成一个数字；当有一个Promise状态为reject时，新的Promise状态为reject，并且会将第一个reject的返回值作为参数。

__缺点__：当有一个Promise变为reject状态时，新的Promise就会立即变成对应的reject状态，对于resolved以及依然处于pending状态的Promise，就获取不到对应的结果。
手写Promise.all
```js
const MyPromiseAll = (iterator) => {
    // 对传入的数据做一个浅拷贝，确保有遍历器
    const promises = Array.from(iterator);
    const len = promises.length;
    let data = []; // 用来存储返回的数据数组
    let index = 0; // 记录data的长度
    return new Promise((resolve, reject)=>{
        for(let i in promises){
            promises[i].then(res => {
                data[i] = res;
                if(++index === len){
                    resolve(data);
                }
            }).catch(err => {
                reject(err);
            })
        }
    })
}
const promise1 = Promise.resolve('promise1')
const promise2 = new Promise(function(resolve, reject){
  setTimeout(resolve, 2000, 'promise2')
})
const promise3 = new Promise(function(resolve, reject){
  setTimeout(resolve, 1000, 'promise3')
})
PromiseAll([promise1, promise2, promise3]).then(function(values){
  console.log(values)
})
```
`Promise.allSettled`：该方法会在所有的Promise都有结果（无论是fullfilled还是reject）才会有最终状态，并且这个Promise的结果一定是fullfilled。
```js
const myPromiseAllSettled = function(it){
    // 浅拷贝，确保有迭代器
    const promises = Array.from(it);
    const len = promises.length;
    let data = []; // 返回值
    let index = 0; // 统计
    return new Promises((resolve, reject)=>{
        promises[i].then(res=>{
            data[i] = {
                status: "fullfilled",
                value: res
            }
            if(++index === len){
                resolve(data);
            }
        }, reason=>{
            data[i] = {
                status: "rejected",
                reason
            }
            if(++index === len){
                resolve(data);
            }
        })
    })
}
const p = new Promise((resolve, reject)=>{
setTimeout(()=>{
        reject(3);
    },1000)
}) 
console.log(myPromiseAllSettled([p,Promise.resolve(1), Promise.reject(2)]).then(res=>{
    console.log(res)
})) 
```
##  4. <a name='Promise'></a>Promise经典考题
- 一
    ```js
     new Promise((res, rej) => {
        console.log(1);
        throw new Error('abc');
        res(2);
        console.log(3);
    }).catch((e) => {
        console.log('catch');
    }).then((t) => {
        console.log(t);
    });
    console.log('end');
    ```
- 二
    ```js
    new Promise((res, rej) => {
        console.log(1);
        res(2);
        throw new Error('abc');
        console.log(3);
    }).catch((e) => {
        console.log('catch');
    }).then((t) => {
        console.log(t);
    });
    console.log('end');
    ```
##  5. <a name='-1'></a>数组常用方法
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
    console.log([1,2,3,4,5].splice(1,2,3,4,5))
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
    console.log([1,2,3,4,5].slice(1,2,3,4,5))
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
##  6. <a name='-1'></a>数组去重常用的方法
- 方法一：使用set
```js
const arr = [1,1,2,2,3,4];
function unique(arr){
    return Array.from(new Set(arr));
}
```
- 方法二：使用reduce + indexOf
```js
const arr = [1,1,2,2,3,4];
function unique(arr){
    const res = arr.reduce((prev, curr)=>{
        if(prev.indexOf(curr)===-1){
            prev.push(curr);
        }
    }, [])
    return res;
}
```
- 方法三：使用filter + indexOf
```js
const arr = [1,1,2,2,3,4];
function unique(arr){
    const res = arr.filter((item, index)=>arr.indexOf(item) === index);
    return res;
}
```
- 方法四：使用map
```js
const arr = [1,1,2,2,3,4];
function unique(arr){
    let map = new Map();
    let res = [];
    arr.forEach(i => {
        if(!map.has(i)){
            map.set(i, 1);
            res.push(i);
        }
    })
    return res;
}
```
##  7. <a name='-1'></a>数组扁平化
将数组拉平的过程
- 方法一：使用递归+concat
```js
const a = [1, [2, [3, [4,5]]]];
const flatten = (arr)=>{
    let result = [];
    for(let i=0; i<arr.length; i++){
        if(Array.isArray(arr[i])){
            result = result.concat(flatten(arr[i]));
        }else{
            result.push(arr[i]);
        }
    }
    return result;
}
console.log(flatten(a));
```
- 方法二：使用扩展运算符
```js
const arrNum = [1, [2, 3], [4, [5, 6, 7]]];
function flatten(arr){
    // 循环，直到arr中没有数组才跳出循环
    while(arr.some(item=> Array.isArray(item))){
        arr = [].concat(...arr);
    }
    return arr;
}
console.log(flatten(arrNum)) //  [1, 2, 3, 4，5, 6, 7]
```
##  8. <a name='shuffle'></a>数组乱序（shuffle）
- Math.floor() 向下取整 2.7 -> 2
- Math.ceil() 向上取整 2.7 -> 3
- Math.round() 四舍五入
- Math.random() 返回介于0（包括）-1（不包括）之间的随机数
```js
// 思路：从最后一个元素开始，从数组中随机选出一个位置，交换，直到第一个元素
function shuffle(arr){
    const len = arr.length;
    let curr = len-1;
    let random;
    while(curr >= 0){
        random = Math.floor(len * Math.random());
        [arr[curr], arr[random]] = [arr[random], arr[curr]];
        curr--;
    }
    return arr;
}
```
##  9. <a name='-1'></a>函数柯里化
通俗理解：用闭包把参数保存起来，当参数的数量足够执行函数了，就开始执行函数。
- 实现
    - 判断当前函数传入的参数是否大于或等于`fn`需要参数的数量，如果是，直接执行fn
    - 如果传入参数数量不够，返回一个闭包，暂存传入的参数，并重新返回`currying`函数
```js
function currying(fn, ...args){
    if(args.length >= fn.length){
        return fn(...args);
    }else{
        return (...args2)=>currying(fn, ...args, ...args2);
    }
}
// 简单验证
function fun(a, b, c){
    console.log(a,b,c)
}
const curryingFun = currying(fun)
curryingFun(1)(2)(3);  // 1 2 3 
// curryingFun(1, 2)(3);  // 1 2 3 
// curryingFun(1, 2, 3);  // 1 2 3 

```
##  10. <a name='instanceoftypeof'></a>instanceof和typeof的原理
- instanceof：测试一个对象在其原型链中是否存在一个构造函数的prototype属性，简单来说就是通过 __原型链__ 去查找。
- typeof：js在底层存储变量的时候，会在变量的机器码的低位1-3位存储其类型信息
    - 000 对象
    - 010 浮点数
    - 100 字符串
    - 110 布尔
    - 111 整数
    - null 所有机器码均为0
    - undefined 用-2^30 整数来表示
    
    所以 typeof在判断null的时候就出现问题，由于null的所有机器码均为0，因此直接被当作了对象来看。
__手写函数__ 传入任意变量，可准确获取类型，如number、string、boolean、object、array、map、regexp
```js
function getType(x){
    const originType = Object.prototype.toString.call(x);
    const spaceIndex = originType.indexOf(' ');
    const type = originType.slice(spaceIndex+1, -1);
    return type.toLowerCase();
}
```
##  11. <a name='js5'></a>js实现继承的5种方法
__父类__
```js
function Father(name){
    this.name = name || 'father';
    // 实例方法
    this.sayName = function(){
        console.log(this.name);
    }
    this.color = ['red', 'blue'];
    // 原型方法
    Father.prototype.age = 18;
    Father.prototype.sayAge = function(){
        console.log(this.age);
    }
}
```
- 方法一：原型链继承：将父类的实例作为子类的原型
    ```js
    function Son(name){
    this.name = name || 'son'
    }
    Son.prototype = new Father()
    let s1 = new Son('s1')
    let s2 = new Son('s2')
    s1.color.push('black')
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
- 方法五：es6——class继承：使用extends表面继承自哪个父类，并且在子类的构造函数中必须调用super
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