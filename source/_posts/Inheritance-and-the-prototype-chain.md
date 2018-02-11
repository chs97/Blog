---
title: 继承和原型链
date: 2018-02-11 13:21:48
tags: 
  - JavaScript
categories: 前端
---

### 一、继承

> 对于有基于类的语言经验 (如 Java 或 C++) 的开发人员来说，JavaScript 有点令人困惑，因为它是动态的，并且本身不提供一个`class`实现。（在 ES2015/ES6 中引入了`class`关键字，但只是语法糖，JavaScript 仍然是基于原型的）。

在继承中：JS只有一种结构：对象，每个对象有一个私有属性（[[prototype]]）浏览器中表示为*\_\_proto\_\_* ,它指向原型对象（prototype）。同理，该对象又有一个私有属性（[[prototype]]）指向该对象的原型对象，通过这样一层一层的直到最后的原型对象为null，并成为原型链的最后一个环节。

我们通过一段代码来描述这个过程：

```javascript
let animal = {}
let bird = Object.create(animal)
let eagle = Object.create(bird)
eagle.__proto__ === bird // true
bird.__proto__ === animal // true
animal.__proto__.__proto__ === null // true
```

<!--more-->

### 二、继承的实现

#### 1.[Object.create(proto)](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/create)

##### 参数

- `proto`

  新创建对象的原型对象。

- `propertiesObject`

  可选。如果没有指定为 [`undefined`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/undefined)，则是要添加到新创建对象的可枚举属性（即其自身定义的属性，而不是其原型链上的枚举属性）对象的属性描述符以及相应的属性名称。这些属性对应[`Object.defineProperties()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperties)的第二个参数。

##### 返回值

在指定原型对象上添加新属性后的对象。

```javascript
let bird = {type: 'bird'}
let eagle = Object.create(bird)
eagle.type // bird
eagle.__proto__ === bird // true
```

#### 2.通过构造函数

先来看一个例子

```javascript
function person(name) {
    this.name = name
}
let hbb = new person('chs')
hbb.__proto__ === person // false
```

hbb是new出来的，继承于person，按道理来说，hbb的原型应该是person？但是person是个函数，不是对象。

```javascript
hbb.__proto__ === person.prototype // true
```

这个prototype又是什么？

> 只有函数才有prototype属性

为什么？因为ES规范就这样。

当创建一个函数时，JS会自动为函数添加一个属性`prototype`，这个属性的值是个对象，里面有一个`constructor`属性，当你使用 `new` 构造一个对象时，会自动把新对象的**\_\_proto\_\_**指向它

原理如下：

```javascript
// 上面例子等同于：
let hbb = {}
hbb.__proto__ = person.prototype
person.call(hbb, 'chs')
```

#### 3.通过class(es6)

class其实就是构造函数的一个语法糖。只是让代码看起来更加OOP。。而已。

```javascript
function person(name) {
    this.name = name
}
person.prototype.say = function() {
    console.log(this.name)
}
//使用Class语法重构
class person {
    constructor(name) {
        this.name = name
    }
    say() {
        console.log(this.name)
    }
}
```

那么，class的继承和构造函数有什么区别？
```javascript
function A(name) {
    this.name = name
}
function B(type, name) {
    this.type = type
    A.call(this, name)
}
let C = new B('name', 'type')
B.prototype.__proto__ === A.prototype // false
B.__proto__ === A // false
// B 和 A 之间的继承其实不是真正的继承，通过一个call的方法实现的伪继承，复制了一下属性而已。
class A {
    constructor(name) {
        this.name = name
    }
}

class B extends A {
    constructor(name, type) {
        super(name)
        this.type = type
    }
}
let C = new B('name', 'type')
C.__proto__ === B.prototype // true
B.prototype.__proto__ === A.prototype // true
B.__proto__ === A // true
// B 和 A实现了继承
```

```javascript
// B 的实例继承 A 的实例
Object.setPrototypeOf(B.prototype, A.prototype);

// B 的实例继承 A 的静态属性
Object.setPrototypeOf(B, A);

const b = new B();
// ----------------------------------------------//
Object.setPrototypeOf(B.prototype, A.prototype);
// 等同于
B.prototype.__proto__ = A.prototype;

Object.setPrototypeOf(B, A);
// 等同于
B.__proto__ = A;
```

### 三、原型链

原型链就是继承实现中的一个类似链表的东西。

比如：

```javascript
function person(name) {
    this.name = name
}
let hbb = new person('chs')
// 原型链为
// hbb.__proto__ -> person.prototype
// person.prototype.__proto__ -> Object.prototype
// Object.prototype.__proto__ -> null


// __proto__ 指向父的原型，构造函数中的原型为prototype属性
```

```javascript
let bird = {type: 'bird'}
let eagle = Object.create(bird)
// 原型链为
// eagle.__proto__ -> bird
// bird.__proto__ -> Object.prototype
// Object.prototype.__proto__ -> null
```

```javascript
class A {
    constructor(name) {
        this.name = name
    }
}

class B extends A {
    constructor(name, type) {
        super(name)
        this.type = type
    }
}
let C = new B('name', 'type')
// 原型链：
// C.__proto__ -> B.prototype
// B.prototype.__proto__ -> A.prototype
// A.prototype.__proto__ -> Object.prototype
// Object.prototype.__proto__ -> null
```



js中是通过`__proto__`指针来实现原型链的。

关键的地方在于：object没有prototype对象，而function有prototype对象。

创建原型链的过程中：`__proto__`是指向父亲的prototype属性。




### 参考链接：

[从__proto__和prototype来深入理解JS对象和原型链](https://github.com/creeperyang/blog/issues/9)

[继承与原型链](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Inheritance_and_the_prototype_chain)

[JavaScript原型及原型链](https://hexianzhi.github.io/2017/04/27/JavaScript%E5%8E%9F%E5%9E%8B/)

[Class继承](http://es6.ruanyifeng.com/#docs/class-extends)