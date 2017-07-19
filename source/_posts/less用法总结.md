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

甚至我们可以使用属性名变量，用较短的变量来表示较长的属性名

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