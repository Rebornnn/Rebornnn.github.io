---
title: 《JavaScript语言精粹》第五章-数组
date: 2017-10-17 11:42:39
categories:
- 读书笔记
- 《JavaScript语言精粹》
tags:
- JavaScript
- 系统学习
description: 数组也算对象，有对象的部分属性。
---
> 你披着羊皮的狼，我要把你赶走。     
> ————威廉-莎士比亚

JavaScript提供了一种拥有一些类数组(array-like)特性的对象。它把数组的下标转变成字符串，用其作为属性。它的属性的检索和更新的方式与对象一模一样，只不过多一个可以用整数作为属性名的特性。数组有自己的字面量格式。

### 数组字面量
一个数组字面量是在一对方括号中包围0个或多个逗号分隔的值的表达式。   
JavaScript允许数组包含任意混合型的值：
```javascript
var misc=[
    'string',98.6,true,{object:true},['array','nest']
];
```

### 长度
每一个数组都有一个`length`属性，且没有上界。    
用大>=当前`length`的数字作为下标来存储一个元素，那么`length`会被增大以容纳新元素，不会发生越界错误。    
把`length`设小将导致所有下标>=`length`的属性被删除。

`length`属性的值是这个数组的最大整数属性名加上1。它不一定等于数组里的属性的个数。


### 删除
`delete`运算符会在数组中留下一个空洞。这是因为排在被删除元素之后的元素保留着它们最初的属性。


### 容易混淆的地方
什么时候用数组或对象：当属性名是小而连续的整数时，用数组，否则，使用对象。


### 方法
通过给Array.prototype扩充一个函数，每个数组都继承了这个方法。   
数组其实是对象，也可以直接给一个单独的数组添加方法。


### 指定初始值
JavaScript的数组通常不会预置值。我们可以修正它：
```javascript
Array.dim=function(dimension,initial){
    var a=[],i;
    for(i=0;i<dimension;i++){
        a[i]=inital;
    }
    return a;
};

var myArray=Array.dim(10,0);
```
构造矩阵
```javascript
Array.matrix=function(m,n,initial){
    var a,i,j,mat=[];
    for(i=0;i<m;i++){
        a=[];
        for(j=0;j<n;j++){
            a[j]=initial;
        }
        mat[i]=a;
    }
    return mat;
};

//构造一个用0填充的4X4矩阵
var myMatrix=Array.matrix(4,4,0);

//构造一个单位矩阵的方法
Array.identity=function(n){
    var i,mat=Array.matrix(n,n,0);
    for(i=0;i<n;i++){
        mat[i][i]=1;
    }
    return mat;
};

myMatrix=Array.identity(4);
```