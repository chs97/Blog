---
title: This 详解&&call/apply/bind的区别
date: 2017-04-27 12:06
tags:
  - JavaScript
categories: 前端
---

### this 的四种调用方式

#### 一、方法调用模式

`当一个函数被保存为对象的一个属性并调用该方法时，this被绑定至该对象。即使用”obj.method”形式`

```javascript
var val = 'outer'
var methodCall = {
  val: 'inner',
  printVal: function() {
    console.log(this.val)
  }
}
methodCall.printVal() //"inner"
```

<!--more-->

#### 二、构造器调用模式

`类似面向对象语言中**对象**的概念，使用new后，this被绑定至该实例。`

```javascript
var val = 'outer'
function methodCall() {
  this.val = 'inner'
}
methodCall.prototype = {
  printVal: function() {
    console.log(this.val)
  }
}
var a = new methodCall()
a.printVal() //"inner"
```

#### 三、函数调用模式

`函数未作为对象的属性时，其当作一个正常函数来调用，此时this被绑定至全局对象。`

> 注意：在严格模式下，此时的 this 为 undefined

```javascript
var val = 'outer'
function methodCall(){
    this.val = 'inner'
}
methodCall.prototype = {
    printVal: function(){
        var that = this
        var innerPrintVal = function(){
            console.log(that.val)
            console.log(this.val)
        }
        innerPrintVal()
    }
}
var a = new methodCall()
a.printVal()
结果为：
inner
outer
```

```javascript
var obj = {
  a: function() {
    console.log(this)
    function b () {
      console.log(this)
    }
    b()
  }
}
obj.a()
结果为：
Object
Window
```

包括匿名函数/自执行函数：

```javascript
var obj = {
  a: function() {
    console.log(this)
    ;(function () {
      console.log(this)
    })()
  }
}
obj.a()
结果为：
Object
Window
```

#### 四、apply/call/bind

`手动指向this的值`

```javascript
var val = 'outer'
function printVal () {
  this.val = 'inter'
  this.fn = function () {
  	console.log(this.val)
  }
}
var obj = new printVal()
obj.fn()
obj.fn.call(window)
obj.fn.apply(window)
obj.fn.bind(window)()
输出：
inter
outer
outer
outer
```

---

##### 1.apply [Function.prototype.apply()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/apply)

> fun.apply(thisArg[, argsArray])
>
> thisArg:在 _fun_ 函数运行时指定的 `this` `值。
>
> argsArray:一个数组或者类数组对象，其中的数组元素将作为单独的参数传给 `fun` 函数。

```javascript
var val = 'outer'
function printVal () {
  this.val = 'inter'
  this.fn = function (name, age) {
    console.log(name, age)
  	console.log(this.val)
  }
}
var obj = new printVal()
obj.fn()
obj.fn.apply(window, ['Chs', 1])
输出：
inter
Chs 1
outer
```

##### 2.call [Function.prototype.call()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/call)

> fun.call(thisArg[, arg1[, arg2[, ...]]])
>
> thisArg: 在*fun*函数运行时指定的`this`值*。*
>
> `arg1, arg2, ...`
>
> 指定的参数列表。

```javascript
var val = 'outer'
function printVal () {
  this.val = 'inter'
  this.fn = function (name, age) {
    console.log(name, age)
  	console.log(this.val)
  }
}
var obj = new printVal()
obj.fn()
obj.fn.call(window, 'Chs', 1)
输出：
inter
Chs 1
outer
```

##### 3.bind [Function.prototype.bind()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/bind)

> fun.bind(thisArg[, arg1[, arg2[, ...]]])
>
> `thisArg`
>
> 当绑定函数被调用时，该参数会作为原函数运行时的 this 指向。当使用`new` 操作符 调用绑定函数时，该参数无效。
>
> `arg1, arg2, ...`
>
> 当绑定函数被调用时，这些参数将置于实参之前传递给被绑定的方法。

```javascript
var val = 'outer'
function printVal () {
  this.val = 'inter'
  this.fn = function (name, age) {
    console.log(name, age)
  	console.log(this.val)
  }
}
var obj = new printVal()
obj.fn()
var fn = obj.fn.bind(window, 'Chs', 1)
fn()
输出：
inter
Chs 1
outer
```

---

### this 的应用

#### 一、js 的链式方法

> 实现 fn.set(number).get() 输出为 number

```javascript
function obj() {
  var val = 0
  this.set = function(val) {
    this.val = val
    return this
  }
  this.get = function() {
    return this.val
  }
}
var fn = new obj()
fn.set(20).get() > 20
```

#### 二、柯里化

> 实现 add(1)(2)() = 3 add(1, 2, 3,4)() = 10
>
> 原理： 将所有参数都丢到一起后相加

```javascript
function add() {
  let oldArr = [].slice.call(arguments)
  let _self = this
  return function() {
    let arr = [].slice.call(arguments)
    let newArr = arr.concat(oldArr)
    if (!arr.length) {
      return newArr.reduce((acc, val) => acc + val, 0)
    } else {
      return add.apply(_self, newArr)
    }
  }
}
let x = add(1, 2, 3)(4)(5)()
console.log(x)
```

> arguments:**arguments** 是一个类似数组的对象, 对应于传递给函数的参数。[].slice.call(arguments) 将类数组对象转为数组。

```javascript
朋神优雅的写法

function adder(...arg) {
  return function(...inarg) {
    return inarg.length === 0 ? arg.reduce((a, b) => a + b, 0) : adder.apply(this, [...arg, ...inarg])
  }
}
```

参考：

[this 与 JavaScript 中的四种调用模式](http://miaoyunze.com/2016/09/24/this-in-js/)

[掌握 JavaScript 函数的柯里化](https://github.com/dreamapplehappy/hacking-with-javascript/blob/master/books/javascript-the-good-parts/chapter-4-function/currying.md)
