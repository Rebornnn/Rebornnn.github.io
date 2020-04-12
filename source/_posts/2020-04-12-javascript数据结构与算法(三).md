---
title: javascript数据结构与算法3
date: 2020-04-12 14:14:50
categories:
  - 数据结构与算法
tags:
  - javascript
  - 数据结构
  - 学习
  - 集合
description: 集合，不允许值重复的无序数据结构
---

# 集合

## 创建 Set 类

```javascript
class Set {
  constructor() {
    this.items = {};
  }

  has(element) {
    return Object.prototype.call(this.items, element);
  }

  add(element) {
    if (!this.has(element)) {
      this.items[element] = element; // 局限性，element如果为对象会覆盖
      return true;
    }
    return false;
  }

  delete(element) {
    if (this.has(element)) {
      delete this.items[element];
      return true;
    }
    return false;
  }

  clear() {
    this.items = {};
  }

  size() {
    return Object.prototype.keys(this.items).length;
  }

  values() {
    // ES2017+
    return Object.values(this.items);

    // 兼容版本
    // valuesLegcy() {
    //   let values = []
    //   for(let key in this.items) {
    //     if(this.items.hasOwnProperty(key)) {
    //       values.push(this.items[key])
    //     }
    //   }
    //   return values
    // }
  }
}
```

## 集合运算

- 并集:对于给定的两个集合，返回一个包含两个集合中所有元素的新集合。
- 交集:对于给定的两个集合，返回一个包含两个集合中共有元素的新集合。
- 差集:对于给定的两个集合，返回一个包含所有存在于第一个集合且不存在于第二个集
  合的元素的新集合。
- 子集:验证一个给定集合是否是另一集合的子集。

### 并集

```javascript
union(otherSet) {
  const unionSet = new Set()
  this.values().forEach(value => this.add(value))
  otherSet.values().forEach(value => this.add(value))
  return unionSet
}
```

### 交集

```javascript
intersection(otherSet) {
  const intersectionSet = new Set();
  const values = this.values();
  const otherValues = otherSet.values();
  let biggerSet = values;
  let smallerSet = otherValues;
  if (otherValues.length - values.length > 0) {
      biggerSet = otherValues;
      smallerSet = values;
  }
  smallerSet.forEach(value => {
    if (biggerSet.includes(value)) {
      intersectionSet.add(value);
    }
  })
  return intersectionSet;
}
```

### 差集
```javascript
difference(otherSet) {
  const differenceSet = new Set(); 
  this.values().forEach(value => { 
    if (!otherSet.has(value)) { 
      differenceSet.add(value); 
    } 
  });
  return differenceSet;
}
```

### 子集
```javascript
  isSubsetOf(otherSet) {
  if (this.size() > otherSet.size()) {
      return false;
  }
  let isSubset = true;
  this.values().every(value => {
    if (!otherSet.has(value)) {
      isSubset = false;
      return false;
    }
      return true;
    });
  return isSubset;
}
```
