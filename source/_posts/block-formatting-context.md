---
title: 盒模型与BFC
date: 2018-02-23 15:05:06
tags: 
  - JavaScript
categories: 前端
---

### 一、盒模型

> 在浏览器中有2种盒模型：W3C标准（标准盒模型），IE采用的盒模型称为（怪异盒模型），大部分浏览器在支持W3C标准的情况下还保留了IE采用的盒模型，在css3中可以通过box-sizing来设置。
>
> 当不对Doctype进行定义时，会触发怪异模式。

<!--more-->

如下例子：

<iframe width="100%" height="300" src="//jsfiddle.net/Chs97/e6xt1kro/embedded/html,css,result/" allowpaymentrequest allowfullscreen="allowfullscreen" frameborder="0"></iframe>

#### 1.标准盒模型

![](content.png)

在标准盒模型下：一个块的总宽度为：width+padding(left+right)+border(left+right)

####2.怪异盒模型

![](border.png)

在怪异盒模型下：一个块的总宽度为：width

也就是说，内容的宽度应该为width - (border-left + padding-left + padding-right + border-right)

### 二、行内元素与块级元素

> HTML (超文本标记语言) 元素大多数都是[行内元素](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Inline_elemente)或[块级元素](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Block-level_elements)。

#### 1.行内元素

> 一个行内元素只占据它对应标签的边框所包含的空间。一般情况下，行内元素只能包含文本数据或者其他行内元素。例子：

<iframe width="100%" height="300" src="//jsfiddle.net/Chs97/hs9dpk69/2/embedded/html,css,result/" allowpaymentrequest allowfullscreen="allowfullscreen" frameborder="0"></iframe>

行内元素会在水平上一个接着一个排列。

#### 2.块级元素

> 块级元素占据其父元素（容器）的整个空间，因此创建了一个“块”。通常浏览器会在块级元素前后另起一个新行。例子：

<iframe width="100%" height="300" src="//jsfiddle.net/Chs97/kw1a3wug/embedded/html,css,result/" allowpaymentrequest allowfullscreen="allowfullscreen" frameborder="0"></iframe>

一般情况下，每一个块级元素的默认宽度与父元素相同。

#### 3.display属性

- 当元素的CSS属性`display`为`block`，`list-item`或 `table`时，它是块级元素 ；


- 当元素的CSS属性`display`的计算值为`inline`，`inline-block`或`inline-table`时，称它为行内级元素；

### 三、块格式化上下文（Block Formatting Context）

#### 1.BFC

> **块格式化上下文（Block Formatting Context，BFC）**是Web页面的可视化CSS渲染的部分，是块级盒布局发生的区域，也是浮动元素与其他元素交互的区域。

#### 2.特征

1. 内部的盒会在垂直方向一个接一个排列（可以看作BFC中有一个的常规流）；
2. 处于同一个BFC中的元素相互影响，可能会发生margin collapse；
3. 每个元素的margin box的左边，与容器块border box的左边相接触(对于**从左往右的格式化**，否则相反)。即使存在浮动也是如此；
4. BFC就是页面上的一个隔离的独立容器，容器里面的子元素不会影响到外面的元素，反之亦然；
5. 计算BFC的高度时，考虑BFC所包含的所有元素，连浮动元素也参与计算；
6. 浮动盒区域不叠加到BFC上；

####  3.BFC创建

- 根元素或其它包含它的元素
- 浮动元素 (元素的 [`float`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/float) 不是 `none`)
- 绝对定位元素 (元素的 [`position`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/position) 为 `absolute` 或 `fixed`)
- 内联块元素 (元素具有 [`display`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/display)`: inline-block`)
- 表格单元格 (元素具有 [`display`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/display)`: table-cell，HTML表格单元格默认属性`)
- 表格标题 (元素具有 [`display`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/display)`: table-caption`, HTML表格标题默认属性)
- 匿名表格元素 (元素具有 [`display`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/display)`: table`, `table-row`, `table-row-group`, `table-header-group`, `table-footer-group` [分别是HTML tables, table rows, table bodies, table headers and table footers的默认属性]，或 `inline-table` )
- [`overflow`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/overflow) 值不为 `visible` 的块元素，
- [`display`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/display) 值为 `flow-root` 的元素
- [`contain`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/contain) 值为 `layout`, `content`, 或 `strict` 的元素
- 弹性元素 ([`display`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/display)`: flex` 或 `inline-flex`元素的子元素)
- 网格元素 ([`display`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/display)`: grid` 或 `inline-grid` 元素的子元素)
- 多列容器 (元素的 [`column-count`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/column-count) 或 [`column-width`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/column-width) 不为 `auto` 即视为多列，`column-count: 1`的元素也属于多列)
- 即便具有 `column-span: all` 的元素没有被包裹在一个多列容器中，[`column-span`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/column-span)`: all `也始终会创建一个新的格式化上下文。

#### 4.实例

##### 一、同一个BFC中的相邻的2个块margin重叠

<iframe width="100%" height="300" src="//jsfiddle.net/efb521or/3/embedded/html,css,result/" allowpaymentrequest allowfullscreen="allowfullscreen" frameborder="0"></iframe>

上面部分有2个块级元素，属于同一个BFC内（html），于是边距发生了重叠。

而下面部分的2个块级元素，属于不同的BFC内，所以边距不发生重叠。

##### 二、BFC中元素重叠

<iframe width="100%" height="300" src="//jsfiddle.net/Chs97/015tgms2/2/embedded/html,css,result/" allowpaymentrequest allowfullscreen="allowfullscreen" frameborder="0"></iframe>

如上：.main和.side重叠了，因为2个box都是属于同一个BFC的。如果.main没有产生浮动，那么.main与.side应该是垂直排列的，但是.main产生了浮动，所以2个box都是从左边开始排列，所以形成重叠。

##### 三、高度坍塌

<iframe width="100%" height="300" src="//jsfiddle.net/Chs97/3m2zuy1h/2/embedded/html,css,result/" allowpaymentrequest allowfullscreen="allowfullscreen" frameborder="0"></iframe>

这个看起来有点不整齐，为什么下面的contain会跑到上面去？因为上面的contain发生了高度坍塌，浮动元素的高度没有被计算进去。下面的contain，因为创建了BFC，所以BFC内浮动元素的高度被计算了进去，于是看到的是正常情况。

### 四、用途

挖坑待填？

### 五、参考

[学习 BFC](https://juejin.im/post/59b73d5bf265da064618731d#heading-1)

[视觉格式化模型](https://developer.mozilla.org/zh-CN/docs/Web/Guide/CSS/Visual_formatting_model)

[CSS中 怪异盒模型VS标准盒模型](https://www.jianshu.com/p/9a3090f1924a)