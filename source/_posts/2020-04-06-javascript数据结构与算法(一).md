---
title: javascript数据结构与算法(一)
date: 2020-04-06 15:49:35
categories:
  - 数据结构与算法
tags:
  - javascript
  - 数据结构
  - 学习
  - 栈
  - 队列
description: javascript对象来模拟栈和队列
---

# 栈

## 基于 javascript 对象创建栈

```javascript
class stack {
  constructor() {
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

  // 算法核心
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
  constructor() {
    this.count = 0;
    this.items = {};
    this.lowestCount = 0;
  }

  // 向队列添加元素
  enqueue(element) {
    this.items[this.count] = element;
    this.count++;
  }

  // 从队列移除元素
  dequeue() {
    if (this.isEmpty()) {
      return undefined;
    }
    let result = this.items[this.lowestCount];
    delete this.items[this.lowestCount];
    this.lowestCount++;
    return result;
  }

  // 查看队列头部
  peek() {
    if (this.isEmpty()) {
      return undefined;
    }
    return this.items[this.lowestCount];
  }

  // 验证栈是否为空
  isEmpty() {
    return this.size() === 0;
  }

  // 查看栈的大小
  size() {
    return this.count - this.lowestCount;
  }

  // 清空栈
  clear() {
    this.items = {};
    this.count = 0;
    this.lowestCount = 0;
  }

  toString() {
    if (this.isEmpty()) {
      return "";
    }
    let objString = `${this.items[this.lowestCount]}`;
    for (let i = this.lowestCount + 1; i < this.count; i++) {
      objString = `${objString},${this.items[i]}`;
    }
    return objString;
  }
}
```

## 基于 javascript 对象创建双端队列

```javascript
class Deque {
  constructor() {
    this.items = {};
    this.count = 0;
    this.lowestCount = {};
  }

  // 向双端队列的前端添加元素
  addFront(element) {
    if (this.isEmpry()) {
      this.addBack(element);
    } else if (this.lowestCount > 0) {
      this.lowestCount--;
      this.items[this.lowestCount] = element;
    } else {
      for (var i = count; i > 0; i--) {
        this.items[i] = this.items[i - 1];
      }
      this.count++;
      this.lowestCount = 0;
      this.items[0] = element;
    }
  }

  // 向双端队列的后端添加元素
  addBack(element) {
    this.items[this.count] = element;
    this.count++;
  }

  // 从双端队列前端移除第一个元素
  removeFront() {
    if (this.isEmpty()) {
      return undefined;
    }
    let result = this.items[this.lowestCount];
    delete this.items[this.lowestCount];
    this.lowestCount++;
    return result;
  }

  // 从双端队列后端移除第一个元素
  removeBack() {
    if (this.isEmpry()) {
      return undefined;
    }

    this.count--;
    let result = this.items[this.count];
    delete this.item[this.count];
    return result;
  }

  // 返回双端队列前端的第一个元素
  peekFront() {
    if (this.isEmpty()) {
      return undefined;
    }
    return this.items[this.lowestCount];
  }

  // 返回双端队列后端的第一个元素
  peekBack() {
    if (this.isEmpty()) {
      return undefined;
    }
    return this.items[this.count - 1];
  }

  // 验证栈是否为空
  isEmpty() {
    return this.size() === 0;
  }

  // 查看栈的大小
  size() {
    return this.count - this.lowestCount;
  }

  // 清空栈
  clear() {
    this.items = {};
    this.count = 0;
    this.lowestCount = 0;
  }

  toString() {
    if (this.isEmpty()) {
      return "";
    }
    let objString = `${this.items[this.lowestCount]}`;
    for (let i = this.lowestCount + 1; i < this.count; i++) {
      objString = `${objString},${this.items[i]}`;
    }
    return objString;
  }
}
```

## 使用队列和双端队列来解决问题

### 循环队列——击鼓传花游戏

```javascript
function hotPotato(elementsList, num) {
  const queue = new Queue();
  const elimitatedList = [];

  for (let i = 0; i < elementsList.length; i++) {
    queue.enqueue(elementsList[i]);
  }

  // 算法核心-构建一个循环队列
  while (queue.size() > 1) {
    for (let i = 0; i < num; i++) {
      queue.enqueue(queue.dequeue());
    }
    elimitatedList.push(queue.dequeue());
  }

  return {
    eliminated: elimitatedList,
    winner: queue.dequeue(),
  };
}

const names = ["John", "Jack", "Camila", "Ingrid", "Carl"];
const result = hotPotato(names, 7);

result.eliminated.forEach((name) => {
  console.log(`${name}在击鼓传花游戏中被淘汰。`);
});
console.log(`胜利者: ${result.winner}`);
```

### 两端队列——回文检查器

```javascript
function palindromeChecker(aString) {
  if (
    aString === undefined ||
    aString === null ||
    (aString !== null && aString.length === 0)
  ) {
    return false;
  }

  const deque = new Deque();
  const lowerString = aString.toLocaleLowerCase().split(" ").join("");
  let isEqual = true;
  let firstChar, lastChar;

  for(let i=0;i < lowerString.length;i++) {
    deque.addBack(lowerString.charAt(i))
  }

  // 算法核心
  while(deque.size() > 1 && isEqual) {
    firstChar = deque.removeFront()
    lastChar = deque.removeBack()
    if(firstchar !== lastChar) {
      isEqual = false;
    }
  }

  return isEqual
}

console.log('a', palindromeChecker('a'));
console.log('kayak', palindromeChecker('kayak'));
console.log('Step on no pets', palindromeChecker('Step on no pets'));
```
