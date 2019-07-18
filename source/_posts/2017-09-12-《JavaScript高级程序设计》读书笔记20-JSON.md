---
title: 《JavaScript高级程序设计》读书笔记20-JSON
date: 2017-09-12 09:26:05
categories:
- 读书笔记
- 《JavaScript高级程序设计》
tags:
- JavaScript
- 系统学习
description: 一种轻量的数据格式
---

## 语法
JSON语法可以表示以下三种类型的值：
1. 简单值
2. 对象
3. 数组


### 简单值
一个注意点：JSON字符串必须使用双引号。

### 对象
- 没有声明变量
- 没有末尾分号
- 属性值，属性名必须加双引号


### 数组
- 没有声明变量
- 没有末尾分号
- 字符串必须加双引号

## 解析与序列化
**JSON解析为JS对象后提取数据比XML方便快捷。**


###  JSON对象
ECMAScript5定义了全局JSON对象，适用IE8+。    
JSON对象有2个方法：
- `stringify()`，把JS对象序列化为JSON字符串。
- `parse()`，把JSON字符串解析为原生JS值。

### 序列化选项
`JSON.stringify()`还可以接收两个参数：
- 第一个参数是过滤器，可以是数组也可以是函数
- 第二个参数是选项，表示是否在JSON字符串中保留缩进。大于等于4则会换行，最大为10。
- `toJSON()`方法，返回其自身的JSON数据格式。
- `JSON.stringify()`序列化对象的顺序：
    1. 如果存在`toJSON()`方法而且能通过它取得有效值，则调用该方法。否则，返回对象本身。
    2. 如果提供第二参数，应用这个函数过滤器。传入函数过滤器的值是第1步返回的值。
    3. 对第2步返回的每个值进行相应的序列化。
    4. 如果提供了第三个参数，执行相应的格式化。

### 解析选项
`JSON.parse()`也可接收另一个参数，该参数是一个函数，将在每个键值对儿上调用。这个函数叫做还原函数。

### JSON解析器实现
```javascript
var json_parse=function(){
    var at,     //当前字符的索引
        ch,     //当前字符
        escapee = {
            '"':'"',
            '\\':'\\',
            b:'b',
            f:'f',
            n:'n',
            r:'r',
            t:'t'
        },
        text,
        
        //单某处出错时，调用error
        error=function(m){
            throw {
                name:    'SyntaxError',
                message: m,
                at:      at,
                text:    text
            };
        },
        
        
        next=function(c){
            //如果提供了参数c，那么检验它是否匹配当前字符
            if
        },
}();
```