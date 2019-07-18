---
title: 《JavaScript语言精粹》第四章-函数
date: 2017-10-03 09:50:33
categories:
- 读书笔记
- 《JavaScript语言精粹》
tags:
- JavaScript
- 系统学习
description: 函数十分重要，是JS中的一级对象。
---
> 所有的过失在未犯之前，都已定下应处的惩罚。      
>——威廉-莎士比亚

- 函数包含一组语句，他们是JS的基础模块单元，用于代码复用、信息隐藏和组合调用。
- 函数用于指定对象行为。
- 编程，就是将一组需求分解成一组函数与数据结构的技能。


### 函数对象
- js中函数是对象，原型对象是`Function.prototype`
- 每个函数在创建时会附加两个隐藏的属性：
    - 函数的上下文（执行环境）
    - 实现函数行为的代码（创建一个函数时，会给该函数对象设置一个“调用”属性）
- 每个函数在创建时也随配一个`prototype`属性，指向原型对象。
- 与众不同之处在于可以被调用


### 函数字面量
- 函数字面量包括4部分
    1. 保留字
    2. 函数名
    3. 包围在圆括号中的一组参数
    4. 包围在花括号中的一组语句
- 函数包含一个连到外部上下文的连接，被称为闭包（closure）。

### 调用(Invocation)
- **调用一个函数会暂停当前函数的执行，传递控制权和参数给新函数。**
- 除了形参，每个函数还接受两个附加的参数：`this`和`arguments`。`this`的取值取决于调用的模式。
- 4中调用模式：
    1. 方法调用
    2. 函数调用
    3. 构造器调用
    4. apply call调用
- 调用运算符是跟在任何产生一个函数值（函数对象）的表达式之后的一对圆括号。
- 实际参数（`arguments`）和形式参数（parameters）的个数可以不匹配。


### 方法调用模式
- 当一个函数被保存为对象的一个属性时，我们他为一个方法。
- 当一个方法被调用时，this被绑定到该对象。
- this到对象的绑定发生在函数调用的时候。延迟绑定好处是函数可以对`this`高度复用。
- 通过this可取得它们所属对象的上下文的方法称为**公共方法**.
```javascript
var myObject={
    value:0,
    increment:function(inc){
        this.value+=typeof inc === "number" ? inc:1;
    }
};

myObject.increment();
console.log(myObject.value);  //1

myObject.increment(2);
console.log(myObject.value);  //3
```

### 函数调用模式
- **函数调用模式时，this被绑定到全局对象**
- 可以在内部函数之外设置`that`变量保存`this`值。
```javascript
myObject.double=function(){
    var that=this;
    
    var helper=function(){
        that.value=add(that.value,that.value);
    };
    
    helper();
};

myObject.double();
consle.log(myObject.value);
```

### 构造器调用
- 如果一个函数前面带上`new`调用，那么背地里将会创建一个连接到该函数的`prototype`成员的新对象，同时`this`会被绑定到那个新对象。
- `new`前缀会改变`return`语句的行为。
    - 返回值是一个对象，则返回这个对象
    - 返回值是不是对象，则返回`this`（该新对象）
- 一个函数，如果创建的目的就是希望结合`new`前缀来调用，那么它被称为**构造（器）函数**.
    - 必须大写字母开头格式
    - 不建议单纯使用构造函数。


### Apply调用模式
- JS是一门函数式的面向对象编程语言，所以函数可以拥有方法——`call`和`apply`.
- `apply`接受两个参数：1.要绑定给`this`的值，2.一个参数数组
```javascript
var array=[3,4]
var sum=add.apply(null,array);  //sum值为7

var statusObject={
    status:"ok"
}

var status=Quo.prototype.get_status.apply(statusObject);  //status值为"A-ok"
```

### 参数(Arguments)
- 函数可以通过`arguments`参数访问所有它被调用时传递给它的参数列表，包括那些没有被分配给函数声明时定义的形式参数的多余参数
- `arguments`不是一个真正的数组，是类数组（array-like）对象。


### 返回(Return)
- 当一个函数调用时，从第一个语句开始执行，并在遇到}结束，然后函数把控制权交还给调用该函数的程序。
- 当`return`被执行时，函数立即返回不再执行余下的语句。
- ==一个函数总会返回一个值。如果没有指定返回值，则返回undefined==。

### 异常(Exceptions)
- 异常是干扰程序的正常流程的不寻常的事故。
- throw中断函数执行。它在后台抛出一个`exception`对象，该对象包含一个用来识别异常类型的`name`属性和一个描述性`message`属性。
```javascript
var add=function(a,b){
    if(typeof a !=="number" ||typeof b !=="number"){
        throw{
            name:"TypeEror",
            message:"add needs numbers"
        };
    }
    return a+b;
}
```

- 该`exception`对象将被送到一个`try`语句的`catch`从句。如果在`try`代码块内抛出一个异常，控制权就会跳转到它的`catch`从句。
```javascript
var try_it=function(){
    try{
        add("li");
    }catch(e){
        console.log(e.name+":"+e.message);
    }
}
try_it();
```

### 扩充类型的功能(Augmenting Types)
- JS允许给语言的基本类型扩充功能。
- 给`Function.prototype`增加方法，来使得该方法对所有函数可用。那么可以用来给内置类型（JS里类型就是构造函数）增加方法，进而可以在内置类型的实例上调用增加的方法。（以下例子并不好，代码不够明朗）
```javascript
//先给Function.prototype增加通用方法
Function.prototype.method=function(name,func){
    this.prototype[name]=func;
    return this;
};

//在内置类型的构造函数上使用增加的方法
Number.method('integer',function(){
    return Math[this<0?'ceil':'floor'](this);
});

String.method('trim',function(){
    return this.replace(/^\s+/\s+$/g,'');
});

//在内置类型的实例上调用增加的方法
console.log((-10/3).integer());   //-3
console.log(''''+'  neat  '+'''');
```
- 内置类型的原型是公用结构，在增加方法前先判断是否存在该方法。
```javascript
Function.prototype.method=function(name,func){
    if(!this.prototype[name]){
        this.prototype[name]=func;
    }
    return this;
};
```


### 递归(Recursion)
- 递归函数是会直接或间接调用自身的一种函数。一般来说，一个递归函数调用自身去解决它的子问题。
- 递归函数可以非常高效地操作树形结构，比如DOM。
```javascript
//从某一节点开始，按照HTML源码顺序访问该树的节点，并在节点上调用函数
var walk_the_DOM=function walk(node,func){
    func(node);
    node=node.firstChild;
    while(node){
        walk(node,func);
        node=node.nextSibling;
    }
}

//实用例子
//定义getElementsByAttribute函数
var getElementsByAttribute=function(att,value){
    var results=[];
    
    walk_the_DOM(document.body,function(node){
        var actual=node.nodeType&&node.getAttribute(att);
        if(typeof actual === 'string' && (typeof actual === value || typeof value !== 'sring')){
            results.push(node);
        }
    });
    
    return results;
}
```

### 作用域(Scope)
- 作用域控制着变量与参数的可见性及生命周期。
```javascript
var foo=function(){
    var a=3,b=5;
    
    var bar=function(){
        var b=7,c=11;   //此时，a=3,b=7,c=11
        a+=b+c;         //此时，a=21,b=7,c=11
    }
                        //此时，a=3,b=5,c没有定义
    bar();              //此时，a=21,b=5,c没有定义
}
```
- JS有函数作用域。一个函数内部定义的变量，在该函数内部任何地方可见，但在函数外部则是不可见的。


### 闭包(Closure)
- 作用域**好处**是，内部函数可以访问定义它们的外部函数的参数和变量（除了`this`和`arguments`）。
- **更有趣**,内部函数拥有比它外部函数更长的生命周期。
- **闭包（closure）**，一个函数可以访问它被创建时所处的上下文环境，该函数称为闭包。
```javascript
//定义一个函数，它设置一个DOM节点为黄色，然后把它渐变为白色

var fade=function(node){
    var level=1;
    var step=function(){
        var hex=level.toString(16);                 //二进制数字转为16进制字符
        node.style.backgroundColor='#FFFF'+hex+hex;
        if(level<15){
            level+=1;
            setTimeout(step,100);
        }
    }
    
    setTimeout(step,100);
};

fade(document.body);
```
- fade函数已经返回了，但只要fade的内部函数需要，它的变量就会持续保留。



### 模块(Module)
- **模块**是一个提供接口却隐藏状态与实现的函数或对象。可由函数和闭包构造。
- 模块模式利用了函数作用域和闭包来创建被绑定对象与私有成员的关联。构造模块几乎完全可以摒弃全局变量的使用。
```javascript
Function.prototype.method=function(name,func){
    this.prototype[name]=func;
    return this;
};

String.method('deentityify',function(){  //这个函数结尾有(),立即调用，下面返回的函数才是deentityify方法
    var entity={
        quot: '"',
        lt: '<',
        gt: '>'
    };

    return function(){  
        //一个括号代表一个捕获组
        return this.replace(/&([^&;]+);/g,function(a,b){
            var r=entity[b];
            return typeof r==='string'?r:a;
        });
    };
}());

console.log('&lt;&quot;&gt;'.deentityify())
```
- **一般形式：**
    1.  一个定义了私有变量和函数的函数；
    2.  利用闭包创建可以访问私有变量和函数的特权函数；
    3.  最后返回这个特权函数，或者把它们保存到一个可访问到的地方。
- 模块模式也可用来产生安全对象。例子如下

```javascript
var serial_maker=function(){

// 唯一字符串由两部分组成：前缀+序列号
// 返回一个用来产生唯一字符串的对象
// 该对象包含一个设置前缀的方法，一个设置序列号的方法
// 和一个产生唯一字符串的gensym方法

    var prefix='';
    var seq=0;
    return {
        set_prefix:function(p){
            prefix=String(p);    //和字符串字面量一样属于基本字符串，不是字符串对象。
        }，
        set_ser:function(s){
            seq=s;
        },
        gensym:function(){
            var result=prefix+seq;
            seq+=1;
            return result;
        }
    };
};

var seqer=serial_maker();
seqer.set_prefix('Q');
seqer.seq(1000);
var unique=seqer.gensym(); 
```

### 级联(Cascade)
- 级联就是链式调用。
- 启用方法：`return this`——返回`this`。


### 柯里化(Curry)
- 函数也是值，所以**柯里化**允许我们把函数与传递给它的参数相结合，产生一个新的函数。
```javascript
Function.method('curry',function(){
    var args=Array.prototype.slice.apply(arguments);
    var that=this;
    return function(){
        var innerArgs=Array.prototype.slice.apply(arguments);
        var finalArgs=args.concat(innerArgs);
        return that.apply(null,finalArgs);
    }
});
```


### 记忆(Memoization)
- 记忆是一种优化。函数可以将先前操作的结果记录在某个对象里，从而避免无谓的重复运算。
- 对象和数组实现这种优化很方便。
```javascript

var fibonacci=function(){
    var memo=[0,1];
    var fib=function(n){
        var result=memo[n];
        if(typeof result !== 'number'){  //检查结果是否已经存在
            result=fib(n-1)+fib(n-2);
            memo[n]=result;
        }
        return result;
    };
    return fib;
}();

console.log(fibonacci(10));
```
- 构造带记忆功能的函数
    1. 取得初始memo数组和formula函数
    2. 返回一个管理memo存储和在需要时调用formula函数的recur函数
    3. 把这个recur函数和它参数传递给formula函数

```javascript
var memoizer=function(memo,formula){
    var recur=function(n){
        var result=memo[n];
        if(typeof result !== 'number'){
            result=formula(recur,n);
            memo[n]=result;
        }
        return result;
    };
    return recur;
};

//memoizer另一种写法，将其中的result省略
var memoizer=function(memo,formula){
    var recur=function(n){
        //var result=memo[n];
        if(typeof memo[n] !== 'number'){
            memo[n]=formula(recur,n);
            //memo[n]=result;
        }
        return memo[n];
    };
    return recur;
};

//定义fibonacci函数
var fibonacci=memoizer([0,1],function(recur,n){
    return recur(n-1)+recur(n-2);
});

//定义阶乘函数
var factorial=memoizer([1,1],function(recur,n){
    return n*recur(n-1);
});
```