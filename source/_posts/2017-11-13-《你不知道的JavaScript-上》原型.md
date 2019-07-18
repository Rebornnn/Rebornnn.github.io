---
title: 《你不知道的JavaScript-上》原型
date: 2017-11-13 09:15:00
categories:
- 读书笔记
- 《你不知道的JavaScript-上》
tags:
- JavaScript
- 系统学习
description: 原型机制应该理解为“委托”而不是“继承”
---
## [[Prototype]]
JS中，几乎所有的对象在创建时都会被赋予一个非空的[[Prototype]]属性，[[Prototype]]其实就是对于其他对象的引用。        
    
[[Prototype]]引用有什么用？     
对象的[[Get]]操作、for..in遍历（且是enumerable）、in操作符(无论是否可枚举)都会查找对象的整条原型链来寻找属性。    
    
    
### 1.Object.prototype    
哪里是[[Prototype]]的“尽头”？————内置的`Object.prototype`。   
    
所有“普通”（内置，不是特定主机的扩展）对象都“源于”（或者说把[[Prototype]]链的顶端设置为）这个`Object.prototype`对象。      
`Object.prototype`对象包含许多JS中通用的功能，`.toString()`、`.valueOf()`、`.hasOwnProperty(..)`、`isPrototypeOf(..)`等。  
    
### 2.属性设置和屏蔽 
给一个对象设置属性并不仅仅是添加一个新属性或者修改已有的属性值。    
> myObject.foo="bar";       

这条语句可以分为4种情况：   
1. foo直接存在于myObject对象中的普通数据访问属性，这条赋值语句会修改已有的属性值。        
2. foo不是直接存在于myObject中，[[Prototype]]链会被遍历，如果原型链上找不到foo，foo会被直接添加到myObject上。    
3. foo即存在于myObject中，也出现在原型链上层，那么会发生屏蔽。      
4. foo不存在于myObject中，而出现在原型链上层，则会有以下3种情况：   
    4.1 如果原型链上层存在foo的普通数据访问属性并且没有被标记为只读(writable:true)，那就会在myObject中添加一个名为foo的新属性，是屏蔽属性。     
    4.2 如果原型链上层存在foo，但是他被标记为只读(writable:false),那么无法修改已有属性或者在myObject上创建屏蔽属性。(可用Object.defineProperty())        
    4.3 如果原型链上存在foo并且是一个setter，那就会调用这个setter。foo不会被添加到（或者说屏蔽于）myObject，也不会重新定义foo这个setter。(可用Object.defineProperty())      
    
    
有些情况会隐式产生屏蔽。
```javascript
var anotherObject={
    a:2
};

var myObject=Object.create(anotherObject);

anotherObject.a;  //2
myObject.a;    //2

anotherObject.hasOwnProperty("a");  //true
myObject.hasOwnProperty("a");   //false

myObject.a++;  //隐式屏蔽！

anotherObject.a;  //2
myObject.a;   //3

myObject.hasOwnProperty("a");   //true
```
++操作相当于`myObject.a=myObject.a+1`,首先会通过`[[Prototype]]`查找属性a并从`anotherObject.a`获取当前属性值2并加1，接着用`[[Put]]`将值3赋给myObject中新建的屏蔽属性a。   
    
    
## “类”
JavaScript中没有类，只有对象，它可以不通过类，直接创建对象。对象直接定义自己的行为。
    
    
### “类”函数
JavaSript中一种奇怪的行为一直被无耻的滥用，那就是模仿类。   
    
这种奇怪的“类似类”的行为利用了函数的一种特殊特性：所有的函数默认都会拥有一个名为`prototype`的公有并且不可枚举的属性，它会指向另一个对象,这个对象通常被称为函数的原型。     
这个对象（就是原型）是在调用`new Foo()`后与`[[Prototype]]`关联。
```javascript
function Foo(){
    //...
}

var a=new Foo();

Object.getPrototypeOf(a)===Foo.prototype;  //true
```
调用`new Foo()`会创建a，其中一步是将a内部的`[[Prototype]]`链接到`Foo.prototype`所指向的对象。        
    
`new Foo()`这个函数调用实际上并没有直接创建关联，这个关联只是一个意外的副作用（将`[[Prototype]]`和`Foo.prototype`关联）。`new Foo()`只是间接完成了我们我们的目标：一个关联到其他对象的新对象。            
        
**关于名称**              
在JavaScript中，并不会将一个对象（“类”）复制到另一个对象（“实例”），只是将他们关联起来，这个机制通常被称为原型继承。        
    
原型继承，并不准确。仅仅加上“原型”并不难区分JS中与类继承几乎完全相反的行为。    
    
继承意味着复制操作，JS（默认）并不会复制对象属性。相反，JS会在两个对象之间创建一个关联，这样一个对象就可以通过委托访问另一个对象的属性和函数。      
委托这个术语可以更加准确的描述JS中对象的关联机制。
    
    
### “构造函数”
```javascript
function Foo(){
    //...
}

var a=new Foo();
```
到底什么让我们认为Foo是一个“类”呢？  
1. 使用了关键字new
2. Foo()的调用方式很像初始化类时类构造函数的调用方式
3. 有公有且不能枚举的constructor属性
4. 首字母大写   
    
    
**1. 构造函数还是调用**     
构造函数和其他普通函数没啥区别，实际上，new劫持所有普通函数并用构造对象的形式调用它。   
```javascript
function NothingSpecical(){
    console.log("Don't mind me!");
}

var a=new NothingSpecical();   //Don't mind me!

a;  //{}
```
函数不是构造函数，但是当且仅当使用`new`时，函数调用会变成“构造函数调用”。
    
    
### 技术
```javascript
function Foo(name){
    this.name=name;
}

Foo.prototype.myName=function(){
    return this.name;
};

var a=new Foo("a");
var b=new Foo("b");

a.myName();  //"a"
b.myName();  //"b"
```
这段代码展示另外两种“面向类”的技巧：
1. `this.name=name`给每个对象都添加了`.name`属性，有点像类实例封装的数据值。
2. `Foo.prototype.myName=...`，给`Foo.prototype`对象添加一个属性（函数）。      
        
    
**回顾“构造函数”**      
把`.constructor`属性指向`Foo`看作是a对象由`Foo`“构造”，其实是不准确的。`a.constructor`只是通过默认的`[[Prototype]]`委托指向`Foo`，这和“构造”毫无关系。          
        
`Foo.prototype`的`.constructor`属性只是函数在声明时的默认属性。它可以被新对象替换。
```javascript
function Foo(){/*..*/}

Foo.prototype={/*..*/}; //创建一个新原型对象

var a1=new Foo();
a1.constructor===Foo;  //false
a1.constructor===Object;  //true
```
a1没有`.constructor`属性，所以会委托原型链上的`Foo.prototype`，但这个对象也没有，所以会继续委托给原型链顶端的`Object.prototype`，此对象的`.constructor`属性指向内置的`Object(..)`函数。    
    
`.constructor`非常不可靠且不安全。尽量避免使用。    
    
    
    
    
## （原型）继承
![](http://img.aisss.top/17-10-29/70348818.jpg)         
    
图中由下到上的箭头表明这是委托关联，不是复制操作。      
    
下面是典型的“原型风格”：   
```javascript
function Foo(name){
    this.name=name;
}

Foo.prototype.myName=function(){
    return this.name;
};

function Bar(name,label){
    Foo.call(this,name);
    this.label=label;
}

//创建一个新的Bar.prototype对象关联到Foo.prototype
Bar.proptotype=Object.create(Foo.prototype);

Bar.prototype.myLabel=function(){
    return this.label;
};

var a=new Bar("a","obj a");

a.myName();   //"a"
a.myLabel();  //"obj a"
```
这段代码的核心部分就是语句`Bar.prototype = Object.create( Foo.prototype ) `。     
调用`Object.create(..)`会凭空创建一个“新”对象并把新对象内部的`[[Prototype]]`关联到你指定的对象（本例中是Foo.prototype ）。        
        
*注意*：下面这两种方式是常见的错误做法
```javascript
// 和你想要的机制不一样！
// 这样并不会创建一个关联到Bar.prototype 的新对象
//它只是让Bar.prototype直接引用Foo.prototype 对象
Bar.prototype = Foo.prototype;

// 基本上满足你的需求，但是可能会产生一些副作用
//因为会new操作会调用执行Foo()函数
Bar.prototype = new Foo();
```
因此，要创建一个合适的关联对象，我们必须使用`Object.create(..)`。       
或者用ES6的`Object.setPropertyOf(..)`.
```javascript
// ES6之前需要抛弃默认的Bar.prototype
Bar.ptototype = Object.create( Foo.prototype );

// ES6开始可以直接修改现有的Bar.prototype
Object.setPrototypeOf( Bar.prototype, Foo.prototype );
```
    
    
### 检查“类”关系
在传统的面向类环境中，检查一个实例（JavaScript中的对象）的继承祖先（JavaScript中的委托关联）通常被称为内省（或者反射 ）。
```javascript
function Foo(){
    //...
}

Foo.prototype.blah=...;
var a=new Foo();
```
第一个方法站在“类”的角度来判断：
```javascript
a instanceof Foo;   //true
```
`instanceof`操作符回答的是：在a的整条`[[Prototype]]`链中是否有指向`Foo.prototype`的对象？        
可惜，这种方法只能处理对象和构造函数之间的关系。不能处理两个对象之间原型链关联问题。    
    
*注意*：使用内置`.bind(..)`函数生成的硬绑定函数是没有`.prototype`属性的。对其使用`instanceof`操作符时目标函数的`.prototype`会代替硬绑定函数的`.prototype`。     
        
第二个方法判断[[Prototype]]反射的方法
```javascript
Foo.prototype.isPrototypeOf(a);  //true
```
`isPrototypeOf(..)`操作符回答的是：在a的整条`[[Prototype]]`链中是否出现过`Foo.prototype`的对象？    
这种方法能处理两个对象之间原型链关联问题。      
        
我们也可以直接获取一个对象的[[Prototype]]链：   
```javascript
//ES5
Object.getPrototypeOf(a)===Foo.prototype;  //true

//ES6
a.__prototype__===Foo.prototype;  //true
```
`.__prototype__`存在于内置的`Object.prototype`中,而且很像一个getter/setter。        
`.__prototype__`大致实现是这样：
```javascript
Object.defineProperty(Object.prototype,"__prototype__",{
    get:function(){
        return Object.getPrototypeOf(this);
    }
    
    set:function(){
        Object.setPrototypeOf(this,o);
        return o;
    }
});
``` 
        
            
## 对象关联
`[[Prototype]]`机制就是存在于对象中的一个内部链接，它会引用其他对象。作用是形成原型链。   

### 创建关联
`[[Prototype]]`机制的意义是什么？为什么JS开发者费这么大力气（模拟类）在代码中创建这些关联？        
先说说`Object.create(..)`，它是一个大英雄。     
`Object.create(..)`会创建一个新对象并把它关联到我们指定的对象，这样就发挥`[[Prototype]]`机制的威力（就是委托）并且避免不必要的麻烦（比如new的构造函数调用会生成.prptotype和.constructor引用）。     
这样一来，我们不需要类（指JS中的构造函数，其他语言中的类）来创建两个对象之间的关系，只需要通过委托来关联对象就足够了。  
    
#### Object.create()的pollfill代码
ES5之前的环境想要支持此功能，得使用一段简单的polyfill代码，来部分实现`Object.create(..)`功能：
```javascript
if(!Object.create){
    Object.create=function(o){
        function F(){};
        F.prototype=o;
        return new F();
    };
}
```
其实就是高程里介绍的原型式继承。    
    
    
### 关联关系是备用
看起来对象之间的关联关系是处理“缺失”属性或方法时的一种备用选项。这个说法有点道理，但是我认为这并不是[[Prototype]]的本质。   
思考一下代码：
```javascript
var anotherObject = { 
    cool: function() {
        console.log( "cool!" );
    }
};

var myObject = Object.create( anotherObject );
myObject.cool(); // "cool!"
```
由于存在[[Prototype]] 机制，可以让myObject在无法处理属性或者方法时能够使用备用的anotherObject。         
当使用这样的备用模式时，假设要调用myObject.cool()，如果myObject中不存在cool()时这条语句也可以正常工作的话，那你的API设计就会变得很奇怪，对于未来维护你软件的开发者来说这可能不太好理解。          
    
以下用内部委托的方法可以解决：
```javascript
var anotherObject = { 
    cool: function() {
        console.log( "cool!" );
    }
};

var myObject = Object.create( anotherObject );

myObject.doCool = function() { 
    this.cool(); // 内部委托！
};

myObject.doCool(); // "cool!"
```
调用的myObject.doCool()是实际存在实际存在于myObject中的，这可以让我们的API设计更加清晰（不那么“神奇”）。从内部来说，我们的实现遵循的是委托设计模式，通过[[Prototype]] 委托
到anotherObject.cool()。    
    
        
        
## 小结
如果要访问对象中并不存在的一个属性，[[Get]]操作（参见第3章）就会查找对象内部[[Prototype]] 关联的对象。这个关联关系实际上定义了一条“原型链”（有点像嵌套的作用域链），在查找属性时会对它进行遍历。        
    
    
所有普通对象都有内置的Object.prototype，指向原型链的顶端（比如说全局作用域），如果在原型链中找不到指定的属性就会停止。`toString()`、`valueOf()`和其他一些通用的功能都存在于Object.prototype 对象上，因此语言中所有的对象都可以使用它们。    
    
JavaScript中的机制与传统面向类语言有一个核心区别，那就是不会进行复制，对象之间是通过内部的[[Prototype]] 链关联的。      
        
出于各种原因，以“继承”结尾的术语（包括“原型继承”）和其他面向对象的术语都无法帮助你理解JavaScript的真实机制（不仅仅是限制我们的思维模式）。
相比之下，“委托”是一个更合适的术语，因为对象之间的关系不是复制而是委托。