---
title: 《你不知道的JavaScript-上》附录class
date: 2017-11-15 09:20:51
categories:
- 读书笔记
- 《你不知道的JavaScript-上》
tags:
- JavaScript
- 系统学习
description: class只是ES6中的语法糖
---
可以用一句话总结本书的第二部分（第4章至第6章）：类是一种可选（而不是必须）的设计模式，而且在JavaScript这样的[[Prototype]] 语言中实现类是很别扭的。    
一是因为语法上的缺点：繁琐杂乱的.prototype引用、试图调用原型链上层同名函数时的显式伪多态（参见第4章）以及不可靠、不美观而且容易被误解成“构造函数”的.constructor 。  
                
二是传统面向类的语言中父类和子类、子类和实例之间其实是复制操作，但是在[[Prototype]] 中并没有复制，相反，它们之间只有委托关联。  
    
    
### A.1 class
我们会介绍class原理并分析其是否改进了上面提到的2点缺点。        
        
首先回顾一下第6章中的Widget /Button 例子：
```javascript
class Widget { 
    constructor(width,height) {
        this.width = width || 50; 
        this.height = height || 50; 
        this.$elem = null;
    }
    render($where){
        if (this.$elem) { 
            this.$elem.css( {
                width: this.width + "px",
                height: this.height + "px" 
            } ).appendTo( $where );
        } 
    }
}
class Button extends Widget { 
    constructor(width,height,label) {
        super( width, height );
        this.label = label || "Default";
        this.$elem = $( "<button>" ).text( this.label );
    }
    render($where) {
        super( $where );
        this.$elem.click( this.onClick.bind( this ) ); 
    }
    onClick(evt) {
        console.log( "Button '" + this.label + "' clicked!" );
    } 
}
```
除了语法更好看 好看 之外，ES6还解决了什么问题呢？   
1. 不再引用杂乱的`.prototype `了。
2. Button 声明时直接“继承”了Widget ，不再需要通过`Object.create(..)` 来替换.prototype 对象，也不需要设置`.__proto__`或者`Object.setPrototypeOf(..)` 。
3. 可以通过`super(..)` 来实现相对多态，这样任何方法都可以引用原型链上层的同名方法。
4. class 字面语法不能声明属性（只能声明方法）。如果没有这种限制的话，原型链末端的“实例”可能会意外地获取其他地方的属性（这些属性隐式被所有“实例”所“共享”）。
5. 可以通过extends 很自然地扩展对象（子）类型，甚至是内置的对象（子）类型，比如Array 或RegExp 。     
        
    
    
### A.2 class陷阱
class 语法并没有解决所有的问题，在JavaScript中使用“类”设计模式仍然存在许多深层问题。    
    
class并没有引入新的“类”机制， 基本上只是现有[[Prototype]] （委托！）机制的一种语法糖。               
    
也就是说，class 并不会像传统面向类的语言一样在声明时静态复制所有行为。如果你（有意或无意）修改或者替换了父“类”中的一个方法，那子“类”和所有实例都会受到影响，因为它们在定义时并没有进行复制，只是使用基于[[Prototype]] 的实时委托：
```javascript
class C {
    constructor(){
        this.num=Math.random();
    }
    rand(){
        console.log("Random: "+this.num);
    }
}

var c1=new C();
c1.rand();  // "Random: 0.4324299..."

C.prototype.rand=function(){
    console.log("Random: "+Math.round(this.num*1000));
};

var c2=new C();
c2.rand(); // "Random: 867" 
c1.rand(); // "Random: 432" ——噢！
```
    
        
此外，ES6中的class还有其他隐患：       
1. class 语法无法定义类成员属性（只能定义方法），如果为了跟踪实例之间共享状态，必须要这么做的话，那你只能使用丑陋的.prototype 语法，像这样：
```javascript
class C {
    constructor(){
        // 确保修改的是共享状态而不是在实例上创建一个屏蔽属性！
        C.prototype.count++;
        
        // this.count可以通过委托实现我们想要的功能
        console.log("Hello: "+this.count);
    }
}

// 直接向prototype对象上添加一个共享状态
C.prototype.count=0;

var c1 = new C();  // Hello: 1
var c2 = new C();  // Hello: 2

c1.count === 2; // true
c1.count === c2.count; // true
```
这种方法最大的问题是，它违背了class 语法的本意，在实现中暴露（泄露！）了`.prototype` 。         
        
如果使用`this.count++ `的话，我们会很惊讶地发现在对象c1 和c2 上都创建了.count 属性，而不是更新共享状态。class没有办法解决这个问题，并且干脆就没有提供相应的语法支持。     
    
2. class 语法仍然面临意外屏蔽的问题：
```javascript
class C { 
    constructor(id) {
        // 噢，郁闷，我们的id属性屏蔽了id()方法
        this.id = id;
    }
    id() {
        console.log( "Id: " + id );
    }
} 
var c1 = new C( "c1" );
c1.id(); // TypeError -- c1.id现在是字符串"c1"
```
3. super 并不是像this一样的动态绑定，它会在声明时“静态”绑定。此外，根据应用方式的不同，super 可能不会绑定到合适的对象（至少和你想的不一样），所以你可能（写作本书时，TC39
正在讨论这个话题）需要用toMethod(..) 来手动绑定super。
```javascript
class P {
    foo() { console.log( "P.foo" ); }
}
class C extends P { 
    foo() {
        super(); 
    }
}
var c1 = new C();
c1.foo(); // "P.foo"

var D = {
    foo: function() { console.log( "D.foo" ); }
};
var E = {
    foo: C.prototype.foo
};

// 把E委托到D
Object.setPrototypeOf( E, D ); 

E.foo(); // "P.foo"
```
出于性能考虑，super 并不像this 一样是晚绑定（late bound， 或者说动态绑定）的，它在`[[HomeObject]].[[Prototype]]` 上，`[[HomeObject]] `会在创建时静态绑定。           
            
在本例中，`super() `会调用`P.foo()`，因为方法的`[[HomeObject]]`仍然是C，`C.[[Prototype]] `是P 。       
        
可以 手动修改super 绑定，使用`toMethod(..) `绑定或重新绑定方法的[[HomeObject]] （就像设置对象的[[Prototype]] 一样！）就可以解决本例的问题：
```javascript
var D = {
    foo: function() { console.log( "D.foo" ); }
};

// 把E委托到 D
var E = Object.create( D );

// 手动把foo的[[HomeObject]]绑定到E，E.[[Prototype]]是D， 所以 super()是D.foo()
E.foo = C.prototype.foo.toMethod( E, "foo" );

E.foo(); // "D.foo"
```
    
        
### A.3 静态大于动态
在传统面向类的语言中，类定义之后就不会进行修改，所以类的设计模式就不支持修改。但是JavaScript最强大的特性之一就是它的动态性，任何对象的定义都可以修改（除非你把它设置成不可变）。      
但是class 似乎想告诉你：“动态太难实现了，所以这可能不是个好主意。这里有一种看起来像静态的语法，所以编写静态代码吧。”       
    
class显得有点不伦不类。         
        
总地来说，ES6的class 想伪装成一种很好的语法问题的解决方案，但是实际上却让问题更难解决而且让JavaScript更加难以理解。



### A.4 小结
class 很好地伪装成JavaScript中类和继承设计模式的解决方案，但是它实际上起到了反作用：它隐藏了许多问题并且带来了更多更细小但是危险的问题。          
        
class 加深了过去20年中对于JavaScript中“类”的误解，在某些方面，它产生的问题比解决的多，而且让本来优雅简洁的[[Prototype]] 机制变得非常别扭。        
        
结论：如果ES6的class 让[[Prototype]]变得更加难用而且隐藏了JavaScript对象最重要的机制——对象之间的实时委托关联，我们难道不应该认为class产生的问题比解决的多吗？难道不应该抵制这种设计模式吗？          

我无法替你回答这些问题，但是我希望本书能从前所未有的深度分析这些问题，并且能够为你提供回答问题所需的所有信息。