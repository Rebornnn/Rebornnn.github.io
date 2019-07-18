---
title: 《你不知道的JavaScript-上》作用域闭包
date: 2017-10-26 11:29:06
categories:
- 读书笔记
- 《你不知道的JavaScript-上》
tags:
- JavaScript
- 系统学习
description: 闭包和词法作用域密切相关
---
## 启示
闭包是基于词法作用域书写时代码所产生的自然结果。    
闭包的创建和使用在你的代码中随处可见。  

## 实质问题
**当函数可以记住并访问所在的词法作用域时，就产生了闭包，即使函数是在当前词法作用域之外执行。**  
```javascript
function foo(){
    var a=2;
    function bar(){
        console.log(a);  //2
    }
    
    bar();
}

foo();
```
这不是一个纯粹的闭包，`bar()`对a的引用是词法作用域的查找规则（RHS），只是闭包的一部分。
```javascript
function foo(){
    var a=2;
    
    function bar(){
        console.log(a);
    }
    
    return bar;
}

var baz=foo();

baz();    //2——朋友，这就是闭包效果。
```
`foo()`执行后，通常其整个内部作用域都会被销毁，但实际上`foo()`内部作用域依然存在，被嵌套在foo函数内的`bar()`引用，这个引用就叫做闭包。   
    
## 现在我懂了
本质上，无论何时何地，如果将（访问他们各自词法作用域的）函数当作第一级的值类型到处传递，你就会看到闭包在这些函数中的应用。  
    
定时器、事件监听器、Ajax请求、跨窗口通信、Web Workers或者任何其他异步（或者同步）任务中，只要使用了回调函数，实际上就是在使用闭包。

## 循环和闭包
```javascript
for(var i=1;i<=5;i++){
    setTimeout(function timer(){
        console.log(i);
    },i*1000);
}
```
预期是，分别输出1～5，每秒一次，每次一个；
实际上，每秒一次的频率输出6次。     
尽管循环中的五个函数在各个迭代中分别定义的，但是它们都被封闭在一个共享的全局作用域中，因此实际上只有一个i。     
解决方法：
```javascript
for(var i=1;i<=5;i++){
    (function(j){
        setTimeout(function timer(){
            console.log(j);
        },j*1000);  
    })(i);
}

//或者
for(var i=1;i<=5;i++){
    (function(){
        var j=i;
        setTimeout(function timer(){
            console.log(j);
        },i*1000);  
    })();
}
```
在迭代内使用IIFE会为每个迭代都生成一个新的作用域，使得延迟函数的回调可以将新的作用域封闭在每个迭代内部，每个迭代中都会含有一个具有正确值的变量供我们访问。

### 重返块作用域
迭代内使用IIFE会为每个迭代都生成一个新的作用域==每次迭代需要一个块作用域。本质上，将一个块转换成一个可以被关闭的作用域。
```javascript
for(var i=1;i<=5;i++){
    let j=i;
    setTimeout(function timer(){
        console.log(j);
    },j*1000);
}
```
`for`循环头部的`let`声明有一个特殊的行为：      
变量在循环过程中不止被声明一次，每次迭代都会声明。随后的每个迭代都会使用上一个迭代结束时的值来初始化这个变量。
```javascript
for(let i=1;i<=5;i++){
    setTimeout(function timer(){
        console.log(i);
    },i*1000);
}
```


## 模块
```javascript
function CoolModule() {
    var something = "cool"; 
    var another = [1, 2, 3];
    
    function doSomething() { 
        console.log( something );
    }
    function doAnother() {
        console.log( another.join( " ! " ) );
    }
    
    return {
        doSomething: doSomething, 
        doAnother: doAnother
    };
}

var foo = CoolModule(); 

foo.doSomething(); // cool
foo.doAnother(); // 1 ! 2 ! 3
```
    
可以将返回的对象类型的值看作本质上的模块的公共API。
    
模块模式需要具备两个必要条件：
1. 必须有外部的封闭函数，该函数必须至少调用一次（每次调用都会创建一个新的模块实例）。
2. 封闭函数必须返回至少一个内部函数，这样内部函数才能在私有作用域中形成闭包，并且可以访问或者修改私有状态。
    
当只需要一个实例时，对模式改进成单例模式：
```javascript
var foo = (function CoolModule() { 
    var something = "cool";
    var another = [1, 2, 3];
    function doSomething() { 
        console.log( something );
    }
    function doAnother() {
        console.log( another.join( " ! " ) );
    }
    return {
        doSomething: doSomething, 
        doAnother: doAnother
    };
})();

foo.doSomething(); // cool 
foo.doAnother(); // 1 ! 2 ! 3
```
    
模块也是普通函数，也可以接受函数。  
    
模块另一个强大的用法：修改将要作为公共API返回的对象。   
通过在模块实例的内部保留对公共API对象的内部引用，可以从内部对模块实例进行修改。
```javascript
var foo=(function coolModule(id){
    function change(){
        //修改公共API
        publicAPI.identify=identify2;
    }
    
    function identify1(){
        console.log(id);
    }
    
    function identify2(){
        console.log(id.toUpperCase());
    }
    
    var publicAPI={
        change: change,
        identify: identify1
    }
    
    return publicAPI;
})("foo module");
```

### 现代的模块机制
大多数模块依赖加载器/管理器本质上都是将这种模块定义封装到一个友好的API。
```javascript
var myModule=(function(){
    var modules={};
    
    function define(name,deps,impl){
        for(var i=0;i<deps.length;i++){
            deps[i]=modules[deps[i]];
        }
        modules[name]=impl.apply(impl,deps);
    }
    
    function get(name){
        return modules[name];
    }
    
    return {
        define: define,
        get: get
    };
})();
```
为了模块的定义引入了包装函数（可以传入任何依赖）`modules[name]=impl.apply(impl,deps)`,并且将返回值，也就是模块API，储存在一个根据名字来管理的模块列表中。