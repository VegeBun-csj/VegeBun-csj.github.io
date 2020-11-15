---
title: "Leetcode100 相同的树"
date: 2020-11-14T16:06:45+08:00
tags:
- 算法
- leetcode
Categories:
- 递归
- 树
---

### **题目描述**

给定两个二叉树，编写一个函数来检验它们是否相同。

如果两个树在结构上相同，并且节点具有相同的值，则认为它们是相同的。

### **题目链接**

> https://leetcode-cn.com/problems/same-tree/
>

### **思路**

> 题目中说 `结构`和`值`相同才是相同的树，所以有一下步骤：
>
> - 所以先对树的`结构`进行判断；
>   - 当都为空的时候，结构相同，递归出口，返回true
>   - 一个为空，一个不为空，结构不同，则递归出口，返回false
>
> - 结构相同时，再对`值`进行判断，如果值不同，则递归出口，返回false
>
> - 值相同再`递归遍历左右子树`
>
> 所以，递归的写法，可以**先考虑所有的递归出口**，然后再写递归具体的流程和返回值

### **代码**

```c++
class Solution {
public:
    bool isSameTree(TreeNode* p, TreeNode* q) {
        //首先判断二叉树的结构是否相同，如果结构都不满足，直接递归出口
        if(p == nullptr && q == nullptr ) return true;      //递归出口   都为空，相等
        if(p == nullptr || q == nullptr) return false;      //递归出口   其中一个为空，所以不等
        //再对值进行判断，值不相同就是递归出口
        if(p -> val != q -> val) return false;    
                            
        //值相同递归
        return (isSameTree(p -> left,q -> left) && isSameTree(p -> right,q -> right));
        
    }
};
```

### **复杂度分析**

> 时间复杂度为O(N)，空间复杂度O(logN)