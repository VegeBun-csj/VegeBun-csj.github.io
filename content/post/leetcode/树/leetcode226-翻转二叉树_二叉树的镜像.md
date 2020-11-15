---
title: "Leetcode226 翻转二叉树_二叉树的镜像"
date: 2020-11-14T20:58:21+08:00
tags:
- 算法
- leetcode
Categories:
- 递归
- 树
---

### **题目描述**

> 翻转一棵二叉树。

![image-20201115090347701](https://cdn.jsdelivr.net/gh/VegeBun-csj/Images/image-20201115090347701.png)

### **题目链接**

> https://leetcode-cn.com/problems/invert-binary-tree/
>

### **思路**

> 首先可以看到，翻转后的二叉树就是：
>
> - 以根节点为基础交换两棵左右子树
> - 交换完成后再对左右子树进行同样的操作（递归操作左右子树）

![img](https://cdn.jsdelivr.net/gh/VegeBun-csj/Images/{CB9CD3D3-5BB2-5E32-6255-25045140B67C}.png)

### **代码**

```c++
class Solution {
public:
    TreeNode* invertTree(TreeNode* root) {
        if(root == nullptr) return root;
        //翻转根节点的左右子树
        TreeNode* r = root -> right;
        root -> right = root -> left;
        root -> left = r;
        //递归翻转左右孩子的子树
        invertTree(root -> left);
        invertTree(root -> right);
        return root;
    }
};
```

### **复杂度分析**

> 时间复杂度为 O(N) ,空间复杂度为O(logN)