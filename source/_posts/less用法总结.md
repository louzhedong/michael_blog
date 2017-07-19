---
title: less用法总结
date: 2017-07-19 10:43:28
tags: "less"
---

## 什么是less

less是一门预处理语言，扩展了css语言，增加了变量、Mixin、函数等特性，使css更易维护和扩展。less完全兼容css语法，可以完全使用css语法。

### 变量

在less中可以使用@符号来定义变量。比较常见的是一些颜色值、宽高之类的属性，他们往往会在一个项目中重复出现，若直接定义，会使后期项目的维护与升级变得十分困难。我们可以通过定义变量将这些属性的值集中到一个地方进行维护。

例如：

``` less
@top-height: 50px;
@letf-width: 100px;
@bg-color-primary: #2d2d2d; 

.left-container {
	position: absolute;
	z-index: 1;
	top: @top-height;
	left: @left-width;
	background-color: @bg-color-primary;
}
```
将被编译成

``` css
.left-container {
	position: absolute;
	z-index: 1;
	top: 50px
	left: 100px;
	background-color: #2d2d2d;
}
```

变量不仅仅可以用来定义属性，还可以用来定义其他一系列的元素

#### 选择器变量
使用 @{...}的语法来表示选择器

``` less
@mySelector: banner;

.@{mySelector} {
	height: 40px;
	line-height: 40px;
	margin: 0 auto;
}
```
将被编译成

``` css
.banner {
	height: 40px;
	line-height: 40px;
	margin: 0 auto;
}
```

#### URLs变量
使用 @{...}的语法来表示选择器

当一个网页需要通过css引入较多的图片，而图片与css文件处于两个深度很大的子文件中，采用css的写法，就需要写一大堆 "../../" 之类的路径，这时候就可以采用url变量

``` less
@imagesPath: "../images"

.arrow-select {
	cursor: pointer;
	height: 24px;
	line-height: 24px;
	background: url("@{imagesPath}/down_crrow.png") no-repeat center;
}
```

将被编译成

``` css
.arrow-select {
	cursor: pointer;
	height: 24px;
	line-height: 24px;
	background: url("../images/down_crrow.png") no-repeat center;
}
```

#### 属性名变量

甚至可以使用属性名变量，用较短的变量来表示较长的属性名

同样使用 @{...}的语法来表示选择器

``` less
@bb: border-bottom;
@radius: 4px
@borderColor: #dadada;

.search-icon {
	cursor: pointer;
	text-align: center;
	@{bb}: 1px solid @borderColor;
	@{bb}-right-radius: @radius;
}
```

将被编译为

``` css
.search-icon {
	cursor: pointer;
	text-align: center;
	border-bottom: 1px solid #dadada;
	border-bottom-right-radius: 4px;
}
```

#### 嵌套变量

还可以对变量进行嵌套，即在一个变量中使用另一个变量

``` less
@louzhedong: "My name is louzhedong";
@var: "louzhedong";
content: @@var;
```

将被编译为

``` css
content: "My name is louzhedong";
```

### 惰性加载（Lazy Loading）

less 与 es5一样，存在变量提升，对于变量的引用不需要考虑顺序，考虑如下示例

``` less
.lazy-eval {
	width: @var;
}

@var: @a;
@a: 9%;
```

``` less
.lazy-eval {
	width: @var;
	@a: 9%;
}
@var: @a;
@a: 100%;
```

上述两段代码都将被编译成

``` css
.lazy-eval {
	width: 9%;
}
```
### Extend

extend（扩展）是一个less伪类，可以将当前元素的属性扩展为选择元素的属性，类似如下例子

``` less
nav ul {
	&:extend(.inline);
	background: blue;
}
.inline {
	color: red;
}
```

将被编译为

``` css
nav ul {
	background: blue;
}
.inline, nav ul {
	color: red;
}
```

可以看到，nav ul 元素原先是没有color属性的，但在内部使用了extend伪元素，扩展了.inline元素的color属性。

#### extend语法—— all

关键字 all 可以进行全匹配

``` less
.c:extend(.d all){
	// extends all instances of ".d" 例如 ".x.d" 或者 ".d.x"
}

.c:extend(.d) {
	//仅仅扩展 ".d"的css属性
}
```

看一个较复杂的例子

``` less
.a.b.test,
.test.c {
	color: orange;
}
.test {
	&:hover {
		color: green;
	}
}

.replacement:extend(.test all){}
```

将被编译为

``` css
.a.b.test,
.test.c,
.a.b.replacement,
.replacement.c {
	color: orange;
}
.test:hover,
.replacement:hover {
	color: green;
}
```

#### extend语法—— &

extend可以使用&符号在元素内部使用

``` less
pre:hover, 
	.some-class {
	&:extend(div pre);
}
```

将被编译为

``` css
pre:hover:extend(div pre),
	.some-class:extend(div pre) {}
```

注意：extend语法无法匹配变量，即括号里的扩展的目标类不能包括变量

#### extend语法—— extend与作用域

``` less
@media print {
  .screenClass:extend(.selector) {} // extend inside media
  .selector { // this will be matched - it is in the same media
    color: black;
  }
}
.selector { // ruleset on top of style sheet - extend ignores it
  color: red;
}
@media screen {
  .selector {  // ruleset inside another media - extend ignores it
    color: blue;
  }
}
```
将会编译成

``` css
@media print {
  .selector,
  .screenClass { /*  ruleset inside the same media was extended */
    color: black;
  }
}
.selector { /* ruleset on top of style sheet was ignored */
  color: red;
}
@media screen {
  .selector { /* ruleset inside another media was ignored */
    color: blue;
  }
}

```

考虑一个情景，

``` html
<div class="apple">apple</div>
<div class="banana">banana</div>
<div class="pear">bear</div>
```

其中有apple, banana, pear 类，所有的这些类有一些共同的属性，比如color,border,width 等。一种做法是从html切入，比如为这些类都添加另一个类fruit，然后在css的fruit中定义这些公共属性，但这样一来，html就会变得复杂。此时，就可以使用extend属性来对进行扩展。

``` less
@color: #ffffff;
@appleColor: red;
@bananaColor: yellow;
@pearColor: green;
.fruit {
	color: @color;
	width: 100px;
	border: 1px solid @dadada;
}

.apple {
	&:extend(.fruit);
	background-color: @appleColor;
}

.banana {
	&:extend(.fruit);
	background-color: @bananaColor;
}

.pear {
	&:extend(.fruit);
	background-color: @pearColor;
}
```

这样一来，html代码将会变得更加纯净，我们无需为了迎合某些样式而去定义一些表达意义不强的类名。


### Mixin（混合）

mixin有些类似于extend，两者都是使一个类能够更方便地拥有其他类的属性。

mixin的用法

``` less
.a, #b {
	color: red;
}
.mixin-class {
	.a; //或者 .a()
}
.mixin-id {
	#b;  //或者#b()
}
```

将被编译为

``` css
.a, #b {
	color: red;
}
.mixin-class {
	color: red;
}
.mixin-id {
	color: red;
}
```

#### minix用法——带参数

mixins可以传递参数

``` less
.border-radius(@radius: 5px) {
	border-radius: @radius
}

.button {
	.border-radius(6px);
}
```
括号里的5px为默认值，如果不传递参数，变量就将以默认值来赋值。传递多个参数的情况一样。注意，在同一一个类中可以使用多个mixin,并且他们的样式会共同继承


#### mixin用法——@arguments

@arguments在mixin中有特殊的意思，它可以代表所有传入的参数

``` less
.box-shadow(@x: 0; @y: 0; @blur: 1px; @color: #000) {
	-webkit-box-shadow: @arguments;
     -moz-box-shadow: @arguments;
          box-shadow: @arguments;
}
.big-block {
	.box-shadow(2px; 5px);
}
```

将被编译为

``` css
.big-block {
  -webkit-box-shadow: 2px 5px 1px #000;
     -moz-box-shadow: 2px 5px 1px #000;
          box-shadow: 2px 5px 1px #000;
}
```

#### mixin用法——@rest

可以使用 ... 来表示一定数量的参数，也可以使用@rest来表示剩下的变量

``` less
.mixin(...) {        // matches 0-N arguments
.mixin() {           // matches exactly 0 arguments
.mixin(@a; @rest...) {
   // @rest is bound to arguments after @a
   // @arguments is bound to all arguments
}

```

#### mixin用法——匹配

mixin还可以对参数进行匹配，来决定该以什么形式来表示样式。

``` less
.mixin(dart; @color) {
	color: darken(@color, 10%);
}
.mixin(light; @color) {
	color: lighten(@color, 10%);
}
.mixin(@_; @color) {
	display: block
}

@switch: light;

.class {
	.mixin(@switch; #888);
}
```

将会被编译成

``` css
.class {
	color: #a2a2a2;
	display: block;
}
```

#### mixin用法——作为函数

mixins可以像函数那样返回变量，在Mixin中定义的所有变量都可以在调用它的类的作用域中使用

``` less
.average(@x, @y) {
  	@average: ((@x + @y) / 2);
}

div {
  	.average(16px, 50px); // "call" the mixin
  	padding: @average;    // use its "return" value
}
```

从上述的一些例子中，mixin可以像函数那样使用，利用这个特性，可以来简化浏览器兼容的代码。某些比较新的属性，比如flex，border-radius等，在不同浏览器中需要使用不同的前缀来进行兼容。因此可以将这些细节影藏在mixin中，在某些类中调用这些mixin，并将需要的参数传递进行，可以得到带有不同前缀的css代码，可以极大地简化代码量。