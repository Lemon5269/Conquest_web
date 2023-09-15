

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
### 手写Promise
Promise是es6新增的语法，解决了回调地狱的问题。
可以把Promise看成一个状态机制 

