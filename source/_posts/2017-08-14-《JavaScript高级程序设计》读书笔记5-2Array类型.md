---
title: 《JavaScript高级程序设计》读书笔记5.2Array类型
date: 2017-08-14 20:59:08
categories:
- 读书笔记
- 《JavaScript高级程序设计》
tags:
- JavaScript
- 系统学习
description: 与其他语言不同的是，ECMAScript数组的每一项可以保存任何类型的数据。
---
**与其他语言不同的是，ECMAScript数组的每一项可以保存任何类型的数据。**

- ##### 两种创建数组方式：`Array`构造函数、数组字面量表示法
```javascript
var colors=new Array(3);            //创建一个包含3项的数组
var names=new Array("libowen");     //创建一个包含1项，即字符串"libowen"的数组
var colors=["red","blue","green"];  //创建一个包含3个字符串的数组
var names=[];                       //创建一个空数组
```
- ##### 读取和设置数组，使用[ 索引值 ]
- ##### 数组`length`属性的特点（**动态**）：
    - 设置`length`属性，可以从数组的末尾移除项或向数组中添加新项.
    ```javascript
    var colors=["red","blue","green"];
    colors.length=2;    //移除最后一项
    alert(colors[2]);   //undefined

    var colors=["red","blue","green"];
    colors[99]="black";   //在位置99添加一种颜色
    alert(colors.length); //100
    ```
    - 实际上，不使用缓存数组长度的方式比缓存版本要慢很多。
    - **数组长度只能通过length属性修改**，`delete`不会改变长度。
    
    
##### 1.检测数组
`Array.isArray()`方法

##### 2.转换方法
- 可以显示调用`toLocaleString()`、`toString()`、`valueOf()`方法转换，都是返回由数组每个值的字符串形式拼接而成的一个逗号分隔的字符串。    
- 后台则是调用`toString()`方法     
- `join()`方法改变分隔符，如join("||")

##### 3.栈方法(LIFO)
- 栈中项的插入（叫做**推入**）和移除（叫做**弹出**），只发生在栈的顶部。
- `push()`方法：接受任意数量参数，逐个添加到数组末尾，并返回修改后数组的长度。**改变原数组**。
- `pop()`方法：从数组末尾移除最后一项，减少数组的`length`值，然后返回移除的项。**改变原数组**。
```javascript
var colors=new Array();
var count=colors.push("red","green"); //推入两项
alert(count);            //2
count=colors.push("black"); //推入另一项
alert(count);            //3
var item=colors.pop()；
alert(item);   //"black"        //取得最后一项
alert(colors.length);   //2
```

##### 4.队列方法(FIFO)
- 队列在列表的末端添加项，从列表的前端移除项。
- `shift()`移除数组中第一个项并返回该项，同时将数组长度减1。**改变原数组**。
```javascript
var colors=new Array();
var count=colors.push("red","green"); //推入两项
alert(count);            //2
count=colors.push("black"); //推入另一项
alert(count);            //3
var item=colors.shift();
alert(item);     //"red"   //取得第一项
alert(item.length);   //2
```
- `unshift()`可以在数组前添加任意个项并返回新数组的长度。**改变原数组**。与`pop()`联用。

##### 5.重排序方法
- `reverse()`:反转数组项的顺序。
- `sort()`:默认情况下，按升序排列。但是通过比较字符串的方式来比较，容易出现问题。可以接受比较参数。
```javascript
比较函数：
function compare(value1,value2){
    if(value1<value2){
        return -1;
    }else if(value1>value2){
        return 1;
    }else{
        return 0;
    }
}

var values=[0,1,5,10,15];
values.sort(compare);
alert(values);     //0,1,5,10,15
```

##### 6.操作方法
- `concat()`：先创建当前数组一个副本，然后将接受到的参数添加到这个副本的末尾，最后返回新构建的数组。**不会改变原数组**。
```javascript
var colors=["red","green","blue"];
var colors2=colors.concat("yellow",["blacke","brown"]);
alert(colors);      //red,green,blue
alert(colors2);     //red,green,blue,yellow,black,brown
```
- `slice()`：截取数组。一个参数，截取参数指定位置到数组末尾所有项。两个参数，参数指定位置的第一项、参数末项前一个位置。**不会改变原数组**。
```javascript
var colors=["red","green","blue","yellow","purple"];
var colors2=colors.slice(1);
var colors3=colors.slice(1,4);
alert(colors2);    //green,blue,yellow,purple
alert(colors3);    //green,blue,yellow
```
- `splice()`：向数组的中部插入项。始终返回一个数组，该数组包含从原始数组删除的项，**改变原数组**。
    - 删除：2个参数。删除位置和删除项数。
    - 插入：3个参数。起始位置、0、插入的项
    - 替换：3个参数。起始位置、删除的项数、插入的项
```javascript
var colors=["red","green","blue"];
var removed=colors.splice(0,1);  //删除第一项
alert(colors);      //green,blue
alert(removed);     //red  返回的数组

removed=colors.splice(1,0,"yellow","orange");
alert(colors);      //green,yellow,orange,blue
alert(removed);     //返回一个空数组

removed=colors.splice(1,1,"red","purple");
alert(colors);      //green,red,purple,orange,blue
alert(removed);     //yellow
```


#### 以下方法支持浏览器最低版本：IE9+、FireFox2+、 Safari3+、Opera9.5+、Chrome 

##### 7.位置方法
`indexof()`和`lastIndexOf()`


##### 8.迭代方法( ECMAScript5)
5种方法。每种方法接受2个参数：每一项上运行的函数和运行该函数的作用域对象——影响`this`值（可选）。    
函数接受3个参数：数组项的值、该项在数组中的位置和数组对象本身。    
**不会改变原数组中的值。**
- `every()`:对数组中的每一项运行给定函数，如果该函数对每一项都返回`true`，则返回`true`。
- `some()`：对数组中的每一项运行给定函数，如果该函数对任一项返回`true`，则返回`true`。
- `filter()`：对数组中的每一项运行给定函数，返回该函数返回`true`的项组成的数组。
- `map()`：对数组中的每一项运行给定函数，返回每次函数调用的结果组成的数组。
- `forEach()`：对数组中的每一项运行给定函数，没有返回值。
```javascript
var numbers=[1,2,3,4,5,4,3,2,1];
var filternumbers=numbers.filter(function(item,index,array){
    return (item>2);
};
alert(filternumbers);    //[3,4,5,4,3]
```
##### 9.归并方法
`reduce()`和`reduceRight()`