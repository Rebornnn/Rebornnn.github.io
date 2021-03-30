---
title: 《深入浅出node.js》读书笔记5-1理解Buffer
date: 2019-08-23 15:08:03
categories:
- 读书笔记
- 《深入浅出node.js》
tags:
- node
- 系统学习
description: Buffer
---
在服务端的应用场景中，Node应用需要处理网络协议、操作数据库、处理图片、接收上传文件等，在网络流和文件的操作中，还要处理大量二进制数据，Javascript自有的字符串远远不能满足这些需求，于是Buffer对象应运而生。

## Buffer结构
Buffer是一个像Array的对象，但主要用于操作字节。     
从模块结构和对象结构但层面来认识它。

### 模块结构
Buffer是典型但Javascript和C++结合模块，它将性能相关部分用于C++实现，将非性能相关的部分用于Javascript实现。  
Node在启动时就已加载Buffer，无须通过`require()`引入。

### Buffer对象
Buffer对象类似于数组，它的元素为16进制的两位数，即0到255的数值。
```shell
console.log(Buffer.from('深入浅出node.js','utf-8'));
<Buffer e6 b7 b1 e5 85 a5 e6 b5 85 e5 87 ba 6e 6f 64 65 2e 6a 73> 
```
不同编码的字符串占用的元素个数各不相同。中文字在UTF-8编码下占用**3个元素**，字母和半角标点占用**1个元素**。     
元素赋值规则
- 给元素的赋值小于0，将该值逐次加256，直到得到一个0～255之间的整数
- 如果数值大于255，就逐次减256，直到得到0～255区间内的数值
- 如果是小数，舍弃小数部分，只保留整数部分

### Buffer内存分配
C++层面申请内存，Javascript采用slab机制分配内存。

## Buffer的转换
Buffer可以与字符串之间相互转换。目前支持的字符串编码类型有如下几种：
- ASCII
- UTF-8
- UTF-16LE/UCS-2
- Base64
- Binary
- Hex

### 字符串与Buffer的转换

- 字符串转Buffer: `Buffer.from()`，[文档](http://nodejs.cn/api/buffer.html#buffer_class_method_buffer_from_array)
- Buffer转字符串: `buf.toString()`，[文档](http://nodejs.cn/api/buffer.html#buffer_buf_tostring_encoding_start_end)

## Buffer的拼接
Buffer在使用场景中，通常是以一段一段的方式传输。以下为常见的从输入流中读取内容的示例代码：
```javascript
var fs = require('fs');

var rs = fs.createReadStream('test.md');
var data = '';
rs.on('data', function(chunk){
    data += chunk;
});
rs.on('end', function(){
    console.log(data)
})
```
这段代码在英文环境下没问题，一旦输入流中有宽字节编码时，就有可能出现􏲥乱码符号。    
这个潜藏的问题在于如下代码：
```javascript
data += chunk;
//等价于
data = data.toString() + chunk.toString();
```
### 乱码如何产生？      
`buf.toString()`方法默认以UTF-8编码，中文在UTF-8下占3个字节。当一个中文字的3个字节被分割到两个Buffer对象中时就会出现乱码。      
可以调用`fs.setEncoding('utf-8')`解决该问题。

### 正确拼接Buffer
将多个小Buffer对象拼接为一个Buffer对象，然后通过`icon-lite`一类的模块来转码。

## Buffer性能
Buffer在文件I/O和网络I/O中运用广泛，尤其在网络传输中，它的性能举足轻重。    
在Node构建的Web应用中，可以选择将静态内容预先转换为Buffer，然后直接传输。

### 文件读取
`fs.createReadStream(path, opts)`方法中opts配置参数，可以提高读取速度。特别是*highWaterMark*。      
highWaterMark越大，读取速度越快。