---
title: 译.一些ES6的小技巧
date: 2018-04-02 13:21:48
tags: 
  - JavaScript
categories: 前端
---

> 原文： [Check out these useful ECMAScript 2015 (ES6) tips and tricks](https://medium.freecodecamp.org/check-out-these-useful-ecmascript-2015-es6-tips-and-tricks-6db105590377)

EcmaScript 2015 (ES6) 已经出现了很多年了，我们可以使用它的一些新特性。

### 1.设置必须的函数参数

ES6 提供了 默认的参数值，可以让你在函数的参数中指定默认值当参数没有传递其值的时候。

```javascript
const required = () => {throw new Error('Missing parameter')};
//The below function will trow an error if either "a" or "b" is missing.
const add = (a = required(), b = required()) => a + b;
add(1, 2) //3
add(1) // Error: Missing parameter.
```

在这个例子中，我们对函数 `add` 的参数 `a` 和 `b` 设置了一个默认值，这个默认值为一个函数。当函数 `add` 被执行时，且参数 `a` 和 `b` 有一个没有传递值的时候，这个 `required` 函数就会执行，我们就会得到一个报错 `Error: Missing parameter.` 

<!--more-->

### 2.非常强力的"reduce"

`Array` 的方法 `reduce` 是一个有非常多用处的函数。 它一个非常具有代表性的作用是将一个数组转换成一个值。但是你可以用它来做更多的事。

> **🔥** **Tip: 这些小技巧的的初始值都是一个数组或者一个对象而不是一个像字符串之类的简单的值**

#### 2.1 使用reduce做到同时有map和filter的作用

假设有这样的一个场景，你有一个list，里面有一些item， 你需要更新这些item， 更新完以后你需要使用filter来过滤掉你不需要的item。但是这样的话你需要遍历这个数组两次！

```javascript
const numbers = [10, 20, 30, 40];
const doubledOver50 = numbers.reduce((finalList, num) => {
  
  num = num * 2; //double each number (i.e. map)
  
  //filter number > 50
  if (num > 50) {
    finalList.push(num);
  }
  return finalList;
}, []);
doubledOver50; // [60, 80]
```

在这个例子里面，我们想要让数组里面的值翻倍，然后我们只想要里面大于 `50` 的元素。注意一下我们是如何使用 `reduce` 方法做到同时有让元素的值翻倍，然后过滤的。

#### 2.2 使用"reduce"代替"map"和"filter"

在原文中没有体现这部分代码，只说了注意2.1的代码。在这里我添加一下这部分代码:

##### 2.2.1 使用"reduce"代替"map"

```javascript
function map(arr, exec) {
    var res = arr.reduce((res, item, index) => {
        var newItem = exec(item, index)
        res.push(newItem)
        return res
    }, [])
    return res
}

[1, 2, 3].map((item) => item * 2) // [2, 4, 6]

map([1, 2, 3], item => item * 2) // [2, 4, 6]
```

##### 2.2.2 使用"reduce"代替"filter"

```javascript
function filter(arr, exec) {
    var res = arr.reduce((res, item, index) => {
        if (exec(item, index)) {
            res.push(item)
        }
        return res
    }, [])
    return res
}

[1, 2, 3].filter((item, index) => index < 2) // [1, 2]
filter([1, 2, 3], (item, index) => index < 2) // [1, 2]
```

#### 2.3 使用"redece"来判断括号是否匹配

这也是个例子来说明 `reduce` 这个函数功能的强大。给你一串字符串，你想要知道这串字符串的括号是否是匹配的。

常规的做法是使用栈来匹配，但是这里我们使用 `reduce` 就可以做到，我们只需要一个变量 `counter` ，这个变量的初始值是 0 , 当遇到 `(` 的时候，`counter ++ ` 当遇到 `)` 的时候， `counter -- ` 。 如果括号是匹配的，那么这个 `counter` 最终的值是0

```javascript
//Returns 0 if balanced.
const isParensBalanced = (str) => {
  return str.split('').reduce((counter, char) => {
    if(counter < 0) { //matched ")" before "("
      return counter;
    } else if(char === '(') {
      return ++counter;
    } else if(char === ')') {
      return --counter;
    }  else { //matched some other char
      return counter;
    }
    
  }, 0); //<-- starting value of the counter
}
isParensBalanced('(())') // 0 <-- balanced
isParensBalanced('(asdfds)') //0 <-- balanced
isParensBalanced('(()') // 1 <-- not balanced
isParensBalanced(')(') // -1 <-- not balanced
```



#### 2.4 计算数组中元素出现的次数(将数组转为对象)

如果你想计算数组中元素出现的次数或者想把数组转为对象，那么你可以使用 `reduce` 来做到。

```javascript
var cars = ['BMW','Benz', 'Benz', 'Tesla', 'BMW', 'Toyota'];
var carsObj = cars.reduce(function (obj, name) { 
   obj[name] = obj[name] ? ++obj[name] : 1;
  return obj;
}, {});
carsObj; // => { BMW: 2, Benz: 2, Tesla: 1, Toyota: 1 }
```

作者建议前往MDN学习，查看更多例子[MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce)

// 我也去看了一眼MDN，发现reduce真是个黑科技，promise串行这个应用非常秒啊。

### 3.对象解构

#### 3.1 移除不想要的属性

有时候你想要移除对象中的一些不需要的属性，这个对象可能是包含了某些敏感的信息或者只是因为这个对象太大了。我们可以使用解构来代替去遍历这些属性。

```javascript
let {_internal, tooBig, ...cleanObject} = {el1: '1', _internal:"secret", tooBig:{}, el2: '2', el3: '3'};
console.log(cleanObject); // {el1: '1', el2: '2', el3: '3'}
```

在这个例子中，我们想要移除 `_internal` 和 `tooBig` 这两个属性，我们可以将他们分配给变量 `_internal` 和 `tooBig` 中，剩下的属性存放在 `cleanObject` 中，这就是我们想要的。

#### 3.2 在函数参数中使用嵌套对象解构

```javascript
var car = {
  model: 'bmw 2018',
  engine: {
    v6: true,
    turbo: true,
    vin: 12345
  }
}
const modelAndVIN = ({model, engine: {vin}}) => {
  console.log(`model: ${model} vin: ${vin}`);
}
modelAndVIN(car); // => model: bmw 2018  vin: 12345
```

在这个例子中 `engine` 是一个嵌套在 `car` 里面的对象，如果我们只需要 `engine` 里面的属性 `vin` 我们可以这样做。

#### 3.3 合并对象

ES6 新增了一个扩展运算符，`...` 使用三个点来表示，它经常用来解构数组的值， 但是它也可以用在对象中。

在这个例子中，我们在一个新的对象中使用扩展运算符，后面一个对象的属性值会把前面一个对象的属性值覆盖。

第二个对象中的属性 `b` 和 `c`  的值把第一个对象中属性 `b` 与 `c` 的值覆盖掉了 

```javascript
let object1 = { a:1, b:2,c:3 }
let object2 = { b:30, c:40, d:50}
let merged = {…object1, …object2} //spread and re-add into merged
console.log(merged) // {a:1, b:30, c:40, d:50}
```

### 4.Sets

#### 4.1 使用set来对数组去重

在ES6可以很方便的使用Set来对数组去重，因为Set表示的是一个集合，一个元素会在集合里面出现过一次。

```javascript
let arr = [1, 1, 2, 2, 3, 3];
let deduped = [...new Set(arr)] // [1, 2, 3]
```

##### 4.2 使用Array的方法

可以通过(...)扩展运算符将 `Set` 转换成 `Array` 这样我们就可以在 `Set` 使用所有 `Array` 的方法了。

这个例子展示了如何在 `Set` 中使用 `filter` 方法

```javascript
let mySet = new Set([1,2, 3, 4, 5]);
var filtered = [...mySet].filter((x) => x > 3) // [4, 5]
```

### 5.数组的解构

#### 5.1 交换2个值

```javascript
let param1 = 1;
let param2 = 2;
//swap and assign param1 & param2 each others values
[param1, param2] = [param2, param1];
console.log(param1) // 2
console.log(param2) // 1
```

#### 5.2 函数返回多个值

```javascript
async function getFullPost(){
  return await Promise.all([
    fetch('/post'),
    fetch('/comments')
  ]);
}
// const [post, comments] = getFullPost();
const [post, comments] = await getFullPost();
// 注释掉的那一行是作者写的，但是我跑不通，我觉得getFullPost是个Promise，所以需要加await。
```

我们使用 `async\await` 来获取2个接口的数据 `/post` 和 `/comments` ， 函数返回的是一个数组，所以可以使用解构来获得这两个接口的返回值。



> 作者原文章的内容就到这里，但是我想补充一下map。

### 6. [Map](http://es6.ruanyifeng.com/#docs/set-map#Map)

在对象中的 `key` 只能是 `string` 没办法接受一个 `Object` 当作 `key` 会被自动转换成字符串，例子如下：

```javascript
const data = {};
const element = document.getElementById('myDiv');

data[element] = 'metadata';
data['[object HTMLDivElement]'] // "metadata"
```

所以对于`键值对`的数据解构，`map` 比 `Object` 更加合适。

#### 6.1 在map中使用对象当作key

```javascript
const M = new Map()
let o = {p: 'Hello World'}
M.set(o, 'hello world')
M.get(o) // hello world
let O = {p: 'Hello World'}
M.get(O) // undefined
```

注意： `O != o` 所以这个 `M.get(O)` 是 `undefined`

#### 6.2 WeakMap的作用

在普通的map中，如果一个 `object` 被当作 `key` 引用了，当除了map以外，其他的变量都丢掉对这个 `object` 的引用，那map还是会阻止垃圾回收机制去回收这个 `object` 这就会导致内存泄漏。所以就有了WeakMap。

字面意思，WeakMap里面的key是弱引用，并且 WeakMap中只能以 `Object` 当作 `key` 因为其他的你都可以用 Map搞啊~~那么 WeakMap的几个小技巧。

##### 6.2.1 使用DOM节点作为键名

```javascript
let myElement = document.getElementById('logo');
let myWeakmap = new WeakMap();

myWeakmap.set(myElement, {timesClicked: 0});

myElement.addEventListener('click', function() {
  let logoData = myWeakmap.get(myElement);
  logoData.timesClicked++;
}, false);
```

以上例子来自 Ruanyifeng

我们用WeakMap来存DOM节点的某些状态，当这个DOM节点被删除时，myElement失去了引用，全世界都失去了引用，WeakMap里面所存的状态就会消失，所以不会存在内存泄漏的问题。

##### 6.2.2 使用WeakMap来部署私有属性

我们都知道，在JS中实现实例的私有属性是比较困难的，有个办法是使用闭包，但是闭包有个问题，实例被强引用，不能被垃圾回收机制处理。如下代码:

```javascript
var Person = (function() {
    var privateData = {},
        privateId = 0;
    function Person(name) {
        Object.defineProperty(this, "_id", { value: privateId++ });
        privateData[this._id] = {
            name: name
        };
    }
    Person.prototype.getName = function() {
        return privateData[this._id].name;
    };
    return Person;
}());
```

privateData对每个实例都是强引用，所以垃圾回收机制是处理不了的。

我们可以用weakMap很好的解决这个问题。

```javascript
var privateData = new WeakMap();
var Person = (function() {
    function Person(name) {
        privateData.set(this, { name: name });
    }
    Person.prototype.getName = function() {
        return privateData.get(this).name;
    };
    return Person;
}());
```

以上代码weakMap不会对实例造成强引用，也就是说，垃圾回收机制是可能把这个实例回收掉的，如果没有其他人对这个实例保持着引用，是会被回收的，也就不会导致内存泄露了。

以上例子来自[JavaScript实现私有属性](http://www.cnblogs.com/ihardcoder/p/4914938.html)