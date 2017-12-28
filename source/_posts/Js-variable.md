---
title: JavaScript 变量作用域&变量提升
date: 2016-11-14 11:16
tags:
  - JavaScript
categories: 前端
---

## 一. 作用域

1.块级作用域任何一对花括号（｛和｝）中的语句集都属于一个块，在这之中定义的所有变量在代码块外都是不可见的，我们称之为块级作用域。

例如：

> for(int i = 0;i<=n;i++)中的 i <br>while(1){这里面的变量}

在 C/C++中是存在块级作用域的，但是在 JS 中不存在。

例如:

> for(var i=0;i<3;i++){ } alert(i); 会弹出 3

也就是说，JS 并不支持块级作用域，它只支持函数作用域，而且在一个函数中的任何位置定义的变量在该函数中的任何地方都是可见的。

2.函数作用域

定义在函数中的参数和变量在函数外部是不可见的。

<!--more-->

## 二. 变量提升

```
var v='Hello World';
(function(){
    alert(v);
})()
```

这个运行之后 弹出的是 Hello World

```
var v='Hello World';
(function(){
    alert(v);
    var v='I love you';
})()
```

这个运行之后 弹出的是 undefined

原因是 在 JS 中存在变量提升，所有变量的声明都会被放到最前面 所以上面的代码 等同于

```
var v='Hello World';
(function(){
    var v;
    alert(v);
    v='I love you';
})()
```

例如:

```
(function(){
    var a='One';
    var b='Two';
    var c='Three';
})()
```

等价于

```
(function(){
    var a,b,c;
    a='One';
    b='Two';
    c='Three';
})()
```

## 三.利用自执行函数模拟出块级作用域

比如

```
for(var i=0 ;i<3;i++)
alert(i);
```

这时候弹出的是 3

```
(function () {for(var i=0 ;i<3;i++);}());
alert(i);
```

这时候的 i 就是 undefined

因为 他变量提升到函数作用域,函数外部是访问不了函数内部的变量。这样能避免变量的污染啊 巴拉巴拉的。。
