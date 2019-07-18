---
title: 《你不知道的JavaScript-上》对象
date: 2017-11-08 15:58:10
categories:
- 读书笔记
- 《你不知道的JavaScript-上》
tags:
- JavaScript
- 系统学习
description: 万物皆对象是错误的说法
---
## 语法
对象通过两种形式定义：字面量、构造形式

## 类型
JS中有6中主要类型：
- undefined
- null
- string
- number
- boolean
- object    
    
前5种是简单基本类型，并不是对象。`null`有时被当作对象类型，这其实是语言本身的一个bug。      
    
有一种常见的错误说法是“JS万物皆是对象”，这显然是错误的。        
实际上，JS中有许多特殊的**对象子类型**，我们可以称之为**复杂的基本类型**。      
函数是对象的一个子类型，它本质上和普通对象一样，只是可以调用。（技术角度来说是“可调用的对象”）      
数组也是对象的一个子类型，具备一些额外的行为。      
        
    
### 内置对象
JS中还有一些对象子类型，通常被称为内置对象。
- Date
- RegExp
- Error
- Array
- Function
- Object
- String
- Number
- Boolean           

他们实际上是只是一些内置函数，这些内置函数可以当作构造函数来使用，从而可以构造一个对应子类型的新对象。      
        
对于字符串字面量、数值字面量、布尔字面量，引擎会自动把字面量转换成对应的对象。      
    
`null`和`undefined`没有对应的构造形式，它们只有文字形式。相反，Date只有构造，没有文字形式。          
    
对于`Object`、`Array`、`Function`、`RegExp`来说，无论使用文字形式还是构造形式，他们都是对象，不是字面量。


## 内容
对象的内容，是由一些存储在特定命名位置的（任意类型的）值组成的，也称之为属性。      
    
需要强调一点，在引擎内部，对象的值的存储方式是多种多样的，一般并不会存在对象容器内部，存储在对象容器内部的是这些属性的名称，他们像指针一样，指向这些值真正存储的位置。      
    
对象中，属性名永远是字符串。如果满足标识符命名规范，可以不带引号“”。        
        
        
### 1.可计算属性名    
ES6增加了可计算属性名，可以在文字形式中使用[]包裹一个表达式来当作属性名：
```javascript
var prefix="foo";

var  myObject={
    [prefix+"bar"]:"hello",
    [prefix+"baz"]:"world"
};

myObject[foobar];   //hello
myObject[foobaz];   //world
```

### 2.属性与方法  
把对象内部引用的函数称为“方法”，这样的叫法不妥。    
    
最保险的说法可能是，“函数”和“方法”在JS中山可以互换的。


### 3.数组            
数组期望的是数值下标，也就是说值存储的位置（通常被称为索引）是非负整数。    
数组也是对象，所以虽然每个下标都是整数，但仍然可以给数组添加属性。
```javascript
var myArray=["foo",42,"bar"];
myArray.baz="baz";
myArray.length; //3
myArray.baz;   //"baz"
```
虽然添加了命名属性（无论通过.语法还是[]语法），但数组的length值并未发生变化。   
    
    
### 4.复制对象     
深复制会由于循环引用导致死循环。    
    
对于JSON安全的对象来说，有一种巧妙的复制方法：
```javascript
var newObj=JSON.parse(JSON.stringify(someObj));
```
浅复制，ES6定义了`Object.assign(..)`方法来实现浅复制。  
`Object.assign(..)`方法第一个参数是目标对象，之后可以跟一个或多个源对象。它会遍历一个或多个源对象的所有可枚举的自由键并把它们复制到目标对象，最后返回目标对象。
```javascript
var newObj=Object.assign({},myObj);

newObj.a;   //2
newObj.b===anotherObject;  //true
newObj.c===anotherArray;   //true
newObj.d===anotherFunction;  //true
```
    

### 5.属性描述符     
从ES5开始，所有的属性都具备了属性描述符。
```javascript
var myObject={a:2};

Object.getOwnPropertyDescriptor(myObject,"a");
//{
//    value:2,
//    writable:true,
//    enumerable:true,
//    configurable:true
//}
```
这个普通对象属性对应的属性描述符（也被称为“数据描述符”，只保存一个数据值），包含4个特性：value、writable、enumerable、configurable。    
    
创建普通属性时属性描述符会使用默认值，我们也可以使用`Object.defineProperty(..)`来添加一个新属性或者修改一个已有属性（如果它是configurable）并对特性进行设置。  
        
##### 5.1 writable--可写         
`writable`决定是否可以修改属性的值。    
    
##### 5.2 configurable--可配置           
只有属性是可配置的，就可以使用`Object.defineProperty(..)`方法来修改属性描述符。

```javascript
var myObject={a:2};

myObject.a = 3;
myObject.a; // 3

Object.defineProperty(myObject,"a",{
    value:4,
    writable: true,
    enumerable:true,
    configurable:false  //不可配置！
});

myObject.a; // 4 
myObject.a = 5; 
myObject.a; // 5

Object.defineProperty( myObject, "a", {
    value: 6,
    writable: true, 
    configurable: true, 
    enumerable: true
} ); // TypeError
```

*注意：* 把`configurable`修改成`false`是单向操作，不可撤销！    
    
一个小小的例外:即便是`configurable:false`，我们还是可以把`writable`的状态由`true`改为`false`，但是无法由`false`改为`true`。     
        
除了无法用`Object.definedProperty(..)`修改属性描述符，`configurable:false`还会禁止删除这个属性。
```javascript
var myObject = { 
    a:2
};

myObject.a; // 2
delete myObject.a; 
myObject.a; // undefined

Object.defineProperty( myObject, "a", {
    value: 2,
    writable: true, 
    configurable: false, 
    enumerable: true
} );

myObject.a; // 2 
delete myObject.a;
myObject.a; // 2
```
    
##### 5.3 enumerable--可枚举            
`enumerable`控制属性是否会出现在对象的属性枚举中，例如`for..in`。   
默认`enumerable:true`。
    
    
### 6.不变性  
ES5有很多方法使属性或者对象不可改变，但所有方法创建的都是浅不变性，也就是说，它们只会影响目标对象和它的直接属性。日光目标对象引用了其他对象（数组、对象、函数等），其他对象内部不受影响，仍然可变。  
        
        
##### 6.1 对象常量           
结合`writable:false`和`configurable:false`，可以创建一个真正的常量属性（不可修改、不能重定义、不可删除）
```javascript
var myObj={};

Object.defineProperty(myObject,"FAVORITE_NUMBER",{
    value:42,
    writable:false,
    configurable:false
});
```
    
##### 6.2 禁止扩展            
使用`Object.preventExtensions(..)`禁止一个对象添加新属性并且保留已有的属性。    
    
##### 6.3 密封           
`Object.seal(..)`会创建一个“密封”的对象，这个方法实际上会在一个现有对象上调用`Object.preventExtensions(..)`并把所有属性标记为`configurable:false`。     
    
密封后不能添加新属性，不能重新配置，不能删除属性，只能修改已有属性。    
    
##### 6.4 冻结           
`Object.freeze(..)`会创建一个冻结对象，这个方法实际上会在一个现有对象上调用`Object.seal(..)`并把所有"数据访问"属性标记为`writable:false`。     
    
冻结是最高级别的不可变性，禁止对于对象本身及其任意直接属性的修改。（这个对象引用的其他对象是不受影响的）。


### 7.[[Get]]
```javascript
var myObject={
    a:2
};

myObject.a;  //2
```
在语言规范中，myObject.a在myObject上实际上是实现了`[[Get]]`操作（有点像函数调用：`[[Get]]()`）。    
对象默认的内置`[[Get]]`操作首先在对象中查找是否有名称相同的属性，如果有就返回这个值；如果没有找到名称相同的属性，则会遍历原型链。无论如何都没找到名称相同属性，那么会返回`undefined`。     
*注意：* 这种方法和访问变量时是不一样的。找不到变量会抛出一个ReferenceError异常，找不到对象属性会返回undefined。   
        
    
### 8.[[Put]]  
`[[Put]]`被触发时，实际的行为取决于许多因素，包括对象中是否已经存在这个属性。   

如果已经存在这个属性，`[[Put]]`算法大致会检查下面内容：     
1. 属性是否是访问描述符？如果是并且存在`setter`就调用`setter`。
2. 属性的数据描述符中`writable`是否是`false`？如果是，在非严格模式下静默失败，在严格模式下抛出TyoeError异常。
3. 如果都不是，将该值设置为属性的值。   
    
    
如果对象不存在这个属性，`[[Put]]`操作会更复杂。在第五章详细介绍。   
    
    
###  9.Getter和Setter   
对象默认的`[[Put]]`和`[[Get]]`操作分别可以控制属性值的设置和获取。  
    
ES5中可以使用getter和setter部分改写默认操作，但只能应用在单个属性上，无法应用到整个对象。   
    
当给一个属性定义setter、getter或者两者都有时，这个属性描述符会被定义为“访问描述符”。对于访问描述符，JS会忽略它们的value和writable特性,取而代之是关心get、set、configurable、enumerable特性。
```javascript
var myObject={
    //给a定义一个getter
    get a(){
        return 2;
    }
};

Object.defineProperty(myObject,"b",{
    //访问描述符
    //给b设置一个getter
    get:function(){
        return this.a*2;
    },
    
    //确保b会出现在对象的属性列表中
    enumerable:true
});

myObject.a;   //2
myObject.b;   //4
```
`get a(){..}`和`defineProperty(..)`，都会在对象中创建一个不包含值的属性，对于这个属性的访问会自动调用一个隐藏函数，它的返回值会被当作属性访问的返回值。    
    
通常来说，`getter`和`setter`是成对出现的。`setter`会覆盖单个属性默认的`[[Put]]`（也被称为赋值）操作。
```javascript
var myObject={
    //给a定义一个getter
    get a(){
        return this._a_;
    },
    
    //给a定义一个setter
    set a(val){
        this._a_=val*2;
    }
};

myObject.a=2;
myObject.a;   //4
```
    
        
### 10.存在性       
问题：`myObject.a`属性访问返回值是`undefined`时，该如何区分属性值是`undefined`还是属性不存在的情况？    
    
使用`in`操作符或者`hasOwnProperty`方法：        
`in`操作符会检查属性是否在对象实例和原型链中；  
`hasOwnProperty`方法只会检查对象实例。  
    
所有普通对象都可以通过对于`Object.prototype`的委托来访问`hasOwnProperty(..)`，但是有的对象可能没有连接到`Object.prototype`(例如`Object.create(nulll)`)。    
这是可用`Object.prototype.hasOwnProperty.call(obj,pro)`。       
    
**注意：** in操作符不是检查某个值，而是检查某个属性名是否存在。对于数组来说很重要，`4 in [2,4,6]`结果不是true，因为这数组包含属性名是0，1，2。   
    
    
##### 10.1 枚举     
设置属性描述符`enumerable:false`后，属性可以访问，可以运用`in`操作符，但不能出现在`for..in`循环中。   
    
`propertyIsEnumerable(..)`会检查给定的属性名是否直接存在于对象中（而不是原型链上）并且满足`enumerable:true`。   
    
`Object.key(..)`会返回一个数组，包含所有可枚举属性，只查找对象直接包含的属性。  
    
`Object.getOwnPropertyNames(..)`会返回一个数组，包含所有属性，无论他们是否枚举，只查找对象直接包含的属性。  
    
    
## 遍历
如何直接遍历值而不是数组下标？      
使用`for..of`循环（如果对象定义了迭代器的话也可以遍历对象）。   
```javascript
var myArray=[1,2,3];

for(var v of myArray){
    console.log(v);
}
//1
//2
//3
```
工作原理：`for..of`首先会向被访问对象请求一个迭代器对象，然后通过调用迭代器对象的`next()`方法来遍历所有返回值。    
    
数组内置的`@@iteraror`，因此`for..of`可以直接应用在数组上。我们使用内置的`@@iterator`来手动遍历数组，看它如何工作：
```javascript
var myArray=[1,2,3];
var it=myArray[Symbol.iterator]();

it.next();  //{value:1,done:false}
it.next();  //{value:2,done:false}
it.next();  //{value:3,done:false}
it.next();  //{done:true}
```
引用类似iterator的特殊属性时要用ES6中的符号Symbol。`@@iteraror`并不是一个迭代器对象，而是一个返回迭代器对象的函数。    
        
普通对象没有内置的`@@iteraror`，所以无法知道完成`for..of`遍历。     
但是，可以给任何想遍历的对象定义`@@iteraror`：  
```javascript
var myObject={
    a:2,
    b:3
};

Object.definedProperty{myObject,Symbol.iterator,{  //把符号当作可计算属性名
    enumerable:false,
    writable:false,
    configurable:true,
    value:function(){
        var o=this;
        var idx=0;
        var ks=Object.keys(o);
        return {
            next:function(){
                return {
                    value:o[ks[idx++]],   
                    done:(idx>ks.length)
                };
            }
        };
    }
}};

//也可以直接在定义对象时进行声明
// var myObject={
//    a:2,
//    b:3,
//    [Symbol.iterator]: function(){}
//}

var it=myArray[Symbol.iterator]();
it.next();  //{value:1,done:false}
it.next();  //{value:2,done:false}
it.next();  //{value:3,done:false}
it.next();  //{value:undefined,done:true}

for(var v of myArray){
    console.log(v);
}
//2
//3
```
只要迭代器的`next()`调用会返回`{value:..}`和`{done:true}`，ES6中的`for..of`就能遍历它。
    
    
## 小结
JS中的对象有字面形式和构造形式。字面形式更常用。    
    
“JS中万物都是对象”是错误的观点。对象是6个（ES6中是7个）基础类型之一。对象包括function在内的子类型，不同子类型具有不同的行为，比如内部标签[object Array]表示这是对象的子类型数组。      
    
对象是键/值对的集合。可以通过.和[]来获取属性值。对象默认的`[[Put]]`和`[[Get]]`操作分别可以控制属性值的设置和获取。      
    
属性的特性可以通过属性描述符来控制。此外，可以使用`Object.preventExtensions(..)`、`Object.seal(..)`、`Object.freeze(..)`来设置对象（及其属性）的不可变级别。    
    
属性不一定包含值——它们可能是具备getter/setter的“访问描述符”。此外，属性可以是可枚举或者不可枚举，这决定是否会出现在`for..in`循环中。   
    
可以使用ES6的`for..of`语法来遍历数据结构中的值，`for..of`会寻找内置或者自定义的`@@iterator`对象并调用它的`next()`方法来遍历数据值。