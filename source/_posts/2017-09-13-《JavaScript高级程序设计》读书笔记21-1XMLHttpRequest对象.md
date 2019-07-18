---
title: 《JavaScript高级程序设计》读书笔记21.1XMLHttpRequest对象
date: 2017-09-13 09:59:58
categories:
- 读书笔记
- 《JavaScript高级程序设计》
tags:
- JavaScript
- 系统学习
description: Ajax中最重要的一个对象
---
IE7之前版本需要编写函数，构造XMLHttpRequest对象，IE7+不用了。
```javascript
var xhr=new XMLHttpRequest();
```
### 1. XHR的用法
#### 准备发送
使用XHR对象时，要调用第一个方式是`open()`，接收3个参数:    
1. 要发生的请求类型
2. 请求的URL
3. 是否异步发送请求的布尔值
```javascript
xhr.open("get","example.php",false);
```
注意：
URL是相对于执行代码的当前页面；`open()`不会发送请求，只是启动一个请求以备发送。

#### 发送请求
```javascript
xhr.open("get","example.php",false);
xhr.send(null);
```
`send()`接收一个参数，既要作为请求主体发送的数据。如果没有，则传入null。      
调用`send()`后，请求会被分派到服务器。

#### 收到响应
收到响应后，响应的数据会自动填充xhr对象的属性，有4个属性：      
1. responseText:作为响应的主体被返回的文本。
2. responseXML:如果响应的内容类型是"text/xml"或"application/xml"，这个属性中将保存包含着响应数据的XML DOM文档。
3. status：响应的HTTP状态。
4. statusText：HTTP状态的说明。    

在接收到响应后，第一步检查status属性，以确定响应已经成功返回。如果HTTP状态代码为200，则代表成功。此时，responseTxt属性和responseXML属性的内容已经就绪，可以访问。
```javascript
xhr.open("get","example.php",false);
xhr.send(null);

if((xhr.status>=200&&xhr.status<300)||xhr.status=304){
    alert(xhr.responseText);
}else{
    alert("Response was unsuccessful:"+xhr.status);
}
```

####  异步方法
- 检测XHR对象的readyState属性
    - 0：未初始化。尚未调用`open()`方法。
    - 1：启动。已经调用`open()`方法，但尚未调用`send()`方法。
    - 2：发生。已经调用`send()`方法，但尚未接收到响应。
    - 3：接收。已经接收到部分响应数据。
    - 4：完成。已经接收到全部响应数据，而且已经可以在客户端使用了。

只有readyState属性值变动一次，就触发一次readystatechange事件。异步方法就是利用这一点。
```javascript
var xhr=new XMLHttpRequest();
xhr.onreadystatechange=function(){
    if(xhr.readyState==4){
        if((xhr.status>=200&&xhr.status<300)||xhr.status=304){
            alert(xhr.responseText);
        }else{
            alert("Response was unsuccessful:"+xhr.status);
        }
    }
};

xhr.open("get","example.text",true);
xhr.send(null);
```

#### 取消请求
在收到响应之前，使用：
```javascript
xhr.anort();
```

### 2. HTTP头部信息
分为**请求头部**、**响应头部**
#### 请求头部
- 默认情况下，发生xhr请求的同时，还发送以下头部信息：
    - Accept：浏览器能够处理的内容类型
    - Accept-Charest：浏览器能够显示的字符集
    - Accept-Encoding：浏览器能够处理的压缩编码
    - Accept-Language：浏览器当前设置的语言
    - Connection：浏览器与服务器之间连接的类型
    - Cookie：当前页面设置的任何cookie
    - Host：发出请求的页面所在域
    - Referer：发出请求的页面的URI。
    - User-Agent：浏览器的用户代理字符串

**必须在open()方法之后且在send()方法之前调用setRequestHeader()**。            
setRequestHeader()可以设置自定义的请求头部信息，接收两个参数：头部字段的名称和头部字段的值。
```javascript
var xhr=new XMLHttpRequest();
xhr.onreadystatechange=function(){
    if(xhr.readyState==4){
        if((xhr.status>=200&&xhr.status<300)||xhr.status=304){
            alert(xhr.responseText);
        }else{
            alert("Response was unsuccessful:"+xhr.status);
        }
    }
};

xhr.open("get","example.text",true);
xhr.setRequestHeader("MyHeader","MyValue");
xhr.send(null);
```

#### 响应头部
调用`getAllResponseHeaders()`方法则取得包含响应头信息的所有长字符串。


### 3. GET请求
**作用**:最常用于向服务器查询某些信息。    
**方法**：把编码后的、用和号&串联的字符串插入到`open()`的URL末尾。
```javascript
编码函数
function addURLParam(url,name,value){
    url+=url.indexof("?")==-1?"?":"&";
    url+=encodeURLComponent(name)+"="+encodeURLComponent(value);
    return url;
}
```

### 4. POST请求
**作用**:最常用于向服务器发送应该保存的数据。     
**方法**：模仿表单提交
1. 将 Content-Type 头部信息设置为表单提交时的内容类型 application/x-www-form-urlencoded，
2. 是以适当的格式创建一个字符串，使用第 14章介绍的 `serialize()`函数来创建这个字符串。
```javascript
function submitData(){  
    var xhr = new XMLHttpRequest(); 
    xhr.onreadystatechange = function(){ 
        if (xhr.readyState == 4){ 
            if ((xhr.status >= 200 && xhr.status < 300) || xhr.status == 304){ 
                alert(xhr.responseText); 
            } else { 
                alert("Request was unsuccessful: " + xhr.status); 
            } 
        } 
    }; 
 
    xhr.open("post", "postexample.php", true); 
    xhr.setRequestHeader("Content-Type", "application/x-www-form-urlencoded"); 
    var form = document.getElementById("user-info"); 
    xhr.send(serialize(form)); 
} 
 
```