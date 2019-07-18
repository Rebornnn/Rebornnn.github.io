---
title: 《你不知道的JavaScript-上》附录-动态作用域与this词法
date: 2017-10-26 14:27:00
categories:
- 读书笔记
- 《你不知道的JavaScript-上》
tags:
- JavaScript
- 系统学习
description: 动态作用域与this是“表亲”
---
动态作用域是JS另一个重要机制`this`的表亲。  
    
词法作用域是一套关于引擎如何寻找变量以及会在何处找到变量的规则。    
词法作用域最重要的特征是它的定义过程发生在代码的书写阶段。  

```javascript
function foo(){
    console.log(a); //2
}

function bar(){
    var a=3;
    foo();
}


var a=2;
bar();
```
词法作用域让`foo()`中的a通过RHS引用到了全局作用域中的a，因此输出2。     
        
动态作用域并不关心函数和作用域是如何声明以及在何处声明，只关心它们从何处调用。换句话说，*作用域链是基于调用栈的，而不是代码中的作用域嵌套。*     
如果JS具有动态作用域，理论上，下面代码中`foo()`在执行时会输出3
```javascript
function foo(){
    console.log(a); //3（不是2）
}

function bar(){
    var a=3;
    foo();
}


var a=2;
bar();
```
**WHY：** 当`foo()`无法找打到a的变量引用时，会顺着调用栈在调用`foo()`的地方查找a，而不是在嵌套的词法作用域链中向上查找。  

**主要区别：** 词法作用域是写在代码或者定义时确定的，而动态作用域是在运行时确定的（`this`也是这样）。词法作用域关注函数在何处声明，而动态作用域关注函数从何处调用。         




箭头函数    

```javascript
var foo=a => {
    console.log(a);
};

foo(2);   //2
``` 
        
        
以下是`this`在闭包中会出现的问题和解决方法：
```javascript
var obj={
    id: "awesome",
    cool: function coolFn(){
        console.log(this.id);
    }
};

var id="not awesome";

obj.cool();   //awesome

setTimeout(obj.cool,100);  //not awesome
```
`obj.cool`函数丢失同`this`之间的绑定。最常用的解决方法是`var self=this`或`var that=this`。
```javascript
var obj={
    count: 0,
    cool: function coolFn(){
        var self=this;
        
        if(self.count<1){
            setTimeout(function timer(){
                self.conunt++;
                console.log("awesome?");
            },100);
        }
    }
};

obj.cool();  //awesome?
```
self只是一个可以通过词法作用域和闭包进行引用的标识符，不关心`this`绑定的过程中发生了什么。    
    
ES6中的箭头函数引入一个叫做`this`词法的行为：
```javascript
var obj={
    conut: 0,
    cool: function coolFn(){
        if(this.conut<1){
            setTimeout( () => {
                this.count++;
                console.log("awesome?");
            },100);
        }
    }
};

obj.cool();   //awesome?
```
上面代码中`this`继承了cool函数的this绑定。      
    
ES6中`this`绑定规则：**用当前词法作用域覆盖了`this`本来的值。**   
    
但是箭头函数还不够理想，因为她是匿名而非具名的。

一种更合适的办法加入`bind()`
```javascript
var obj={
    count: 0,
    cool: funtion coolFn(){
        if(this.count<1){
            setTimeout(function timer(){
                this.count++;   //this是安全的
                                //因为bind(..)
                console.log("more awesome");
            }.bind(this),100);
        }
    }
}

obj.cool();   //more awesome
```