---
title: javascript数据结构与算法5
date: 2020-05-03 17:45:11
tags:
  - javascript
  - 数据结构
  - 学习
  - 树
description: 树，存储需要快速查找的数据的非顺序数据结构
---

## 树的相关术语

![](http://img.aisss.top/FnhO6vaXorOB3X6jyBjNlI50kG3K)

## 二叉树和二叉搜索树

**二叉树**中的节点最多只能有两个子节点:一个是左侧子节点，另一个是右侧子节点。  
**二叉搜索树(BST)**是二叉树的一种，但是只允许你在左侧节点存储(比父节点)小的值， 在右侧节点存储(比父节点)大的值。

### 创建 BinarySearchTree 类

![](http://img.aisss.top/FklgY35zzl1TszkQA4dzuLYK2aOi)

```javascript
class Node {
  constructor(key) {
    this.key = key;
    this.left = null;
    this.right = null;
  }
}

class BinarySearchTree extends Node {
  constructor(compareFn = defaultCompare) {
    this.compareFn = compareFn;
    this.root = null;
  }

  insert(key) {
    if (this.root == null) {
      this.root = new Node(key);
    } else {
      this.insertNode(this.root, key);
    }
  }

  insertNode(node, key) {
    if (this.compareFn(key, node.key) === Compare.LESS_THAN) {
      if (node.left == null) {
        node.left = new Node(key);
      } else {
        this.insertNode(node.left, key);
      }
    } else {
      if (node.right == null) {
        node.right = new Node(key);
      } else {
        this.insertNode(node.right, key);
      }
    }
  }
}
```

## 树的遍历

访问树的所有节点有三种方式:中序、先序、后序。

### 中序遍历

**中序遍历**是一种以上行顺序访问 BST 所有节点的遍历方式，也就是以从最小到最大的顺序访问所有节点。  
**中序遍历**的一种应用就是对树进行排序操作。
![](http://img.aisss.top/FtnsenGwteLSa9WalOOe0iVGBpWj)

```javascript
inOrderTraverse(callback) {
  this.inOrderTraverseNode(this.root, callback)
}

inOrderTraverseNode(node, callback) {
  if(node != null) {
    this.inOrderTraverseNode(node.left, callback)
    callback(node.key)
    this.inOrderTraverseNode(node.right, callback)
  }
}
```

### 先序遍历

**先序遍历**是以优先于后代节点的顺序访问每个节点的。先序遍历的一种应用是打印一个结构化的文档。
![](http://img.aisss.top/FjUezogYNwOXM1PyOhj01sqnpWLY)

```javascript
preOrderTraverse(callback) {
  this.preOrderTraverseNode(this.root, callback);
}

preOrderTraverseNode(node, callback) {
  if (node != null) {
    callback(node.key);
    this.preOrderTraverseNode(node.left, callback);
    this.preOrderTraverseNode(node.right, callback);
  }
}
```

### 后序遍历

**后序遍历**则是先访问节点的后代节点，再访问节点本身。后序遍历的一种应用是计算一个目录及其子目录中所有文件所占空间的大小。
![](http://img.aisss.top/FgMr6rUWcloyA0NX-3XRIrxQ5B0d)

```javascript
postOrderTraverse(callback) {
  this.postOrderTraverseNode(this.root, callback);
}

postOrderTraverseNode(node, callback) {
  if (node != null) {
    this.postOrderTraverseNode(node.left, callback);
    this.postOrderTraverseNode(node.right, callback);
    callback(node.key);
  }
}
```

## 搜索树中的值

在树中，有三种经常执行的搜索类型:

- 搜索最小值
- 搜索最大值
- 搜索特定的值

### 搜索最大值和最小值

![](http://img.aisss.top/Fg9NPJK4iNFB3AP31BNyWDTNUlue)

```javascript
min() {
  return this.minNode(this.root)
}

minNode(node) {
  let current = node;
  while(current != null && current.left != null) {
    current = current.left
  }
  return current
}

max() {
  return this.maxNode(this.root)
}

maxNode(node) {
  let current = node;
  while(current != null && current.right != null) {
    current = current.right
  }
  return current
}
```

### 搜索一个特定的值

```javascript
search(key) {
  return this.searchNode(this.root, key)
}

searchNode(node, key) {
  if(node == null) {
    return false
  }
  if(this.compareFn(node.key, key) === Compare.LESS_THAN) {
    return this.searchNode(node.left, key)
  } else if(this.compareFn(node.key, key) === Compare.BIGGER_THAN) {
    return this.searchNode(node.right, key)
  } else {
    return true
  }
}
```

### 移除一个节点

```javascript
remove(key) {
  this.root = this.removeNode(this.root, key)
}

removeNode(node, key) {
if(node == null) {
    return false
  }
  if(this.compareFn(node.key, key) === Compare.LESS_THAN) {
    node.left = this.removeNode(node.left, key)
    return node
  } else if(this.compareFn(node.key, key) === Compare.BIGGER_THAN) {
    node.right = this.removeNode(node.right, key)
    return node
  } else {
    // 移除一个叶节点
    if(node.left == null && node.right == null) {
      node = null
      return node
    }

    // 移除有一个左侧或者右侧子节点的节点
    if(node.right == null) {
      node = node.left
      return node
    } else if(node.left == null) {
      node = node.right
      return node
    }

    // 移除有两个子节点的节点
    const aux = this.minNode(node.right)
    node.key = aux.key
    node.right = this.removeNode(node.right, aux.key)
    return node
  }
}
```

## 自平衡树

**AVL 树**是一种自平衡二叉搜索树，意思是任何一个节点左右两侧子树的高度之差最多为 1。

### Adelson-Velskii-Landi 树(AVL 树)

AVL 树是一种自平衡树。添加或移除节点时，AVL 树会尝试保持自平衡。任意一个节点（不论深度）的左子树和右子树高度最多相差 1.添加或移除节点时，AVL 树会尽可能尝试转换为完全树。

#### 节点的高度和平衡因子

节点的高度：从节点到其任意子节点的边的最大值。如图：
![](http://img.aisss.top/Fnut3toXeg6Zw91SNm2teTNEbOOG)

```javascript
getNodeHeight(node) {
  if(node == null) {
    return -1
  }
  return Math.max(this.getNodeHeight(node.left), this.getNodeHeight(node.right)) + 1
}
```

计算一个节点的平衡因子并返回其值的代码：

```javascript
const BalanceFactor = {
  UNBALANCED_RIGHT: 1,
  SLIGHTLY_UNBALANCED_RIGHT: 2,
  BALANCED: 3,
  SLIGHTLY_UNBALANCED_LEFT: 4,
  UNBALANCED_LEFT: 5
};

getBalanceFactor(node) {
  const heightDifference = this.getNodeHeight(node.left) - this.getNodeHeight(node.right)
  switch(heightDifference) {
    case -2:
      return BalanceFactor.UNBALANCED_RIGHT;
    case -1:
      return BalanceFactor.SLIGHTLY_UNBALANCED_RIGHT;
    case 1:
      return BalanceFactor.SLIGHTLY_UNBALANCED_LEFT;
    case 2:
      return BalanceFactor.UNBALANCED_LEFT;
    default:
      return BalanceFactor.BALANCED;
  }
}
```

#### 平衡操作——AVL旋转
向 AVL 树插入节点时，可以执行单旋转或双旋转两种平衡操作，分别对应四种场景。
- 左-左（LL）：向右的单旋转
- 右-右（RR）：向左的单旋转
- 左-右（LR）：向右的双旋转（先LL旋转，再RR旋转）
- 右-左（RL）：向左的双旋转（先RR旋转，再LL旋转）

##### 左-左(LL):向右的单旋转
这种情况出现于节点的左侧子节点的高度大于右侧子节点的高度时，并且左侧子节点也是平衡或左侧较重的，如下图所示:
![](http://img.aisss.top/Fhlcev-K4oq4Hccf_JExO4rs6VSV)  
```javascript
rotationLL(node) {
  const tmp = node.left
  node.left = tmp.right
  tmp.right = node;
  return tmp
}
```

##### 右-右(RR):向左的单旋转
它出现于右侧子节点的高度大于左侧子节点的高度，并且右侧子节点也是平衡或右侧较重的,如下图所示:
![](http://img.aisss.top/FvNAshmxDXSN5acRMQQ6g11pSrlh)  
```javascript
rotationRR(node) {
  const tmp = node.right
  node.right = tmp.left
  tmp.left = node
  return tmp
}
```

##### 左-右(LR):向右的双旋转
左侧子节点的高度大于右侧子节点的高度，并且左侧子节点右侧较重。在这种情况下，我们可以对左侧子节点进行左旋转来修复，这样会形成左左的情况，然后再对不平衡的节点进行一个右旋转来修复，如下图所示： 
![](http://img.aisss.top/FkCTa7xvh33fUqKfw_llqmHTAihh)
![](http://img.aisss.top/FgLKjPoTn1bMazLF662LtwEV77iK)
基本上，先做一次LL旋转，再做一次RR旋转
```javascript
rotationLR(node) {
  node.right = this.rotationLL(node.right)
  return this.rotationRR(node)
}
```

##### 右-左(RL):向左的双旋转
右侧子节点的高度大于左侧子节点的高度，并且右侧子节点左侧较重。在这种情况下我们可以对右侧子节点进行右旋转来修复，这样会 形成右-右的情况，然后我们再对不平衡的节点进行一个左旋转来修复，如下图所示。  
![](http://img.aisss.top/FsYJhtZoEKRod7Dt7Or_AFQ7F67g)  
![](http://img.aisss.top/FlQaw-_KxgF7uALV04tJXtvfmLXL)  
基本上，先做一次RR旋转，在做一次LL旋转
```javascript
rotationRL(node) {
  node.left = this.rotationRR(node.left)
  return this.rotationLL(node)
}
```

#### 向AVL树插入节点
插入树后还需验证是否平衡，如果不是，需要进行必要的旋转操作
```javascript
insert(key) {
  this.root = this.insertNode(this.root, key)
}

insertNode(node, key) {
  // 像在BST树中一样插入节点
  if(node == null) {
    return new Node(key)
  } else if(this.compareFn(key, node.key) === Compare.LESS_THAN) {
    node.left = this.insertNode(node.left, key)
  } else if(this.compareFn(key, node.key) === Compare.BIGGER_THAN) {
    node.right = this.insertNode(node.right, key)
  } else {
    return node // 重复的键
  }

  const balanceFactor = this.getBalanceFactor(node)
  if(balanceFactor === BalanceFactor.UNBALANCED_LEFT) {
    if(this.compareFn(key, node.left.key) === Compare.LESS_THAN) {
      node = this.rotationLL(node)
    } else {
      return this.rotationLR(node)
    }
  }
  if(balanceFactor === BalanceFactor.UNBALANCED_RIGHT) {
    if(this.compareFn(key, node.left.key) === Compare.BIGGER_THAN) {
      node = this.rotationRR(node)
    } else {
      return this.rotationRL(node)
    }
  }
  return node
}
```
#### 从AVL树中移除节点
```javascript
removeNode(node, key) {
  node = super.removeNode(node, key);
  if(node == null) {
    return node; //null, 不需要进行平衡
  }

  // 检测是否平衡
  const balanceFactor = this.getBalanceFactor(node)
  if(balanceFactor = BalanceFactor.UNBALANCED_LEFT) {
    const balanceFactorLeft = this.getBalanceFactor(node.left);
    if(balanceFactorLeft === BalanceFactor.BALANCED || balanceFactorLeft === BalanceFactor.SLIGHTLY_UNBALANCED_LEFT) {
      return this.rotationLL(node)
    }
    if(balanceFactorLeft === BalanceFactor.SLIGHTLY_UNBALANCED_RIGHT) {
      return this.rotationLR(node.left)
    }
  }
  if(balanceFactor = BalanceFactor.UNBALANCED_RIGHT) {
    const balanceFactorLeft = this.getBalanceFactor(node.right);
    if(balanceFactorRight === BalanceFactor.BALANCED || balanceFactorRight === BalanceFactor.SLIGHTLY_UNBALANCED_RIGHT) {
      return this.rotationRR(node)
    }
    if(balanceFactorRight === BalanceFactor.SLIGHTLY_UNBALANCED_LEFT) {
      return this.rotationLR(node.right)
    }
  }
  return node
}
````


### 红黑树
**红黑树**也是一个自平衡二叉搜索树。
红黑树适用于多次插入和删除。  
在红黑树中，每个节点都遵循以下规则：
- 每个节点不是红的就黑的
- 树的根节点树黑的
- 所有叶节点都是黑的（用NULL引用表示的节点）
- 如果一个节点是红的，那么它的两个子节点都是黑的
- 不能有两个相邻的红节点，一个红节点不能有红的父节点或子节点
- 从给定的节点到它的后代节点（NULL叶节点）的所有路径包含相同数量的黑色节点
