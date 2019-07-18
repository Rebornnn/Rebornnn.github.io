---
title: 《JavaScript高级程序设计》读书笔记5.5Function类型
date: 2017-08-15 09:57:00
categories:
- 读书笔记
- 《JavaScript高级程序设计》
tags:
- JavaScript
- 系统学习
description: 这里把函数当作对象来介绍
---
- 函数实际上是对象，也就是`Function`类型的实例。
- 函数名也是一个指向函数对象的指针，不会与函数绑定。
- 两种定义方法：函数声明语法、函数表达式
- *使用不带圆括号的函数名是访问函数指针，而非调用函数*。
- 一个函数可能会有多个名字
```javascript
function sum(num1,num2){
    return num1+num2;
}
alert(sum(10,20));          //30

var anothersum=sum;          //引用类型值的复制
alert(anothersum(10,20));    //30

sum=null;                    //解除引用
alert(anothersum(10,20));    //30
```
#### 1.函数声明与函数表达式
**函数声明提升过程**：JavaScript引擎会将声明的函数放到源代码树的顶部。    
函数声明：随时调用即可用。    
函数表达式：必须等到解析器执行到所在行，才能执行。

#### 2.作为值的函数
ECMAScript中的函数名本身是变量，所以函数也可以作为值来使用。不仅可以像传递参数一样把一个函数传递给另一个函数，而且可以将一个函数作为另一个函数的结果返回。
```javascript
把函数作为参数传递
function callSomeFunction(someFunction,someArgument){
    return someFunction(someArgument);
}

从一个函数中返回另一个函数
function createComparsionFunction(propertyName){
    return function(object1,object2){
        var value1=object1[propertyName];
        var value2=object2[propertyName];
        if(value1<value2){
            return -1;
        }else if{
            return 1;
        }else{
            return 0;
        }
    };
}
```
#### 3.函数内部属性
- 函数内部，有两个特殊对象：`arguments`和`this`。
- `callee`：`arguments`的一个属性，该属性是一个指针，指向拥有这个`arguments`对象的函数，即正在被执行的`Function`对象。
```javascript
经典阶乘函数
function factorial(num){
    if(num<=1){
        return 1;
    }else{
        return num*factorial(num-1);
    }
}

var trueFactorial=factorial;
factorial=function(){
    return 0;
};
alert(trueFactorial(5));   //0 计算时会调用factorial新赋值的新函数而返回0
alert(factorial(5));       //0
```

升级重写之后
```javascript
function factorial(num){
    if(num<=1){
        return 1;
    }else{
        return num*arguments.callee(num-1);
    }
}

var trueFactorial=factorial;
factorial=function(){
    return 0;
};
alert(trueFactorial(5));   //120 仍然调用老factorial函数
alert(factorial(5));       //0
```

- `this`：引用的是函数执行的环境对象。函数在谁的环境下执行就引用这个对象。

```javascript
window.color="red";
var o={color:"blue"};
function sayColor(){
    alert(this.color);
};
sayColor();      //"red"

o.sayColor=sayColor;
o.sayColor();    //"blue"
```
- `caller`：ECMAScript5中规范的一个函数对象的属性。保存着调用当前函数的函数的引用。就是指当前函数的父函数。
```javascript
function outer(){
    inner();
}

function inner(){
    alert(inner.caller);
}
outer();    //显示outer()函数的源代码
```
#### 4.函数属性和方法
- 每个函数有两个属性：`length`和`prototype`。两个方法：`apply()`和`call()`
- `length`：表示函数希望接收的命名参数的个数。
- `prototype`：保存着所有实例方法和属性。不可枚举。继承。
- `apply()`和`call()`:能在特定的作用域中调用函数，实际上等于设置函数体内`this`对象的值。
- `apply()`接受两个参数：一个是给其运行函数的作用域，一个是参数数组（可以是实例，也可以是`arguments`对象）
```javascript
function sum(num1,num2){
    return num1+num2;
}
function callSum1(num1,num2){
    return sum.apply(this,arguments);  //this在callSum1函数体内，callSum1在全局作用域，this引用window对象
}
function callSum2(num1,num2){
    return sum.apply(this,[num1,num2])
}
alert(callSum1(10,10));    //20
alert(callSum2(10,10));   //20
```

- `call()`和`apply()`都一样，只在接受参数时必须一一列举参数。

```javascript
function sum(num1,num2){
    return num1+num2;
}
function callSum1(num1,num2){
    return sum.call(this,num1,num2);
}
alert(callSum1(10,10));   //20
``` 

- **`apply()`和`call()`真正强大**：能够扩充函数赖以运行的作用域。**对象不需要与方法有任何耦合关系。**

```javascript
window.color="red";
var o={color:"blue"};
function sayColor(){
    alert(this.color);
};
sayColor();             //"red"

sayColor.call(this);    //"red"
sayColor.call(window);  //"red"
sayColor.call(o);       //"blue",将this值指向o对象
```
- `bind()`：ECMAScript5中定义。它会创建一个函数的实例，其this值会被绑定到传给bind()函数的值。
```javascript
window.color="red";
var o={color:"blue"};
function sayColor(){
    alert(this.color);
};
var objectSayColor=sayColor.bind(o);
objectSayColor();         //"blue"
```
