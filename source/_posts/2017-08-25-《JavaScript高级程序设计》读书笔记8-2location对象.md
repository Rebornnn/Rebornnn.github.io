---
title: 《JavaScript高级程序设计》读书笔记8.2location对象
date: 2017-08-25 09:54:00
categories:
- 读书笔记
- 《JavaScript高级程序设计》
tags:
- JavaScript
- 系统学习
description: 即是window对象的属性，也是document对象的属性
---
**不仅提供与当前窗口中加载的文档有关信息，还提供了一些导航功能。**
- 即是`window`对象的属性，也是`document`对象的属性；换句话说，`window.location`和`document.location`引用的是同一个对象。
- 将URL解析为独立片段，不同属性对应不同片段
![clipboard.png](https://ooo.0o0.ooo/2017/05/09/5911d452b17e4.png)

#### 1.查询字符串参数
用来解析查询字符串并返回包含所有参数的一个对象的函数
```javascript
function getQueryStringArgs(){
    
    //取得查询字符串并去掉问号？
    var qs=(location.search.length>0?location.search.substring(1):""),
    
    //保存数据的对象
    args={},
    
    //取得每一项
    items=(qs.length?qs.split("&"):[]),
    
    item=null,
    name=null,
    value=null;
    
    //逐个将每一项添加到args对象中
    for(var i;i<items.length;i++){
        item=items[i].split("=");
        name=decodeURIComponent(item[0]);
        value=decodeURIComponent(item[1]);
        
        if(name.length){
            args[name]=value;
        }
    }
    return args;
}

调用例子
    
    //假设查询字符串是？q=javascript&num=10
    var args=getQueryStringArgs();
    
    alert(args["q"]);     //javascript
    alert(args["num"]);   //10
```

#### 2.位置操作
- `location.assign()`，需要传递一个URL。与`location.href()`和`window.location()`一样。
- 修改`location`的属性（`hash`,`search`,`hostname`,`parhname`,`port`）也可以改变当前加载的页面,且页面会以新URL重新加载（hash除外）。
- `location.replace()`，不能回到前一个页面。
- `location.reload()`，重新加载当前显示的页面。