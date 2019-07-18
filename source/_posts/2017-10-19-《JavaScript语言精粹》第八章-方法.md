---
title: 《JavaScript语言精粹》第八章-方法
date: 2017-10-19 15:34:13
categories:
- 读书笔记
- 《JavaScript语言精粹》
tags:
- JavaScript
- 系统学习
description: 主要介绍数组和字符串方法
---
> 他虽疯，但却有他的一套理论。      
> ———威廉-莎士比亚

### Array
*array.concat(item..)*      
`concat`方法产生一个新数组，包含一份*array*数组的浅复制(shallow copy)并把一个或多个参数*item*附加在其后。如果*item*是数组，它的每个元素会被分别添加。
    
*array.join(separator)*     
`join`方法把一个*array*构造成一个字符串。它先把*array*中每一个元素构造成一个字符串，接着用一个*separator*分隔符把它们连接在一起。默认*separator*是逗号“，”。       
        
*array.pop()*和*array.push(item...)*   
`pop`和`push`方法使得数组像堆栈一样工作。   
*push(item...)*如果*item*是数组，则它会把参数数组作为单个元素整个添加到数组中。
```javascript
//pop可以这样实现
Array.method('pop',function(){
    return this.splice(this.length-1,1)[0];
});

//push可以这样实现
Array.method('push',function(){
    this.splice.apply(this,[this.length,0].concat(Array.prototype.slice.apply(arguments));
    return this.length;
});
```

*array.shift()*     
`shift`方法移除数组*array*中的第一个元素并返回该元素。如果这个数组为空，它会返回`undefined`。shift通常比pop慢得多。
```javascript
Array.method('shift',function(){
    return this.splice(0,1)[0];
});
```

*array.slice(start,end)*     
`slice`方法对array中的一段做浅复制。首先复制`array[start]`，一直复制到`array[end]`为止。“前包后不包”。end参数可选，默认值是该数组长度。    
如果两个参数中的任何一个是负数，`array.length`会和它们相加，试图让他们成为非负数。      
如果`start`>=`array.length`，得到的结果将是一个新的空数组。


*array.sort(comparefn)*     
`sort`方法对array中的内容进行排序，但不能正确的给一组数字排序，因为默认的比较函数会把要被排序的元素视为字符串。      
但可以使用自己的比较函数作为参数替代默认比较函数。下面是给任何包含简单值的数组排序：
```javascript
var m=['aa','bb','a',4,8,15,16,23,43];
m.sort(function(){
    if(a===b){
        return 0;
    }
    if(typeof a === typeof b){
        return a<b ? -1:1;
    }
    return typeof a < typeof b ? -1:1;
});
```
我们可以编写一个构造更智能的比较函数的函数，来满足一般情况。    
```javascript
//by函数接受一个成员名字符串作为参数
//并返回一个可以用来对包含该成员的对象数组进行排序的比较函数
var by=function(name){
    return function(o,p){
        var a,b;
        if(typeof o ==='object'&&typeof p ==='object'&& o &&p){
            a=o[name];
            b=p[name];
            if(a===b){
                return 0;
            }
            if(typeof a === typeof b){
                return  a < b?-1:1;
            }
            return typeof a < typeof b?-1:1;
        }else{
            throw{
                name:'Error',
                message:'Expected an object when sorting by'+name
            };
        }
    };  
};
```
`sort`方法是不稳定的，不建议链式调用`sort`方法。    

*array.splice(start,deleteCount,item...)*   
`splice`方法从array中移除一个或多个元素，并用新的item替换它们。它返回一个包含被移除元素的数组。      
参数`start`代表开始位置，`deleteCount`代表被移除的元素个数，可以为0，`item`会插入到被移除元素的位置。    
`splice`可在像这样实现：
```javascript
Array.method('splice',function(start,deleteCount){
    var max=Math.max,
        min=Mathh.min,
        delta,
        element,
        insertCount=max(arguments.length-2,0),
        k=0,
        len=this.length,
        new_len,
        result=[],
        shift_count;
        
    start=start||0;
    //start若为负数，则从数组末位开始计位（从1开始）
    if(start<0){
        start+=len;
    }
    
    //这一条代码兼顾两种特殊情况：
    //若start为正，且大于等于数组长度，则从数组末尾开始添加内容
    //若start为负，且大于等于数组长度，则从数组第一项开始操作
    start=max(min(start,len),0);
    
    //这一条代码兼顾3种特殊情况
    //若deleteCount被省略（不是一个数），则其相当于len-start
    //若deleteCount为正，且大于等于start之后的元素的总数，则其相当于len-start
    //若deleteCount为负，且大于等于start之后的元素的总数，则其等于0
    deleteCount=max(min(typeof deleteCount==='number' ? deleteCount : len,len-start),0);
});
    
    //添加的元素数和删除的元素数之差
    delta=insertCount-deletCount;
    new_len=len+delta;
    
    //给要返回的数组赋值
    while(k<deleteCount){
        element=this[start+k];
        if(element!=undefined){
            result[k]=element;
        }
        k+=1;
    }
    
    //start后面不用改变的元素的个数
    shift_count=len-start-deleteCount;
    //将start后面不用改变的元素赋值给改变后的新数组
    if(delta<0){
        k=start+insertCount;
        while(shift_count){  //从起始点开始逐个替换
            this[k]=this[k-delta];
            k+=1;
            shift_count-=1;
        }
        this.length=new_len;
    }else if(delta>0){  //在新数组长度下从后往前替换
        k=1;
        while(shift_count){
            this[new_len-k]=this[len-k];
            k+=1;
            shift_count-=1;
        }
        this.length=new_len;
    }
    
    //将添加的元素赋值给数组
    for(k=0;k<insertCount;k++){
        this[start+k]=arguments[k+2];
    }
    
    //返回被删除的元素组成的数组
    return result;
});
```


*array.unshift(item...)*        
`unshift`把元素添加到数组开始部分。返回array的新length。        
可以这样实现：
```javascript
Array.method('unshift',function(){
    this.splice.apply(this,[0,0].concat(Array.prototype.slice.apply(arguments)));
    return this.length;
});
```

### Number
*number.toExponential(fractionDigits)*      
`toExponential`方法把这个number转换成一个指数形式的字符串。可选参数控制小数点位数。参数值必须在0-20中。      
        
*number.toFixed(fractionDigits)*    
`toFixed`方法把这个number转换成十进制数形式的字符串。可选参数控制小数点位数。参数必须在0-20中。    
    
*number.toPrecision(precision)*    
`toPrecision`方法把这个number转换成十进制数形式的字符串。可选参数控制小数点位数。参数必须在0-21中。   


### RegExp
*regexp.exec(string)*       
`exec`方法如果成功的匹配regexp和字符串string，它会返回一个数组。数组下标为0的元素将包含regexp匹配的子字符串。下标为1的元素是分组1捕获的文本，以此类推。     
如果regexp带有`g`标识，会改变`regexp.lastIndex`的值。   
        
*regexp.test(string)*       
如果regexp匹配string，就返回true,否则返回false。不要对这个方式使用`g`标识。
    
    
### String
*string.charAt(pos)*    
可以这样实现：
```javascript
String.method('string',function(){
    return this.slice(pos,pos+1);
});
```

*string.match(regexp)*      
`match`方法让字符串和一个正则表达式匹配。它依据`g`标识来决定如何进行匹配。      
如果没有`g`标识，那么调用`string.match(regexp)`的结果与调用`regexp.exec(string)`相同。      
如果带有`g`标识，那么它生产一个包含所有匹配（无捕获分组）的数组。   
        
*string.replace(searchValue,replaceValue)*      
`replace`方法对string进行查找和替换操作，并返回一个新的字符串。`searchValue`可以是字符串或正则表达式，`replaceValue`可以是字符串或函数。    
如果`replaceValue`是字符，字符`$`有特别的含义。     
如果`replaceValue`是函数，那么每遇到一次匹配，函数就会调用一次，该函数返回的字符串会被用做替换文本。传递给这个函数第1个参数是整个被匹配的文本，第2个参数是分组1捕获的文本，以此来类推。
```javascript
String.method('entityify',function(){
    var character={
        '<':'&lt',
        '>':'&gt',
        '&':'&amp',
        '"':'&quot'
    };
    
    return function(){
        return this.replace(/[<>&'']/g,function(c){
            return character[c];
        });
    };
}());

console.log('<&>'.entityify());
```

*string.search(regexp)*     
与`indexOf`类似，但它只接受一个正则表达式对象作为参数。忽略`g`标识，没有position参数。

*string.slice()*
和`array.slice`很像，都是复制再截取构造一个新实例。两个参数若为负数，则与length相加，或者从末位往前数（从1计数）。前包后不包。     

*string.split(separator,limit)*     
`split`方法把string分割成片段来创建一个字符串数组。可选参数limit可以限制被分割的片段数量。separator参数可以是字符串或者正则表达式。此方法会忽略`g`标识。    
**注意**：来自分组捕获的文本会被包含在被分割后的数组中
```javascript
var text='last,first,middle';
var e=text.split(/\s*(,)\s*/);
//e是['last',',','first',',','middle']
```

