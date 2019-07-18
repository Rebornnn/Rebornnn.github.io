---
title: 《JavaScript语言精粹》附录-毒瘤
date: 2017-10-19 16:13:18
categories:
- 读书笔记
- 《JavaScript语言精粹》
tags:
- JavaScript
- 系统学习
description: 全是深坑！
---
> 那会在一言一行中证明其可怕。      
> ————威廉-莎士比亚     


**以下是JavaScript中一些难以避免的问题特性！**


### 全局变量
JS对全局变量的依赖，是最糟糕的一个特性。JS没有链接器（linker），所有的编译单元都载入一个公共全局对象中。    
共有3种方式定义全局变量：   
1. 在任何函数之外放置一个`var`语句
```javascript
var foo=value;
```
2. 直接给全局对象添加一个属性。全局对象是所有全局变量的容器。在web浏览器中，全局对象名是`window`：
```javascript
window.foo=value;
```
3. 直接使用未声明的变量，被称为*隐式的全局变量*。
```javascript
foo=value;
```

### 块级作用域
没有块级作用域：代码块中声明的变量在包含此代码块的函数的任何位置都是可见的。

### 自动插入分号
JS会有一个自动修复机制，它试图通过自动插入分号来修正有缺损的程序。但不要指望它，一个语句的结尾，一定要带上分号；

### typeof
```javascript
typeof null
```
返回`object`而不是`null`！！
```javascript
typeof /a/
```
返回`object`而不是`regexp`！！

### parseInt
`parseInt`是一个把字符串转换成整数的函数。它遇到非数字时会停止解析。    
如果该字符串第1个字符是0，那么会基于八进制求值。所有最好加上第二个基数参数！

### 浮点数
二进制的浮点数不能正确的处理十进制的小数，因此0.1+0.2不等于0.3！

### NaN
`NaN`表示的不是一个数字。但是：
```javascript
typeof NaN === "number"  //true
```
`typeof`不能识别数字和`NaN`,而且`NaN`也不等于自己。     
检查数字的函数
```javascript
var isNumber=function(num){
    return typeof num === "number"&& isFinite(num);
}
```
### 对象
JS的对象永远不会是真的空对象，因为它们可以从原型链取得成员属性。