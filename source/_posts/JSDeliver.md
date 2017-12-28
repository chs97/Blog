---
title: JavaScript 按值传递&&按引用传递
date: 2017-4-10 21:43
tags:
  - JavaScript
categories: 前端
---

### 一、例子

在掘金的文章[最好用的 javascript 编码规范中文版](https://juejin.im/entry/57a032115bbb500064f19b24)提到：

> 当你需要拷贝数组时，使用 Array#slice。

```javascript
var len = items.length
var itemsCopy = []
var i

for (i = 0; i < len; i++) {
  itemsCopy[i] = items[i]
}

itemsCopy = items.slice(0)
```

<!--more-->

为什么不直接用等号传递？

于是有了这段代码

```javascript
var a = [1, 2, 3]
var b = a
a.push(4)
console.log(b)
```

`结果是：[1, 2, 3, 4] , 不是：[1,2,3]`

### 二、变量类型

1. 基本类型（Primitive data type）：number, string, boolean, null, undefined
2. 引用类型（Reference data type）：Object Function

基本类型的数据是存放在栈内存中的，而引用类型的数据是存放在堆内存中的

定义了一个对象其实是在栈内存中存储了一个指针，这个指针指向堆内存中该对象的存储地址。复制给另一个对象的过程其实是把该对象的地址复制给了另一个对象变量，两个指针都指向同一个对象，所以若其中一个修改了，则另一个也会改变。

### 三、应用

```javascript
var obj = {
  name: 'asd'
}
var output = change(obj)

function change(input) {
  var copy = input // 引用 copy指向input的指向
  copy.name = 'asdd' // copy的指向发生改变
  return copy
}
console.log(obj) //Object {name: "asdd"}
```

### 深拷贝和浅拷贝

```javascript
// 浅拷贝将对象的各个属性进行依次复制
var obj = {
  name: 'asd',
  child: {
    name: '123',
    child: {
      name: '456'
    }
  }
}
function copy(obj) {
  var result = {}
  for (var key in obj) result[key] = obj[key]
  return result
}
var ans = copy(obj)
obj.child.name = '1231'
// ans.child 指向的地址和 obj.child一样
console.log(ans)
```

```javascript
// 深拷贝，它不仅将原对象的各个属性逐个复制出去，而且将原对象各个属性所包含的对象也依次采用深复制的方法递归复制到新对象上。
var obj = {
  name: 'asd',
  child: {
    name: '123',
    child: {
      name: '456'
    }
  }
}
function deepCopy(obj) {
  var result = {}
  for (var key in obj) {
    if (typeof obj[key] === 'object') result[key] = deepCopy(obj[key])
    else result[key] = obj[key]
  }
  return result
}
var ans = deepCopy(obj)
obj.child.name = '1231'
console.log(ans)
```
