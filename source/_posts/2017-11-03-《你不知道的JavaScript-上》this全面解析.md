---
title: 《你不知道的JavaScript-上》this全面解析
date: 2017-11-03 14:44:38
categories:
- 读书笔记
- 《你不知道的JavaScript-上》
tags:
- JavaScript
- 系统学习
description: 记住this四种调用方式，也没那么难
---
每个函数的this是在调用时被绑定的，完全取决于函数的调用位置（也就是函数的调用方法）。

## 调用位置
怎么找调用位置？    
通过分析调用栈（就是为了到达当前执行位置所调用的所有函数），来找到当前正在执行的函数的前一个调用，这个调用就是调用位置。
```javascript
function baz(){
    //当前调用栈是：baz
    //因此，当前调用位置是全局作用域
    
    console.log("baz");
    bar();   // <-- bar的调用位置
}

function bar(){
    //当前调用栈是：baz -> bar
    //因此，当前调用位置在baz中
    
    console.log("bar");
    foo();   // <-- foo的调用位置    
}

function foo(){
    //当前调用栈是：baz -> bar -> foo
    //因此，当前调用位置是在bar中
    
    console.log("foo");
}

baz();   // <-- baz的调用位置
```


## 绑定规则
必须先找到调用位置，然后判断需要应用下面四条规则中的哪一条。

### 1.默认绑定  
最常用的函数调用类型：**独立函数调用**。这条规则是无法应用其他规则时的默认规则。
```javascript
function foo(){
    console.log(this.a);
}

var a=2;
foo();  //2
```
如果使用严格模式（strict mode），则不能将全局对象用于默认绑定，因此`this`会绑定到`undefined`。   
一个重要的细节：在严格模式下调用（非运行）`foo()`则不影响默认绑定。     
        
### 2.隐式绑定     
调用位置是否有上下文对象，或者说是否被某个对象拥有或者包含，隐式绑定规则会把函数调用中的`this`绑定到这个上下文对象。
```javascript
function foo(){
    console.log(this.a);
}

var obj={
    a:2,
    foo:foo
};

obj.foo();   //2
```
对象属性引用链中只有上一层或者说最后一层在调用位置中起作用。
```javascript
function foo(){
    console.log(this.a);
}

var obj2={
    a:42,
    foo:foo
}

var obj1={
    obj2:obj2,
    a:2
}

obj1.obj2.foo();   //42
```

##### 2.1隐式丢失            
被隐式绑定的函数会丢失绑定对象，也就是说它会应用默认规则。
```javascript
function foo() { 
    console.log( this.a );
}
var obj = { 
    a: 2,
    foo: foo 
};
var bar = obj.foo; // 函数别名！
var a = "oops, global"; // a是全局对象的属性
bar(); // "oops, global"
```
bar引用的是foo函数本身，因此`bar()`是一个不带任何修饰的函数调用。   
传入回调函数时，几乎都会隐式丢失（如下两种）。
```javascript
function foo() { 
    console.log( this.a );
}
function doFoo(fn) {
    // fn其实引用的是foo
    fn(); // <-- 调用位置！
}
var obj = { 
    a: 2,
    foo: foo 
};
var a = "oops, global"; // a是全局对象的属性
doFoo( obj.foo ); // "oops, global"


function foo() { 
    console.log( this.a );
}
var obj = { 
    a: 2,
    foo: foo 
};
var a = "oops, global"; // a是全局对象的属性
setTimeout( obj.foo, 100 ); // "oops, global"
```

### 3.显示绑定     
`call(...)`和`apply(...)`称为显示绑定。     
如果传入一个原始值当作`this`的绑定对象，这个原始值会被转换成它的对象形式（`new String(..)`、`new Boolean(..)`、`new Number(..)`）。则通常称为装箱。        
    
##### 3.1 硬绑定    
硬绑定是一种显示的强制绑定，是一个函数。    
硬绑定可以解决隐式丢失问题。    

硬绑定的典型应用场景是创建一个包裹函数，负责接收参数并返回值。
```javascript
function foo(something){
    console.log(this.a,something);
    return this.a+something;
}

var obj={
    a:2
}

var bar=function(){
    return foo.apply(obj,arguments);
};

var b=bar(3);  //2 3
console.log(b);  //5
```
另一种使用方法是，创建一个可以重复使用的辅助函数。
```javascript
function foo(something){
    console.log(this.a,something);
    return this.a+something;
}

//简单的辅助绑定函数
function bind(fn,obj){
    return function(){
        return fn.apply(obj,arguments);
        }
    };
}

var obj={
    a:2
};

var bar=bind(foo,obj);

var b=bar(3)； //2 3
console.log(b);  //5
```
ES5内置bind方法`Function.prototype.bind`。      
        
##### 3.2 API调用的“上下文”      
一些函数提供一个可选的参数，通常称为“上下文”（context），其作用和`bind(..)`一样，确保你的回调函数使用指定的`this`。

    
### 4.new绑定           
使用`new`来调用函数，或者说发生构造函数调用时，会自动执行下面操作：     
1. 创建（或者说构造）一个全新的对象。
2. 这个对象会被执行`[[prototype]]`连接。
3. 这个对象会绑定到函数调用的`this`。
4. 如果函数没有返回其他对象，那么`new`表达式中的函数调用会自动返回这个对象。


## 优先级
如果某个调用位置可以应用多条规则该怎么办？      
        
先比较隐式绑定和显示绑定，谁的优先级高-->
```javascript
function foo(){
    console.log(this.a);
}

var obj1={
    a:2,
    foo:foo
};

var obj2={
    a:3,
    foo:foo
};

obj1.foo();  //2
obj2.foo();  //3

obj1.foo.call(obj2);  //3
obj2.foo.call(obj1);  //2
```
-->显示绑定高于隐式绑定。   
        
再比较new绑定和隐式绑定的优先级-->
```javascript
function foo(something){
    this.a=something;
}

var obj1={
    foo:foo
};

var obj2={};

obj1.foo(2);
console.log(obj1.a);  //2

obj1.foo.call(obj2,3);
console.log(obj2.a);   //3

var bar=new obj1.foo(4);
console.log(obj1.a);  //2
console.log(bar.a);  //4
```
-->new绑定比隐式绑定优先级高。      
        
最后比较new绑定和显示绑定-->
```javascript
function foo(something){
    this.a=something;
}

var obj1={};

var bar=foo.bind(obj1);
bar(2);
console.log(obj1.a);  //2

var baz=new bar(3);
console.log(obj1.a);  //2
console.log(baz.a);   //3
```
-->书上解释很多，结论一句话：硬绑定函数被`new`调用，会使用新创建的`this`（即新创建的对象）替换硬绑定的`this`。    
    
在`new`中使用硬绑定函数，主要目的是预先设置函数的一些参数，这样在使用`new`进行初始化时就可以只传入其余的参数。  
`bind(..)`的功能之一是可以把除了第一个参数之外的其他参数都传给下层函数。
```javascript
function foo(p1,p2) { 
        this.val = p1 + p2;
}
// 之所以使用null是因为在本例中我们并不关心硬绑定的this是什么
// 反正使用new时this会被修改
var bar = foo.bind( null, "p1" );

var baz = new bar( "p2" ); 

baz.val; // p1p2
```
    
### 判断this    
1. 函数是否在`new`中调用（`new`绑定）？如果是，`this`绑定的是新创建的对象。
2. 函数是否通过`call`、`apply`（显示绑定）或者硬绑定调用？如果是，`this`绑定的是指定的对象。
3. 函数是否在某个上下文对象中调用（隐式绑定）？如果是，`this`绑定的是那个上下文对象。
4. 如果都不是，使用默认绑定。如果在严格模式下，就绑定到`undefined`，否则绑定到全局对象。



## 绑定例外
### 1.被忽略的this
把`null`或者`undefined`作为`this`的绑定对象传入`call`、`apply`或者`bind`，这些值在调用时会被忽略，实际应用的是默认绑定规则，绑定全局对象。    
    
那么什么情况下会传入`null`？    
解构时
```javascript
function foo(a,b){
    console.log("a:"+a+",b:"+b);
}

//把数组“展开”成参数
foo.apply(null,[2,3]);   //a:2,b:3

//使用bind(..)进行柯里化
var bar=foo.bind(null,2);
bar(3);    //a:2,b:3
```
使用`null`来忽略this绑定可能产生一些副作用，可能会用默认规则把this绑定到全局对象。                    
    
##### 更安全的this
创建一个空的非委托的对象`Object.create(null)`。     
`Object.create(null)`和`{}`很像，但是并不会创建`Object.prototype`这个委托，所以它比`{}`“更空”。     
        
### 2.间接引用      
（有意或无意）创建一个函数的“间接引用”时，调用这个函数会应用默认绑定规则。
```javascript
function foo(){
    console.log(this.a);
}

var a=2;
var o={a:3,foo:foo};
var p={a:4};

o.foo();  //3
(p.foo=o.foo)();  //2
```
赋值表达式`p.foo=o.foo`的返回值是目标函数的引用，因此调用位置是`foo()`而不是`p.foo()`和`o.foo()`。      
*注意：*  对于默认绑定，函数体处于严格模式下，`this`会绑定到`undefined`;调用位置是否处于严格模式，则无影响。      
        
###  3.软绑定       
原理：给默认绑定指定一个全局对象和`undefined`以外的值，那就可以实现和硬绑定相同的效果，同时保留隐式绑定或者显示绑定修改`this`的能力。      
也就是四种规则中，默认绑定不再是全局对象或者undefined，而是软绑定那个值，同时其他3种规则不变。
    
以下是软绑定的实现
```javascript
if(!Function.prototype.softBind){
    Function.prototype.softBind=function(obj){
        var fn=this;
        //捕获所有curried参数
        var curried=[].slice.call(arguments,1);
        var bound=function(){
            return fn.apply(
                (!this||this===(window||global))?obj:this,
                curried.concat.apply(curried,arguments)
            );
        };
        
        bound.prototype=Object.create(fn.prototype);
        retunr bound;
    };
}
```
以下是软绑定功能例子
```javascript
function foo(){
    console.log("name:"+this.name);
}

var obj={name:"obj"},
    obj2={name:"obj2"},
    obj3={name:"obj3"};

var fooOBJ=foo.softBind(obj);

fooOBJ();  //name:obj

obj2.foo=foo.softBind(obj);
obj2.foo();  //name:obj2

fooOBJ.call(obj3);   //name:obj3，此时应该使用这个新的 this 作为 foo 的 this,而不是默认的 obj。

setTimeout(obj2.foo,10);   //name:obj  <--- 应用了软绑定
```


## this词法
箭头函数不使用`this`的四种标准规则，而是根据外层（函数或者全局）作用域来决定`this`。
```javascript
function foo(){
    //返回一个箭头函数
    return (a)=>{
        //this继承自foo()
        console.log(this.a);
    };
}

var obj1={a:2};

var obj2={a:3};

var bar=foo.call(obj1);
bar.call(obj2);   //2，不是3！
```
foo内部创建的箭头函数会捕获调用时foo()的this，且箭头函数的绑定无法被修改。      
    
箭头函数最常用于回调函数中，例如事件处理器或者定时器：
```javascript
function foo(){
    setTimeout(()=>{
        //这里this在词法上继承自foo()
        console.log(this.a);
    },100);
}

var obj={a:2};

foo.call(obj); //2
```

## 小结
如果要判断一个运行中函数的this绑定，就需要找到这个函数的直接调用位置。找到之后就可以顺序应用下面四条规则来判断this的绑定对象。
1. 由`new`调用？绑定到新创建的对象。
2. 由`call`或者`apply`(或者`bind`)调用？绑定到指定的对象。
3. 由上下文对象调用？绑定到那个上下文对象。
4. 默认：在严格模式下绑定到`undefined`，否则绑定到全局对象。    
        
安全的忽略this绑定，使用`φ=Object.create(null)`。   
    
ES6的箭头函数不会使用四条规则，而是根据当前的词法作用域来决定`this`。具体说，箭头函数会继承外层函数调用的`this`绑定，这其实和`self=this`机制一样。