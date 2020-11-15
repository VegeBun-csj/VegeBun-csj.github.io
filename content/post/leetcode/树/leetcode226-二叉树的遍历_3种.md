---
title: "Leetcode226 二叉树的遍历_3种"
date: 2020-11-14T21:00:34+08:00
tags:
- 算法
- leetcode
Categories:
- 递归
- 树
---

### **题目描述**

对二叉树进行前中后序遍历

### **题目链接**

> https://leetcode-cn.com/problems/binary-tree-preorder-traversal/
>
> https://leetcode-cn.com/problems/binary-tree-inorder-traversal/
>
> https://leetcode-cn.com/problems/binary-tree-postorder-traversal/

### **思路**

> 递归遍历，三种遍历的唯一区别就是对`根节点的操作`和`递归操作左右子树`的顺序不一样，

### **代码**

> 前序遍历——根左右

```c++
class Solution {
public:

    void pre(TreeNode* root,vector<int>& res){
        if(root == nullptr) return;
        res.push_back(root -> val);	//根
        pre(root -> left,res);		//左
        pre(root -> right,res);		//右
    }

    vector<int> preorderTraversal(TreeNode* root) {
        vector<int> res;
        pre(root,res);
        return res;
    }
};
```

> 中序遍历——左根右

```c++
public:

    void mid_order(TreeNode* root,vector<int>& res){
        if(root == nullptr) return;
        mid_order(root -> left,res);	//左
        res.push_back(root -> val);	//根
        mid_order(root -> right,res);	//右
    }

    vector<int> inorderTraversal(TreeNode* root) {
        vector<int> res;
        mid_order(root,res);
        return res;
    }
};
```

> 后序遍历——左右根

```c++
public:

    void post_order(TreeNode* root,vector<int>& res){
        if(root == nullptr) return;
        post_order(root -> left,res);	//左
        post_order(root -> right,res);	//右
        res.push_back(root -> val);	//根
    }

    vector<int> postorderTraversal(TreeNode* root) {
        vector<int> res;
        post_order(root,res);
        return res;
    }
};
```

### **复杂度分析**

> 时间复杂度为 O(n)，空间复杂度为O(logN)