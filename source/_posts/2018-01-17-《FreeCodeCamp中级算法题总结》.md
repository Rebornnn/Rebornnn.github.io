---
title: 《FreeCodeCamp中级算法题总结》
date: 2018-01-17 15:23:55
categories: F算法
tags: FCC中级算法
description: FCC算法题总结
---
### Sum All Numbers in a Range
sumAll([5, 10]) should return 45.   
sumAll([10, 5]) should return 45.   
```javascript
//ES5
function sumAll(arr){
    var max=Math.max.apply(null,arr),
        min=Math.min.apply(null,arr);
        
    for(var i=min+1;i<max;i++){
        arr.push(i);
    }
    
    return arr.reduce(function(sum,value){
        return sum+value;
    });
}

//ES6
function sumAll(arr){
    //运用扩展模板
    var max=Math.max(...arr),
        min=Math.min(...arr);
        
    for(var i=min+1;i<max;i++){
        arr.push(i);
    }
    
    return arr.reduce(function(sum,value){
        return sum+value;
    });
}

sumAll([5,10]);
```

### Diff Two Arrays
#### 两数组合并去重
```javascript
//方法一：先找出相同项，合并两数组，筛选出不同项
function diffArray(arr1, arr2) {
  var sameArr=[];
  var concatArr=arr1.concat(arr2);
  
  for(var i=0;i<arr1.length;i++){
    if(arr2.indexOf(arr1[i])!==-1){
      sameArr.push(arr1[i]);
    }
  }
  
  var newArr=concatArr.filter(function(elem){
    return sameArr.indexOf(elem)==-1;
  });
  return newArr;
}

//方法二：两次循环，先以arr1为基准在arr2中查找不同，再反过来
function diffArray(arr1, arr2) {
  var newArr = [];

  for(var i=0;i<arr1.length;i++){
    if(arr2.indexOf(arr1[i])==-1){
      newArr.push(arr1[i]);
    }
  }
  
  for(var j=0;j<arr2.length;j++){
    if(arr1.indexOf(arr2[j])==-1){
      newArr.push(arr2[j]);
    }
  }
  
  return newArr;
}

diffArray([1, 2, 3, 5], [1, 2, 3, 4, 5]);
```

### Roman Numeral Converter
#### 将5位以下的阿拉伯数字转化为罗马数字
```javascript
//暴力对应法：将阿拉伯数字和罗马数字分别用数组列出，再一一对应
function convertToRoman(num) {
  //列出阿拉伯数字 和 罗马数字数组
  var arabicNum=[1,2,3,4,5,6,7,8,9,10,20,30,40,50,60,70,80,90,100,200,300,400,500,600,700,800,900,1000];
  var romanNum=['I','II','III','IV','V','VI','VII','VIII','IX','X','XX','XXX','XL','LX','LX','LXX','LXXX','XC','C','CC','CCC','CD','D','DC','DCC','DCCC','CM','M'];
  
  //将num每个数字分拆成数组
  num=new Number(num).toString();
  var numArr=num.split('');
  
  //阿拉伯数字与罗马数字对应
  var newArr=numArr.map(function(elem,index,arr){
    //单列出2000及以上的情况
    if(elem*Math.pow(10,arr.length-index-1)>1000){
      return 'M'.repeat(elem);
    }
    var getIndex=arabicNum.indexOf(elem*Math.pow(10,arr.length-index-1));
    return romanNum[getIndex];
  });
  
  return newArr.join('');
}

convertToRoman(2014);

//不需要拆分数字的方法
//重点是1.罗马数字中有重复的且在关键字母右边的数字踢出；2.阿拉伯数字和罗马数字从大到小排列；3.循环递减
function convertToRoman(num){
    var arabic=[1000,900,500,400,100,90,50,40,10,9,5,4,1];
    var roman=['M','CM','D','CD','C','XC','L','XL','X','IX','V','IV','I'];
    var str='';
    
    arabic.forEach(function(elem,index,arr){
        while(num>=elem){
            str+=roman[index];
            num-=elem
        }
    });
    
    return str;
}
```

### Wherefore art thou
#### 由一个对象筛选另一个对象
```javascript
//用到Object.keys()、forEach()、every()
function whatIsInAName(collection, source) {
  
  var arr = [];
  var proArr=Object.keys(source);
 
  collection.forEach(function(elem,index){
    //所有属性值必须都匹配上
    var pass=proArr.every(function(item,index){
      return (elem[item]===source[item]);
    });
    
    if(pass){
      arr.push(elem);
    }
  });
  
  
  return arr;
}

whatIsInAName([{ first: "Romeo", last: "Montague" }, { first: "Mercutio", last: null }, { first: "Tybalt", last: "Capulet" }], { last: "Capulet" });
```

### Search and Replace
#### 字符串类型数据，将第二个参数替换成第三个参数，同时保持首字母大小写一致
```javascript
//运用search()、正则表达式
function myReplace(str, before, after) {
  //首字母大小写替换
  var newAfter=after.replace(/[a-z]/,function(match){
    return match.toUpperCase();
  });
  
  //判断首字母是否需要大写
  after=(before.search(/[A-Z]/)!==-1) ? newAfter:after;
  return str.replace(before,after);
}

myReplace("A quick brown fox jumped over the lazy dog", "jumped", "leaped");

//运用charAt()、自身判断
function myReplace(str, before, after) {
  if(before.charAt(0)===before.charAt(0).toUpperCase()){
      after=after.charAt(0).toUpperCase()+after.slice(1);
  }
  return str.replace(before,after);
}
```

### Pig Latin
#### 根据儿童黑话的规则改变字符串
```javascript
//自己的做法，筛选条件应该可以更精进一点
function translatePigLatin(str) {
  var patternV=/[aeiou]/i;
  var patternCC=/[cmthsgl]/i;
  if(patternV.test(str.charAt(0))){
    str=str+'way';
    return str;
  }
  
  if(patternCC.test(str.charAt(0))&&patternCC.test(str.charAt(1))){
    str=str.slice(2)+str.slice(0,2)+"ay";
  }else{
    str=str.slice(1)+str.charAt(0)+'ay';
  }
  return str;
}

translatePigLatin("california");
```


### DNA Pairing
#### DNA匹配，A配T，C配G
```javascript
//运用spilt()、map()、switch()
function pairElement(str) {
  var noPair=str.split('');
  str=noPair.map(function(elem,index,arr){
    switch(elem){
      case "A:
        return ['A',"T"];
        
      case "T":
        return ["T","A"];
        
      case "C":
        return ["C","G"];
        
      case "G":
        return ['G','C'];
    }
  });
  
  return str;
}

pairElement("GCG");
```

### Missing letters
#### 找出缺失的字母
```javascript
function fearNotLetter(str) {
  var num=str.charCodeAt(0);
  var letter;
  for(var i=1;i<str.length;i++){
    if(str.charCodeAt(i)!==(num+i)){
      letter=String.fromCharCode(num+i);
      num+=1;       //遇到缺失的字母需要补1
    }
  }
  
  
  return letter;
}

fearNotLetter("abcefgh");
```

### Boo who
#### 判断是不是布尔值
```javascript
//恒等法
function booWho(bool) {
  if(bool===true||bool===false){
    bool=true;
  }else{
    bool=false;
  }
  return bool;
}

booWho(true);

//typeof法
function boo(bool) {  
  return typeof bool==='boolean';  
}  
  
boo(null);  
```

### Sorted Union
#### 排序并集,多个数组合并,依次去重
```javascript
//先合并,再筛选
function uniteUnique() {
  var argArr=Array.prototype.slice.call(arguments);
  var sumArr=argArr.reduce(function(acc,elem,index,arr){
    return acc.concat(elem);
  },[]);
  
  var resArr=sumArr.filter(function(elem,index,arr){  
    return index==arr.indexOf(elem);
  });
  
  return resArr;
}

uniteUnique([1, 3, 2], [5, 2, 1, 4], [2, 1]);


```

### Convert HTML Entities
#### 文本转换
```javascript
//利用switch方法
function convertHTML(str) {
  // &colon;&rpar;
  str=str.replace(/[&<>"']/g,function(match,offset,string){
    switch(match){
      case "&":
        return "&amp;";
        
      case "<":
        return "&lt;";
        
      case ">":
        return "&gt;";
        
      case '"':
        return "&quot;";
        
      case "'":
        return "&apos;";
    }
    });
  return str;
}

convertHTML("Shindler's List");

//利用对象的
function convertHTML(str) {
  // &colon;&rpar;
  var enityMap={
    '&':'&amp;',
    '<':'&lt;',
    '>':'&gt;',
    '"':"&quot;",
    "'":'&apos;'
  };
  
  str=str.replace(/[&<>"']/g,function(match){
    return enityMap[match];
  });
  return str;
}

convertHTML("Shindler's List");
```

### Spinal Tap Case
#### 单词链接，将大写、下划线_、空格变成'-'
```javascript
//关键要分两步，而不是用选择：先将大写变成'下划线_+大写'模式，再用replace一次性都替换成'-'
function spinalCase(str) {
  //第一步，将大写变成'下划线_+小写'模式
  str=str.replace(/[A-Z]/g,function(match){
     if(str.indexOf(match)>0){
       return '_'+match;
     }else{
       return match;
     }
  });
  
  //第二步，用replace一次性都替换成'-'
  str=str.toLowerCase();
  str=str.replace(/[_\s]+/g,'-');
 
  return str;
}

spinalCase('AllThe-small Things');

//链式调用，关键是/^_/将首字符大写排除掉
function spinalCase(str){
    str=str.replace(/([A-Z])/g,'_$1').
        replace(/^_/,'').
        replace(/[_\s]+/g,'-').
        toLowerCase();
    
    return str;
}
```

### Sum All Odd Fibonacci Numbers
#### 求小于给定数值的奇数斐波那契数之和
```javascript
//不带数组记忆优化的递归实现斐波那契数列
function sumFibs(num) {
  function fibs(n){
    return n<=2?fibs(n-1)+fibs(n-2);
  }
  
  var sumNum=0;
  for(var i=1;i<=num;i++){
    if(fibs(i)<=num&&fibs(i)%2==1){
      sumNum+=fibs(i);
    }
  }
  return sumNum;
}

sumFibs(10);

//带数组记忆优化的for循环
function sumFibs(num) { 
  var arr=[1,1];
  //判断条件时必须是arr[i-1]，因为在这里arr[i]还没被计算是undefined
  for(var i=2;arr[i-1]<=num;i++){
    arr[i]=arr[i-1]+arr[i-2];
  }
  arr.pop();
  
  var sum = 0;
  for(var j = 0; j < arr.length; j++) {
    if(arr[j] % 2 !== 0) {
      sum += arr[j];
    }
  }
  return sum;

}

sumFibs(4);

//带数组记忆优化的do..while循环
function sumFibs(num) {
  var arr = [];
  var i = 2;
  do{
    arr[0] = 1;
    arr[1] = 1;
    arr[i] = arr[i-2] + arr[i-1];
    i++;
  }while(arr[i-1]<=num); //判断条件时必须是arr[i-1]，因为在这里arr[i]还没被计算是undefined
  
  arr.pop();
  
  var sum = 0;
  for(var j = 0; j < arr.length; j++) {
    if(arr[j] % 2 !== 0) {
      sum += arr[j];
    }
  }
  return sum;
}

sumFibs(19)

//简单粗暴的直接加法实现斐波那契数列，不能保存数列，在只使用一次的情况非常合适
function sumFibs(num) {
  var a=0,b=0,c=1,sum=0;
  for(var i=0;c<=num;i++){
    sum+=(c%2==1?c:0);
    a=b;
    b=c;
    c=a+b;
  }
  return sum;
}
```


### Sum All Primes
### 求小于给定数值的质数之和
```javascript
function sumPrimes(num) {
  //判断质数的函数
  function isPrime(n) {
    if (n <= 3) { return n > 1; }
    if (n % 2 == 0 || n % 3 == 0) { return false; }
 
    for (var  i = 5; i * i <= n; i += 6) {
        if (n % i == 0 || n % (i + 2) == 0) { return false; }
    }
    return true;
  }
  
  var sum=0;
  while(num>0){
    if(isPrime(num)){
      sum+=num;
    }
    --num;
  }
  return sum;
}

sumPrimes(10);
```


### Smallest Common Multiple
#### 求多个数的最小公倍数
```javascript
//穷尽法
function smallestCommons(arr) {
  arr.sort();
  for(var i=arr[0]+1;i<arr[1];i++){
    arr.push(i);
  }
  
  var base=arr[0]*arr[1];
  while(true){
    var boo=arr.every(function(elem){
      return base%elem==0;
    });
    
    if(boo){
      return base;
    }
    
    ++base;
  }
}

smallestCommons([23,18]);
 
//欧几里德算法（辗转相除法）求最大公约数，再由最大公约数求最小公倍数
function smallestCommons(arr) {
  arr.sort();
  function euc(a,b){
    if(a%b===0){return b;}
    return euc(b,a%b);
  }
  
  var num=arr[0];
  for(var i=arr[0]+1;i<=arr[1];i++){
    num*=i/euc(num,i);   //十分留意此行代码，还是有点想不通
  }
  
  return num;

}


smallestCommons([23,18]);
```

### Finders Keepers
#### 根据第二个参数筛选数组
```javascript
function findElement(arr, func) {
  arr=arr.filter(func);
  var num = arr[0];
  return num;
}

findElement([1, 2, 3, 4], function(num){ return num % 2 === 0; });
```

### Drop it
#### Drop the elements of an array , starting from the front, until the predicate  returns true
```javascript
//使用slice()方法
function dropElements(arr, func) {
  // Drop them elements.
  for(var i=0;i<arr.length;i++){
    if(func(arr[i])){
       return arr.slice(i);
    }
  }
  
  //当所有数字元素都不匹配时
  return [];
}

dropElements([1, 2, 3,4], function(n) {return n >= 3; });

//使用shift()方法
function dropElements(arr, func) {
  for(var i=arr.length;i>0;i--){
    if(!func(arr[0])){
      arr.shift();
    }
  }
 return arr; 
}
```

### Steamroller
#### 数组扁平化处理
```javascript
//嵌套问题，得用递归
function steamrollArray(arr) {
  var newArr=[];
  function steamer(item){
    if(Array.isArray(item)){
      item.forEach(steamer);
    }else{
        newArr.push(item);
    }
  }
  
  arr.forEach(steamer);
  
  return newArr;
}

steamrollArray([1, [2], [3, [[4]]]]);
```

### Brainy Ageints
#### 二进制代码转换为字符
```javascript
function binaryAgent(str) {
  str=str.replace(/[01]+[\s]?/g,function(match){  //parseInt('$&',2)不起作用，还是不解
    return String.fromCharCode(parseInt(match,2));
  });
  return str;
}

binaryAgent("01000001 01110010 01100101 01101110 00100111 01110100 00100000 01100010 01101111 01101110 01100110 01101001 01110010 01100101 01110011 00100000 01100110 01110101 01101110 00100001 00111111");
```

### Everything Be True
#### 判断对象里给定的属性值是否为真
```javascript
function truthCheck(collection, pre) {
  pre=collection.every(function(elem){
    return !!elem[pre];            //强制转为布尔值
  });
  return pre;
}

truthCheck([{"user": "Tinky-Winky", "sex": "male"}, {"user": "Dipsy", "sex": "male"}, {"user": "Laa-Laa", "sex": "female"}, {"user": "Po", "sex": "female"}], "sex");
```


### Arguments Optional
#### 参数操作
```javascript
function addTogether() {
  var arg=Array.prototype.slice.call(arguments);
  var boo=arg.every(function(elem){
    return typeof elem=='number';
  });
  if(boo&&arg.length==2){
    return arg[0]+arg[1];
  }else if(boo&&arg.length==1){
    return function(){
      if(typeof arguments[0]=='number'){return arg[0]+arguments[0];}
    };
  }
}

addTogether(2)(3);
```