---
title: 《JavaScript语言精粹》第四章-继承
date: 2017-10-16 17:44:19
categories:
- 读书笔记
- 《JavaScript语言精粹》
tags:
- JavaScript
- 系统学习
description: 对象的继承
---
> ···往往会把一件完整的东西化成无数的形象。就像凹凸镜一般，从正面望去，只见一片模糊。       
> ————威廉-莎士比亚



### 伪类(Pseudoclassical)
当一个函数对象被创建时，Function构造器产生的函数对象会运行类似这样的代码：
```javascript
this.prototype={constructor:this};
```
新函数对象自动被赋予一个`prototype`属性，它的值是一个包含`constructor`属性以及存放继承特征的对象。`constructor`属性没什么用，重要的是`prototype`对象。     


用`new`前缀去调用一个函数时，函数执行的方式会被修改。如果new运算符是一个方法而不是一个运算符，如下
```javascript
Function.method('new',function(){
    
    //创建一个新对象，它继承自构造器函数的原型对象
    var that=Object.create(this.prototype);
    
    //调用执行构造器函数，绑定-this-到新对象上，进而为新对象添加属性
    var other=this.apply(that,arguments);
    
    //other有返回值且是一个对象，则返回此对象；
    //若不是一个对象，则返回上面创建的新对象that
    return (typeof other==='object' && other)||that;
});
```
**注意**:调用构造函数时没有加上`new`前缀，那么`this`将不会被绑定到一个新对象上，而是被绑定到全局对象上。这样不仅没有扩充新对象，反而破坏了全局变量环境。


### 对象说明符
可以将一大串参数，构建成有说明规格的对象
```javascript
var myObject=marker(f,l,m,c,s);
改为
var myObject=marker({
    first:f,
    middle:m,
    last:l,
    state:s,
    city:c
});
```

### 原型(Prototypal)
纯粹的原型模式可以避免把一个应用拆解成一系列嵌套类的分类过程，转而专注于对象。


### 函数化(Functional)
继承模式的一个弱点是没法保护隐私，函数化模式可以解决这个问题。简单分四步：
1. 创建一个新对象
2. 有选择的定义私有实例变量和方法。函数中通过var定义的普通变量。
3. 给这个对象扩充方法。这些方法有特权去访问参数，以及第2步中通过var定义的变量。
4. 返回那个新对象。     

函数化构造器的伪代码模板：
```javascript
var constructor=function(spec,my){
    var that ,其他的私有实例变量；
    
    my=my||{};
    把共享的变量和函数添加到my中
    
    that=一个新对象
    添加给that的特权方法
    
    
    return that;
}
```
函数化构造器详细讲解：      
1. `spec`对象包含构造器需要构造一个新实例的所有信息。    
2. `my`对象是一个为继承链中的构造器提供秘密共享的容器。可以选择性使用。       
3. 接下来，声明该对象（即返回的`that`）私有的实例变量和方法。       
4. 接下来，给`my`对象添加共享的秘密成员。
```javascript
my.member=value;
```
5. 构造一个新对象并把它赋值给`that`。很多种方式创建一个新对象。
6. 接下来，扩充`that`，加入组成该对象接口的特权方法。先把函数定义为私有方法，然后再把它们分配给`that`:
```javascript
var methodical=function(){
    ...
};

that.methodical=methodical;
```
7. 返回`that`。

例子：
```javascript
var mamal=function(spec){
    var that={};
    
    that.get_name=function(){
        return spec.name;
    };
    that.says=function(){
        return spec.saying||'';
    };
    
    return that;
}

var myMamal=mamal({name:'Herb'});

//构造器cat调用mamal,让mamal去做对象创建的大部分工作
//Cat只需关注自身的差异
var cat=function(spec){
    spec.saying=spec.saying||"meow";
    var that=mamal(spec);
    that.purr=function(){};
    that.get_name=function(){
        return that.says()+' '+spec.name+' '+that.says();
    };
    return that;
}

var myCat=cat({name:'LI'});
```
函数化模式还提供一个处理父类方法的方法。构造一个superior方法，取得一个方法名并返回调用那个方法的函数，该函数会调用原来的方法。
```javascript
Object.method('superior',function(name){
    var that=this,
        method=that[name];
    return function(){
        return method.apply(that,arguments);
    };
});

var coolcat=function(spec){
    var that=cat(spec),
        super_get_name=that.superior('get_name');
    that.get_name=function(n){
        return 'like'+super_get_name()+'baby';
    };
    return that;
};

var  myCoolCat=coolcat({name:'bo'});
var name=myCoolCat.get_name();     // 'like meow Bix meow baby'
```

函数化模式有很大灵活性。比伪类（构造函数）模式不仅带来工作更少，还让我们得到更好的封装和信息隐蔽，以及访问父类方法的能力。


### 部件
可以从一套部件中把对象组装出来。       
我们可以构造一个给任何对象添加简单事件处理特性的函数。它给对象添加一个on方法、一个fire方法和 一个私有的事件注册表对象：
```javascript
var eventuality=function(that){
    var registry={};
    
    that.fire=function(event){
//在一个对象上触发一个事件。该事件可以是一个包含事件名称的字符串
//或者一个拥有包含事件名称的type属性的对象
//通过'on'方法注册的事件处理程序中匹配事件名称的函数将被调用

        var array,
            func,
            handler,
            i,
            type=typeof event === 'string' ? event:event.type;
//如果这个事件存在一组事件处理程序，那么遍历它们并按顺序依次执行。

        if(registry.hasOwnProperty(type)){
            array=registry[type];
            for(i=0;i<array.length;i++){
                handle=array[i];

//每个处理程序包含一个方法和一组可选的参数
//如果该方法是一个字符串形式的名字，那么寻找到该函数。
                func=handle.method;
                if(typeof func === 'string'){
                    func=this[func];
                }
                
//调用一个处理程序。如果该条目包含参数，那么传递他们过去。否则，传递该事件对象
                func.apply(this,handler.parameters || [event]);
            }
        }
        return this;
    };
    
    that.on=function(type,method,parameters){
//注册一个事件。构造一条处理程序的条目。将它插入到处理程序数组中，
//如果这种类型的事件还不存在，就构造一个。
        var handler={
            method:method,
            parameters:parameters
        };
        if(registry.hasOwnProperty(type)){
            registry[type].push(handler);
        }else{
            registry[type]=[handler];
        }
        return this;
    };
    return that;
};
```
