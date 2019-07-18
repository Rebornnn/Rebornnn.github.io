---
title: 《你不知道的JavaScript-上》行为委托
date: 2017-11-14 09:32:04
categories:
- 读书笔记
- 《你不知道的JavaScript-上》
tags:
- JavaScript
- 系统学习
description: 行为委托设计模式的基础是JS中[[Prototype]]机制的本质是对象之间的关联关系。
---
JavaScript中[[Prototype]]机制的本质是对象之间的关联关系。   
    
    
## 面向委托的设计
[[Prototype]]代表的是一种不同于类的设计模式。       
    
    
### 类理论
假设我们需要在软件中建模一些类似的任务。    
    
类的设计方法：  
1. 定义一个通用父（基）类，可以将其命名为Task，在Task类中定义所有任务都有的行为。
2. 接着定义子类XYZ 和ABC ，它们都继承自Task 并且会添加一些特殊的行为来处理对应的任务。  
    
类设计模式鼓励你在继承时使用方法重写（和多态）。
```java
class Task { 
    id;
    // 构造函数Task()
    Task(ID) { id = ID; }
    outputTask() { output( id ); }
}

class XYZ inherits Task { 
    label;
    // 构造函数XYZ()
    XYZ(ID,Label) { super( ID ); label = Label; }
    outputTask() { super(); output( label ); } 
}

class ABC inherits Task { 
    // ...
}
```
构造完成后，通常只需要操作这些实例（而不是类），因为每个实例都有完成任务的所有行为。    
        
        
### 委托理论
委托行为的设计方法：    
1. 你会定义一个名为Task的对象（和许多JavaScript开发者告诉你的不同，它既不是类也不是函数），它会包含所有任务都可以使用（写作使用，读作委托）的具体行为。      
2. 接着，对于每个任务（“XYZ”、“ABC”）你都会定义一个对象来存储对应的数据和行为。你会把特定的任务对象都关联到Task 功能对象上，让它们在需要的时候可以进行委托。
```javascript
Task={
    setID:function(ID){this.id=ID},
    outputID:function(){console.log(this.id);}
};

//让XYZ委托Task
XYZ=Object.create(Task);

XYZ.prepareTask=function(ID,Label){
    this.setID(ID);
    this.label=Label;
};

XYZ.outputTaskDetails=function(){
    this.outputID();
    console.log(this.label);
};
```
*这种编码风格可以被称为“对象关联”。*    
    
    
对象关联风格的代码还有一些不同之处：    
1. 通常来说，在[[Prototype]] 委托中最好把状态保存在委托者（XYZ、ABC）而不是委托目标（Task ）上。
2.  在类设计模式中，我们故意让父类（Task）和子类（XYZ）中都有outputTask方法，这样就可以利用重写（多态）的优势。在委托行为中则恰好相反：我们会尽量避免在[[Prototype]]链的不同级别中使用相同的命名，否则就需要使用笨拙并且脆弱的语法来消除引用歧义（就是显示伪多态）。
3.  委托行为 意味着某些对象（XYZ）在找不到属性或者方法引用时会把这个请求委托给另一个对象（Task ）。 
    
    
#### 1.互相委托（禁止）
无法在两个或两个以上互相（双向）委托的对象之间创建循环委托。

#### 2.调试
详情见书本P168
    
    
    
### 比较思维模型
通过代码来比较两种设计模式（类和行为委托）具体的实现方法。    
下面是典型的（“原型”）面向类风格——也就是高程里的组合继承：
```javascript
function Foo(who){
    this.me=who;
}
Foo.prototype.identify=function(){
    return 'I am'+this.me;
};

function Bar(who){
    Foo.call(this,who);
}
Bar.prototype=Object.create(Foo.prototype);
Bar.prototype.speak=function(){
    alert('Helo,'+this.identify()+'.');
};

var b1=new Bar('b1');
var b2=new Bar('b2');

b1.speak();
b2.speak();
```
子类Bar 继承了父类Foo ，然后生成了b1和b2两个实例。b1委托了`Bar.prototype`，`Bar.prototype`委托了`Foo.prototype`。
    
        
下面是对象关联风格的代码： 
```javascript
Foo={
    init:function(who){
        this.me=who;
    },
    identify:function(){
        return 'I am'+this.me;
    }
};

Bar=Object.create(Foo);
Bar.speak=function(){
    alert('Helo,'+this.identify()+'.');
};

var b1=Object.create(Bar);
b1.init('b1');

var b2=Object.create(Bar);
b2.init('b2');

b1.speak();
b2.speak();
```
这段代码同样完成任务且简洁了许多，我们只是把对象关联起来，并不需要那些既复杂又令人困惑的模仿类的行为（构造函数、原型以及new ）。        
    
    
下面看看两段代码对应的思维模型：    
1. 类风格代码的思维模型强调实体以及实体间的体系：      
![](http://img.aisss.top/17-11-11/36434319.jpg)     
        
    
这张图不够清晰，且其中有些小BUG，误导人。       
下面看看简化版————只展示必要的对象和关系：    
![](http://img.aisss.top/17-11-11/96514462.jpg)     
上图中虚线可有可无，反正可以通过原型链找到Foo()函数。       
    
    
    
2. 再看看对象关联风格的代码思维模型：   
![](http://img.aisss.top/17-11-11/48076431.jpg)     
对象关联风格的代码更加简洁，只关注一件事：**对象之间的关联关系**。  
        

        
        
## 类与对象
现在看看在真实场景中如何应用“类”和“行为委托”两种设计模式。      
    
首先看Web中非常典型的一种前端场景：创建UI控件（按钮、下拉表单···）。    
    
    
### 控件“类”
在面向对象设计模式里，一般会有一个包含所有通用控件行为的父类（可能叫做widget）和继承父类的特殊控件子类（可能叫button）
```javascript
//父类
function Widget(width,height){
    this.width=width||50;
    this.height=height||50;
    this.$elem=null;
}

Widget.prototype.render=function($where){
    if(this.$elem){
        this.$elem.css({
            width:this.width+"px",
            height:this.height+"px"
        }).appendTo($where);
    }
};

//子类
function Button(width,height,label){
    Widget.call(this,width,height);        //显示伪多态
    this.label=label||"Default";
    
    this.$elem=$("<button>").text(this.label);
} 

//让Button“继承”Widget
Button.prototype=Object.create(Widget.prototype);

//重写render(..)
Button.prototype.render=function($where){
    //"super"调用
    Widget.prototype.render.call(this,$where);   //显示伪多态
    this.$elem.click(this.onClick.bind(this));
};

Button.prototype.onClick=function(evt){
    console.log("Button'"+this.label+"'clicked!");
};

$(document).ready(function(){
    var $body=$(document.body);
    var btn1=new Button(125,30,"Hello");
    var btn2=new Button(125,40,"World");
    
    btn1.render($body);
    btn2.render($body);
});
```
子类中重写的`render(..)`不会替换父类中基础的`render(..)`，只是添加一些按钮特有行为。    
    
代码中也出现了显式伪多态。呸！  
    
    
    
#### ES6的class语法糖
简单介绍如何使用class来实现相同功能：
```javascript
class Widget{
    constructor(width,height){
        this.width=width||50;
        this.height=height||50;
        this.$elem=null;
    }
    render($where){
        if(this.$elem){
            this.$elem.css({
                width:this.width+"px",
                height:this.height+"px"
            }).appendTo($where);
        }
    }
}

class Button extends Widget{
    constructor(width,height,label){
        super(width,height);
        this.label=label||"Default";
        this.$elem=$("<button>").text(this.label);
    }
    render($where){
        super.render($where);
        this.$elem.click(this.onClick.bind(this));
    }
    onClick(evt){
        console.log("Button'"+this.label+"'clicked!");
    }
}

$(document).ready(function(){
    var $body=$(document.body);
    var btn1=new Button(125,30,"Hello");
    var btn2=new Button(125,40,"World");
    
    btn1.render($body);
    btn2.render($body);
});
```
使用ES6中`class`后，显示伪多态子类丑陋的语法不见了，但实际上这里并没有真正的类，`class`仍然是通过[[prototype]]机制实现的。          
关于class详细的介绍在附录A。        
    
    
### 委托控件对象
下面使用对象关联风格委托来实现更简单的Widget/Button：
```javascript
var Widget={
    init:function(width,height){
        this.width=width||50;
        this.height=height||50;
        this.$elem=null;
    },
    insert:function($where){
        if(this.$elem){
            this.$elem.css({
                width:this.width+"px",
                height:this.height+"px"
            }).appendTo($where);
        }
    }
};

var Button=Object.create(Widget);

Button.setup=function(width,height,label){
    //委托调用
    this.init(width,height);
    this.label=label||"Default";
    this.$elem=$("<button>").text(this.label);
};
Button.build=function($where){
    //委托调用
    init.insert($where);
    this.$elem.click(this.onClick.bind(this));
};
Button.onClick=function(evt){
    console.log("Button'"+this.label+"'clicked!");
};

$(document).ready(function(){
    var $body=$(document.body);
    
    var btn1=Object.create(Button);
    btn1.setup(125,30,"Hello");
    
    var btn2=Object.create(Button);
    btn2.setup(150,40,"world");
    
    btn1.build($body);
    btn2.build($body);
});
```
委托设计模式，能够使用不相同且更具描述性的方法，还能避免使用丑陋的显式伪多态，代之使用委托调用。    
    
之前的一次调用（`var btn1=new Button(..)`）现在变成了两次（`var btn1=Object.create(Button);btn1.setup(..)`），代码变多了一点，但是这样能更好关注分离原则，创建和初始化并不需要合并为一个步骤。  
    
    
    
## 更简洁的设计
对象关联除了能让代码看起来更简洁并且更具扩展性外，还能通过行为委托模式简化代码结构。下面看一个例子，对象关联如何简化整体设计。      
    
这个场景中我们有两个控制器对象，一个用来操作网页中的登录表单，另一个用来与服务器进行验证（通信）。我们使用jQuery创建Ajax通信。      
        
在传统的类设计模式中，我们会把基础函数定义在名为Controller的类中，然后派生两个子类LoginController和AuthController，他们都继承自Controller并且重写了一些基础行为：
```javascript
//父类
function Controller(){
    this.errors=[];
}
Controller.prototype.showDialog=function(title,msg){
    //给用户显示标题和消息
};
Controller.prototype.success=function(msg){
    this.showDialog("Success",msg);
};
Controller.prototype.failure=function(err){
    this.errors.push(err);
    this.showDialog("Error",err);
};

//子类
function LoginController(){
    Controller.call(this);
}
//把子类关联到父类
LoginController.prototype=Object.create(Controller.prototype);
LoginController.prototype.getUser=function(){
    return document.getElementById("login_username").value;
};
LoginController.prototype.getPassword=function(){
    return document.getElementById("login_password").value;
};
LoginController.prototype.validateEntry=function(user,pw){
    user=user||this.getUser();
    pw=pw||this.getPassword();
    
    if(!(ueser&&pw)){
        return this.failure("Please enter a username & password!");
    }
    else if(pw.length<5){
        return this.failure("Password must be 5+ characters!");
    }
    
    //如果执行到这里说明通过验证
    return true;
};
//重写基础的failure()
LoginController.prototype.failure=function(err){
    //“super”调用
    Controller.prototype.failure.call(this,"Login invaild: "+err);
};


//子类
function AuthController(login){
    Controller.call(this);
    //合成
    this.login=login;
}
//把子类关联到父类
AuthController.prototype=Object.create(Controller.prototype);
AuthController.prototype.server=function(url,data){
    return $.ajax({
        url:url,
        data:data
    });
};
AuthController.prototype.checkAuth=function(){
    var user=this.login.getUser();
    var pw=this.login.getPassword();
    
    if(this.login.validateEntry(user,pw)){
        this.server("/check-auth",{
            user:user,
            pw:pw
        }).then(this.success.bind(this)).fail(this.failure.bind(this));
    }
};
//重写基础的success()
AuthController.prototype.success=function(){
    //"super"调用
    Controller.prototype.success.call(this,"Authentitcated");
};
//重写基础的failure()
AuthController.prototype.failure=function(err){
    //"super"调用
    Controller.prototype.failure.call(this,"Auth Failed: "+err);
};

var auth=new AuthController(
    //除了继承，我们还需要合成
    new LoginController()
);
auth.checkAuth();
```
所有控制器共享的基础行为是`success(..)`、`failure(..)`和`showDialog(..)`。子类LoginController和AuthController 通过重写`failure(..)` 和`success(..)` 来扩展默认基础类行为。      
    
另一个需要注意的是我们在继承的基础上进行了一些合成。AuthController需要使用LoginController ，因此我们实例化后者（`new LoginController()`）并用一个类成员属性`this.login`来引用它，这样AuthController 就可以调
用LoginController 的行为。
    
    
### 反类
我们真的需要一个controller父类、两个子类加上合成来对这个问题进行建模吗？          
该对象关联的行为委托设计模式登场了
```javascript
var LoginController={
    errors:[],
    getUser:function(){
        return document.getElementById("login_username").value;
    },
    getPassword:function(){
        return document.getElementById("login_password").value;
    },
    validateEntry:function(user,pw){
        user=user||this.getUser();
        pw=pw||this.getPassword;
        
        if(!(ueser&&pw)){
        return this.failure("Please enter a username & password!");
        }
        else if(pw.length<5){
        return this.failure("Password must be 5+ characters!");
        }
    
        //如果执行到这里说明通过验证
        return true;
    },
    showDialog:function(title,msg){
        //给用户显示标题和消息
    },
    failure:function(err){
        this.errors.push(err);
        this.showDialog("Error","Login invaild: "+err);
    }
};

//让AuthController委托LoginController
var AuthController=Object.create(LoginController);

AuthController.errors=[];
AuthController.checkAuth=function(){
    var user=this.login.getUser();
    var pw=this.login.getPassword();
    
    if(this.login.validateEntry(user,pw)){
        this.server("/check-auth",{
            user:user,
            pw:pw
        }).then(this.accepted.bind(this)).fail(this.rejected.bind(this));
    }
};
AuthController.server=function(url,data){
    return $.ajax({
        url:url,
        data:data
    });
};
AuthController.accepted=function(){
    this.showDialog("Success","Authenticated!");
};
AuthController.rejected=function(err){
    this.failure("Auth Failed: "+err);
};

AuthController.checkAuth();
```
在行为委托模式中，AuthController 和LoginController 只是对象，它们之间是**兄弟关系** ，并不是父类和子类的关系。代码中AuthController委托了LoginController，反向委托也完全没问题。        
    
这种模式的重点在于只需要两个实体（LoginController和AuthController），而之前的模式需要三个，省去了Controller基类来“共享”，也避免了面向类设计模式中的多态。      
    
总结：我们用一种（极其）简单的设计实现了同样的功能，这就是**对象关联风格代码**和**行为委托设计模式**的力量。    
    
    
    
    
## 更好的语法
**1. ES6中的class**               
class可以简洁定义类方法，能够不使用关键字function，如下：
```javascript
class Foo{
    methodName(){/*..*/}
}
```
**2. ES6中的简洁方法声明**          
ES6中任意对象的字面量都能使用简洁方法声明，如下：
```javascript
var LoginController = { 
    errors: [],
    getUser() { //妈妈再也不用担心代码里有function了！
        // ...
    },
    getPassword() {
        // ...
    }
    // ...
};
```
**3. ES6中更好的对象字面量语法**        
使用对象字面量+`Object.setPrototypeOf(..)`方法，抛弃`Object.create(..)`：
```javascript
// 使用更好的对象字面形式语法和简洁方法
var AuthController = { 
    errors: [],
    checkAuth() { 
        // ...
    },
    server(url,data) {
        // ...
    }
    // ...
};
// 现在把AuthController关联到LoginController
Object.setPrototypeOf( AuthController, LoginController );
```
        
        
### 反词法
```javascript
var Foo = {
    bar() { /*..*/ },
    baz: function baz() { /*..*/ }
};
```
去掉语法糖后：
```javascript
var Foo = {
    bar: function() { /*..*/ }, 
    baz: function baz() { /*..*/ }
};
```
由于函数对象本身没有名称标识符，所以bar() 的缩写形式（function().. ）实际上会变成一个 匿名函数表达式 数表达式并赋值给bar属性。相比之下，具名函数表达式具名函数表达式（function baz().. ）会额外给.baz 属性附加一个**词法名称标识符baz** （baz只在其函数内部可见）。      
        
简洁方法有一个非常小但是非常重要的缺点——自我引用更难。      
因为它不具备可以自我引用的词法标识符。
```javascript
var Foo = {
    bar: function(x) {
        if(x<10){
           return Foo.bar( x * 2 );   //注意这一行
        }
        return x; 
    },
    baz: function baz(x) { 
        if(x < 10){
           return baz( x * 2 );       //注意这一行
        }
        return x; 
    }
};
```
在本例中使用`Foo.bar(x*2)`就足够了，但是在许多情况下无法使用这种方法，比如多个对象通过代理共享函数、使用this 绑定，等等。这种情况下最好的办法就是使用*函数对象的name标识符*来进行真正的自我引用。  
        
        
## 内省
自省就是检查实例的类型。        
    
类实例的自省主要目的是通过创建方式来判断对象的结构和功能。  
        
`instanceof`语法会产生语义困惑而且非常不直观，不能直接判断两个对象是否关联。
```javascript
function Foo() { /* .. */ } 
Foo.prototype...

function Bar() { /* .. */ }
Bar.prototype = Object.create( Foo.prototype );

var b1 = new Bar( "b1" );
```
如果要使用instanceof和prototype语义来检查上面中实体的关系，需要这么做：
```javascript
// Foo和Bar互相关联
Bar.prototype instanceof Foo; // true 
Object.getPrototypeOf( Bar.prototype )
   === Foo.prototype; // true 
Foo.prototype.isPrototypeOf( Bar.prototype ); // true

// b1关联到Foo和Bar
b1 instanceof Foo; // true
b1 instanceof Bar; // true
Object.getPrototypeOf( b1 ) === Bar.prototype; // true 
Foo.prototype.isPrototypeOf( b1 ); // true
Bar.prototype.isPrototypeOf( b1 ); // true
```
    
对象关联风格代码的内省更加简洁。
```javascript
var Foo = { /* .. */ };
var Bar = Object.create( Foo );
Bar...
var b1 = Object.create( Bar );
```
使用对象关联，所有对象都是通过[[prototype]]委托互相关联的。内省方法更加简洁。
```javascript
// Foo和Bar互相关联
Foo.isPrototypeOf( Bar ); // true 
Object.getPrototypeOf( Bar ) === Foo; // true

// b1关联到Foo和Bar
Foo.isPrototypeOf( b1 ); // true
Bar.isPrototypeOf( b1 ); // true 
Object.getPrototypeOf( b1 ) === Bar; // true
```
    
    
## 小结
在软件架构中你可以选择是否选择是否使用类和继承设计模式。大多数开发者理所当然地认为类是唯一（合适）的代码组织方式，但是本章中我们看到了另一种更少见但是更强大的设计模式：**行为委托** 。         
        
行为委托认为对象之间是兄弟关系，互相委托，而不是父类和子类的关系。JavaScript的[[Prototype]] 机制本质上就是行为委托机制。也就是说，我们可以选择在JavaScript中努力实现类机制（参见第4和第5章），也可以拥抱更自然的[[Prototype]] 委托机制。        
        
当你只用对象来设计代码时，不仅可以让语法更加简洁，而且可以让代码结构更加清晰。      
        
对象关联（对象之前互相关联）是一种编码风格，它倡导的是直接创建和关联对象，不把它们抽象成类。对象关联可以用基于[[Prototype]] 的行为委托非常自然地实现。