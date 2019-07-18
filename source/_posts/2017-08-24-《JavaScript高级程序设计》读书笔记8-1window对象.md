---
title: 《JavaScript高级程序设计》读书笔记8.1window对象
date: 2017-08-24 09:28:09
categories:
- 读书笔记
- 《JavaScript高级程序设计》
tags:
- JavaScript
- 系统学习
description: window对象有双重性
---
**BOM的核心是window,它表示浏览器的一个实例。**
- window有双重角色：
    - 即是通过JavaScript访问浏览器窗口的一个接口。
    - 也是ECMAScript规定的Global对象。

#### 1.全局作用域
**所有在全局作用域中声明的变量、函数都会变成window对象的属性和方法。**
```javascript
var age=29;
function sayAge(){
    alert(this.age);
}

alert(window.age);   //29
sayAge();            //29
window.sayAge();     //29
```
- 全局变量不能通过`delete`操作删除，而直接在`window`对象上的定义的属性可以。
- 尝试访问未声明的变量会抛出错误，但是通过查询window对象，可以知道某个可能未声明的变量是否存在。


#### 2.窗口关系及框架
- 如果页面中包含框架，则每个框架都拥有自己的`window`对象，并且保存在`frames`集合中。
- 每个`window`对象都有一个`name`属性，其中包含框架的名称。
- `top`对象始终指向最高（最外）层的框架，也就是浏览器窗口。
- 在一个框架编写的任何代码，其中的window对象指向的都是那个框架的特定实例。
- `parent`对象始终指向当前框架的直接上层框架。
- `self`对象始终指向`window`。
- 所有这些对象都是`window`对象的属性。

#### 3.窗口位置
- IE、Safari、Opera、Chrome提供`screenLeft`和`screenTop`属性，分别用于表示窗口相对于屏幕左边和上边的位置
- FireFox提供`screenX`和`screenY`属性。一样。

#### 4.窗口大小
`innerWidth`、`innerHeight`、`outerWidth`、`outerHeight`

#### 5.导航和打开窗口
**`window.open()`方法既可以导航到一个特定的URL，也可以打开一个新的浏览器窗口。**
- 接受4个参数
    1. 要加载的URL
    2. 窗口目标
    3. 一个特性字符串
    4. 一个表示新页面是否取代浏览器历史记录中当前加载页面的布尔值。
    - 通常只须传递第一个参数，最后一个参数只在不打开新窗口的情况下使用。 

##### 5.1弹出窗口
- 如果第二参数并不是一个已经存在的窗口或框架，那么该方法根据传入的字符串（第三个参数）创建一个新窗口或新标签。
- 如果没传入第三个参数，会打开带有全部默认设置的新窗口。
- 第三个参数是逗号分隔的字符串，设置如下
![clipboard.png](https://ooo.0o0.ooo/2017/05/09/5911b5ebbf5af.png)
- `window.open()`会返回一个指向新窗口的引用。

##### 5.2安全限制
弹出窗口配置方面增加限制。各个浏览器不同。



#### 6.间歇调用和超时调用
 超时调用`setTimeout()`：指定的时间过后执行代码。        
 
两个参数：第一个参数是要执行的代码；第二个参数是以毫秒表示的时间
```javascript
setTimeout(function(){
    alert("Hello world!");
},1000);
```
经过第二个参数等待时间后，指定代码不一定会执行。因为JavaScript是一个单线程的解释器，一定时间内只能执行一段代码。     

调用`setTimeout()`后，该方法会返回一个数值ID，表示超时调用。这个超时调用ID是计划执行代码的唯一标识符，可以通过它来取消超时调用。
```javascript
var timeoutId=setTimeout(function(){
    alert("Hello world!");
},1000);

//取消超时调用
clearTimeout(timeoutId);
```
函数中`this`在非严格模式下指向`window`对象，在严格模式下是`undefined`。           


间歇调用`setInterval()`：每隔指定的时间就执行一次。最好不使用。
```javascript
var num=0;
var max=10;
var intervalId=null;

function incrementNumber(){
    num++;
    
    if(num==max){
        clear(intervalId);
        alert("Done");
    }
}

intervalId=setInterval(incrementNumber,500);

使用超时模式来实现
var num=0;
var max=10;

function incrementNumber(){
    num++;
    
    if(num<max){
        setTimeout(incrementNumber,500);
    }else{
        alert("Done");
    }
}

serTimeout(incrementNumber,500);
```

#### 7.系统对话框
**`alert()`、`confirm()`、`prompt()`**
- `confirm()`:点"OK"会返回true，点"cancel"会返回false
- `prompt()`:显示一个文本输入域，以供用户输入。