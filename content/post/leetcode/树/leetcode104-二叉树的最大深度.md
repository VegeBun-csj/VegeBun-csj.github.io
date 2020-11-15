---
title: "Leetcode104 二叉树的最大深度"
date: 2020-11-13T14:58:27+08:00
tags:
- 算法
- leetcode
Categories:
- 树
- 递归
---

### **题目描述**

给定一个二叉树，找出其最大深度。

二叉树的深度为根节点到最远叶子节点的最长路径上的节点数。

说明: 叶子节点是指没有子节点的节点。

示例：

> 给定二叉树 [3,9,20,null,null,15,7]，
> 返回它的最大深度 3 。

### **题目链接**

> https://leetcode-cn.com/problems/maximum-depth-of-binary-tree/
>

### **思路**

> 树的最大深度为左右子树的最大深度+1，所以我们只需要计算出左右子树的深度，然后选出最大者+1即可。二叉树的遍历比较适合`递归`
>

### **代码**

```c++
class Solution {
public:
    int maxDepth(TreeNode* root) {
        if(root == nullptr) return 0;
        return max(maxDepth(root -> left),maxDepth(root -> right)) + 1;
    }
};
```

### **复杂度分析**

> 时间复杂度为O(n)
>
> 因为使用的是递归，所以隐式地使用了函数调用栈，栈的深度为树高，所以空间复杂度为O(logN)，