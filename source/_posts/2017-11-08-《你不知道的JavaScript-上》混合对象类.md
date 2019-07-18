---
title: 《你不知道的JavaScript-上》混合对象“类”
date: 2017-11-08 16:51:55
categories:
- 读书笔记
- 《你不知道的JavaScript-上》
tags:
- JavaScript
- 系统学习
description: 本章主要了解类的概念
---
本章会先介绍面向类的设计模式：实例化（instantiation）、继承（inheritance）、（相对）多态（polymorphism）。


## 类的理论
类/继承描述了一种代码组织结构形式————一种在软件中对真实世界中问题领域的建模方法。   
    
面向对象编程强调**数据和操作数据的行为本质上的互相关联的**，因此好的设计就是把数据以及和它相关的行为打包（封装）起来。      
例如，所有字符串是String类的一个实例，是一个包裹，包含字符数据和我们可以应用在数据上的函数（行为）。          
    
什么是类、继承、和实例化？      
一个常见的例子，“汽车”可以被看作“交通工具”的一个特例，后者是更广泛的类。    
在软件中，先定义一个Vehicle类，定义Car时，只要声明它继承（或扩展）了Vehicle的基础定义就行。Car的定义就是对通用Vehicle定义的特殊化。      
虽然Vehicle和Car会定义相同的方法，但是实例中的数据可能是不同的。        

类的另一个核心概念是多态：父类的通用行为可以被子类用更特殊的行为重写。实际上，相对多态性允许我们从重写行为中引用基础行为。    
        
        
### 1.“类”的设计模式            
软件设计中，类是一种可选的设计模式，另一种是函数式编程。    
Java中类并不是可选的————万物皆是类。C/C+或者PHP会提供过程化和面向类两种语法。   
    
### 2.JavaScript中的“类”       
JS中没有类！    
类是一种设计模式，可以通过语法糖和“类”库近似实现类，不过其他语言中的类和JS中的“类”并不一样，在近似类的表象下，JS的机制其实和类完全不同。      

    
    
## 类的机制
许多面向类的语言中，标准库会提供Stack类，它是一种“栈”数据结构。Stack类内部会有一些变量来存储数据，同时会提供一些公有的可访问行为（“方法”）。    
但这些语言中，你并不是实际操作Stack，而是操作Stack类的一个实例。    
    
        
###  1.建造     
“类”和“实例”的概念来源于房屋建造。      
    
建筑工人按照蓝图建造建筑，完成后，建筑就成为了蓝图的物理实例，实际上，就是把蓝图上规划好的特性从蓝图中复制到现实世界的建筑中。    
    
一个类就是一张蓝图，一个实例就是一栋建筑。      

### 2.构造函数   
类实例是由一个特殊的类方法构造的，这个方法名通常和类名相同，被称为**构造函数**。这个方法任务就是*初始化实例需要的所有信息（状态）*。           
    
类构造函数属于类，而且通常和类同名。此外，构造函数大多数需要用new来调用。
    
    
## 类的继承
面向类的语言中，你可以先定义一个类（通常称为“父类(DNA)”），然后定义一个继承前者的类（通常称为“子类(DNA)”）。       
    
定义好一个子类之后，相对于父类来说它就是一个独立并且完全不同的类。子类会包含父类行为的原始副本，但是也可以重写所有继承的行为甚至定义新行为。    
    
非常重要一点！！ ——**我们讨论的父类和子类并不是实例。**         
关于不同类型交通工具的伪代码
```java
class Vehicle{
    engines=1
    
    igition(){
        output("Turning on my engine.");
    }
    
    drive(){
        iginition();
        output("Steering and moving forward!")
    }
}

class Car inherits Vehicle{
    wheels=4
    
    drive(){
        inherited:drive()
        output("Rolling on all",wheels,"wheels")
    }
}

class SpeedBoat inherits Vehicle{
    engines=2
    
    ignition(){
        output("Turning on my",engines,"engines.")
    }
    
    pilot(){
        inherited:drive()
        output("Speeding through the water with ease!")
    }
}
```
    
### 1.多态      
Car重写了继承自父类的`drive()`方法，但是Car调用了`inherited:drive()`方法，这表明Car可以引用继承来的原始`drive()`方法。       
这个技术被称为*多态*或者*虚拟多态*。本例中，更恰当叫*相对多态*。    
    
任何方法都可以引用继承层次中高层的方法。“相对”指我们不会定义想要访问的绝对继承层次，而是使用相对引用“查找上一层”。      
        
许多语言中，`super`代替`inherited:`，它的含义是“超类”，表示当前类的父类/祖先类。        
        
在继承链的不同层次中一个方法名可以被多次定义，当调用方法时会自动选择合适的定义。代码例子中的两个drive()和两个ignition()。       
    
一个问题，在`polot()`中通过相对多态引用了（继承来的）Vehicle中的`dirve()`。但是那个`dirve()`方法直接通过名字引用了`ignition()`方法。那么语言引擎会使用哪个`ignition()`呢？      
它会使用SpeedBoat的`ignition()`。如果你直接实例化了Vehicle类然后调用它的`drive()`，那么语言引擎就会使用Vehicle中的`ignition()`方法。       
换言之，`ignition()`方法定义的多态性取决于你是哪个类的实例中引起它。        

在子类中也可以相对引用它继承的父类，这种相对引用通常被称为`super`。     
需要注意，子类得到的仅仅是继承自父类行为的一份副本。        
类的继承其实就是复制。
    
![](http://img.aisss.top/17-10-22/52022745.jpg)     
注意实例a1、a2、b1、b2和继承bar，箭头表示复制操作。     
    
    
### 2.多重继承     
多重继承指：有些面向类的语言允许你继承多个“父类”，意味着所有父类的定义都会被复制到子类中。      
多个父类情况下，多重继承会导致不能确定引用哪个父类方法的问题。      
    
JS并不提供“多重继承”功能。但开发者有各种各样方法来实现多重继承。    
        
            
            
## 混入
继承或者实例化时，JS的对象机制并不会自动执行复制行为。简单说，JS中只有对象，并不存在可以被实例化的“类”。一个对象并不会被复制到其他对象，他们会被关联起来。      
但其他语言中表现出来都是复制行为，因此JS开发者想出模拟类的复制行为，就是**混入**。      
两种类型的混入：显式和隐式。    
    
    
### 1.显式混入     
```javascript
//非常简单的mixin(..)例子
function mixin(sourceObj,targetObj){
    for(var key in sourceObj){
        //只会在不存在的情况下复制
        if(!(key in targetObj)){
            targetObj[key]=sourceObj[key];
        }
    }
    
    return targetObj;
}

var Vehicle={
    engines：1，
    
    ignition:function(){
        console.log("Turning on my engine.");
    },
    
    drive:function(){
        this.ignition();
        console.log("Steering and moving forward!");
    }
};

var Car=mixin(Vehicle,{
    wheels:4,
    
    drive:function(){
        Vehicle.drive.call(this);
        console.log(
            "Rolling on all"+this.wheels+"wheels!"
        );
    }
});
```
Car中有一份Vehicle属性和函数的副本了。Car中的drive重写了父类中的drive。     
    
##### 1.1再说多态       
`Vehicle.drive.call(this)`是显式（伪）多态。      
应当避免使用显式伪多态。    
        
##### 1.2混合复制    
只在能够提高代码可读性的前提下使用显式混入，避免使用增加代码理解难度或者让对象关系更加复杂的模式。      
    
##### 1.3寄生继承   
显式混入模式的一种变体被称为“寄生继承”，它既是显式也是隐式的，主要推广者是Douglas Crockford.      
下面是它的工作原理：
```javascript
//"传统的javascript类"Vehicle
function Vehicle(){
    this.engines=1;
}

Vehicle.prototype.ignition=function(){
    console.log("Turning on my engine.");
};
Vehicle.prototype.drive=function(){
    this.ignition();
    console.log("Steering and moving forward!");
};


//"寄生类"Car
function Car(){
    //首先，car是一个vehicle
    var car=new Vehicle();
    
    //接着我们对car进行定制
    car.wheels=4;
    
    //保存到Vehicle：：drive()的特殊引用
    var vehDrive=car.drive;
    
    //重写Vehicle::drive()
    car.drive=function(){
        vehDrive.call(this);
        console.log("Rolling on all"+this.wheels+"wheels!");
    }
    
    return car;
}

var myCar=new Car();

myCar.drive();
//Turning on my engine.
//Steering and moving forward!
//Rolling on all4wheels!
```
首先复制一份vehicle父类（对象）的定义，然后混入子类（对象）的定义，然后用这个复合对象构建实例。    
    
    
        
### 2.隐式混入     
隐式混入和之前提到的显式伪多态很像，也具备同样的问题。      
```javascript
var Something={
    cool:function(){
        this.greeting="Hello World";
        this.count=this.count?this.count+1:1;
    }
};

Something.cool();
Something.greeting;  //"Hello World"
Something.count;  //1 

var Another={
    cool:function(){
        //隐式把Something混入Another
        Something.cool.call(this);
    }
};

Another.cool();
Another.greeting; //"Hello World"
Another.count;    //1(count不是共享状态)
```
通过使用`Something.cool.call(this)`，让函数`Something.cool()`在Another的上下文中调用了它，最终的结果是`Something.cool()`中的赋值操作都会应用在Another对象上而不是Something对象上。      
        
因此，我们把Something行为“混入”到Another中。        
        
尽量避免使用这种结构，以保证代码的整洁和可维护性。      
    
    
    
## 小结
类是一种设计模式。许多语言提供了面向类软件设计的原生语法。JavaScript也有类似的语法，但是和其他语言中的类完全不同。  
    
类意味着复制。      
    
传统的类被实例化时，它的行为会被复制到实例中。类继承时，行为也会被复制到子类中。    
    
多态（在继承链的不同层次名称相同但是功能不同的函数）看起来似乎是从子类引用父类，但是本质上引用的其实是复制的结果。  
    
JavaScript并不会（像类那样）自动创建对象的副本。    
    
混入模式（无论显式还是隐式）可以用来模拟类的复制行为，但是通常会产生丑陋并且脆弱的语法。，比如显式伪多态。      

此外，显式混入实际上无法完全模拟类的复制行为，因为对象只能复制引用，无法复制被引用的对象或者函数本身。忽视这一点会导致许多问题。        
    
总的来说，在JavaScript中模拟类是得不偿失，虽然能解决当前问题，但是可能会埋下更多隐患。
