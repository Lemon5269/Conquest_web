

### 深拷贝浅拷贝
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
### Promise
Promise是es6新增的语法，解决了回调地狱的问题。
可以把Promise看成一个状态机。初始是`pending`状态，可以通过函数`resolve`和`reject`,将状态转变为 `resolve`或者`rejected`状态，状态一旦改变就不能再次变化。

`.then和.catch`用于处理回调，`.then`中会接收两个函数，第一个函数是成功的回调函数，第二个参数是失败的回调函数。`.catch`中只能接收一个参数就是失败的回调函数。`.catch`一般是放在最后，可以解决一个Promise异常穿透的问题。
- 手写Promise
```js
class MyPromise{
    constructor(executor){
        // 初始值
        this.initValue();
        // 初始化this指向 this指向永远是当前MyPromise实例
        this.initBind();
        try{
            // 执行传进来的函数
            executor(this.resolve, this.reject);
        }
    }
    initValue(){
        this.PromiseResult = null; // 终值
        this.PromiseState = 'pending' // 状态
        this.onFulfilledCallbacks = []; // 保存成功回调
        this.onRejectedCallbacks = []; // 失败的回调
    }
    initBind(){
        this.resolve = this.resolve.bind(this);
        this.reject = this.reject.bind(this);
    }
    resolve(value){
        // 状态一旦改变不可以再次改变
        if(this.PromiseState !== 'pending') return;
        // 如果执行resolve,状态改为fulfilled
        this.PromiseState = 'fulfilled';
        // 终值为传进来的值
        this.PromiseResult = value;
        // 执行保存的成功回调
        while(this.onFulfilledCallbacks.length){
            this.onFulfilledCallbacks.shift()(this.PromiseResult);
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
    }
}
```


