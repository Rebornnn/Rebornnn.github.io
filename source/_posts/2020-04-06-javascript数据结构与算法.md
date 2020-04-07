---
title: 基于javascript对象创建Stack类
date: 2020-04-06 15:49:35
categories:
  - 数据结构与算法
tags:
  - javascript
  - 数据结构
  - 学习
description: 使用一个javascript对象来存储所有的栈元素
---

# 栈

## 基于 javascript 对象创建栈

```javascript
class stack {
  constract() {
    this.count = 0;
    this.items = {};
  }

  // 向栈中插入元素
  push(element) {
    this.items[this.count] = element;
    this.count++;
    return this.count;
  }

  // 从栈中弹出元素
  pop() {
    if (this.isEmpry()) {
      return undefined;
    }

    this.count--;
    let result = this.items[this.count];
    delete this.item[this.count];
    return result;
  }

  // 查看栈顶值
  peek() {
    if (this.isEmpty()) {
      return undefined;
    }
    return this.items[this.count - 1];
  }

  // 验证栈是否为空
  isEmpty() {
    return this.count === 0;
  }

  // 查看栈的大小
  size() {
    return this.count;
  }

  // 清空栈
  clear() {
    /* while (!this.isEmpty()) {
        this.pop();
      } */
    this.items = {};
    this.count = 0;
  }

  
  toString() {
    if (this.isEmpty()) {
      return "";
    }
    let objString = `${this.items[0]}`;
    for (let i = 1; i < this.count; i++) {
      objString = `${objString},${this.items[i]}`;
    }
    return objString;
  }
}
```

## 用栈解决的问题

### 十进制转二进制

```javascript
function decimalToBinary(decNumber) {
  // const remStack = new Stack()
  const remStack = [];
  let number = decNumber;
  let rem;
  let binaryString = "";

  while (number > 0) {
    rem = Math.floor(number % 2);
    remStack.push(rem);
    number = Math.floor(number / 2);
  }

  while (remStack.length) {
    binaryString += remStack.pop().toString();
  }

  return binaryString;
}

console.log(decimalToBinary(10));
```

### 进制转换算法

```javascript
function baseConverter(decNumber, base) {
  // const remStack = new Stack()
  const remStack = [];
  const digits = "0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ";
  let number = decNumber;
  let rem;
  let baseString = "";

  if (!(base >= 2 && base <= 36)) {
    return;
  }

  while (number > 0) {
    rem = Math.floor(number % base);
    remStack.push(rem);
    number = Math.floor(number / base);
  }

  while (remStack.length) {
    baseString += digits[remStack.pop()];
  }

  return baseString;
}

console.log(baseConverter(10, 16));
```

# 队列和双端队列

## 基于 javascript 对象创建队列
```javascript
class Queue {
  constructor {
    this.count = 0;
    this.items = {};
    this.lowestCount = 0;
  }
}
```