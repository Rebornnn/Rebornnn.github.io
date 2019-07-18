---
title: 《你不知道的JavaScript-上》关于this
date: 2017-10-27 09:24:13
categories:
- 读书笔记
- 《你不知道的JavaScript-上》
tags:
- JavaScript
- 系统学习
description: this两大误解：指向自身、它的作用域
---
## 为什么要使用this
this可以让函数自动引用合适的上下文对象。

### 误解
2种常见的误解：*指向自身*、*它的作用域*

##### 1. 指向自身    
想要记录一下函数foo被调用次数，以下是错误的代码
```javascript
function foo(num){
    console.log("foo:"+num);
    
    //记录foo被调用的次数
    this.count++;
}

foo.count=0;

var i;

for(i=0;i<10;i++){
    if(i>5){
        foo(i);
    }
}

//foo:6
//foo:7
//foo:8
//foo:9

//foo被调用多少次？
console.log(foo.count);  //0
```
以上代码证明`this`并不指向自己。    
    
        
##### 2. 它的作用域      
`this`在任何情况下都不指向函数的词法作用域。*JS内部，作用域确实和对象类似，可见的标识符都是它的属性。但是作用域“对象”无法通过JS代码访问，它存在于JS引擎内部。*   
```javascript
function foo(){
    var a=2;
    this.bar();
}

function bar(){
    console.log(this.a);
}

foo();   //ReferenceError:a is not defined
```
代码中`this`联通`foo()`和`bar()`的词法作用域，但是不可能实现。  
    
    
### this到底是什么
`this`是在运行时绑定的，并不是编写时绑定，它的上下文取决于函数调用时的各种条件。`this`的绑定和函数声明的位置没有任何关系，只取决于函数的调用方式。      
    
当一个函数被调用时，会创建一个活动记录（或称执行上下文，也称执行环境）。这个记录会包含函数在哪里被调用（调用栈）、函数的调用方式、传入的参数等信息。`this`是这个记录的一个属性，会在函数执行过程中用到。    
    
    
### 小结
首先明白`this`既不是指向函数自身，也不是指向函数的词法作用域。      
    
`this`实际上是在函数被调用时发生的绑定，它指向什么完全取决于函数在哪里被调用。