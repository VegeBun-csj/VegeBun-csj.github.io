---
title: "Leetcode129 求根到叶子节点数字之和"
date: 2020-11-15T15:30:47+08:00
tags:
- 算法
- leetcode
Categories:
- 递归
- 树
---

### **题目描述**

> 给定一个二叉树，它的每个结点都存放一个 0-9 的数字，每条从根到叶子节点的路径都代表一个数字。
>
> 例如，从根到叶子节点路径 1->2->3 代表数字 123。
>
> 计算从根到叶子节点生成的所有数字之和。
>
> 说明: 叶子节点是指没有子节点的节点。
>

### **题目链接**

> https://leetcode-cn.com/problems/sum-root-to-leaf-numbers
>

### **思路**

> 根节点到叶子节点的所有路径，可以想到用DFS
>
> - 从根开始遍历，每次向下一个节点遍历的时候就将`之前遍历的所有节点的值*10`，然后`加上当前节点值`
> - 最终`到达叶子节点`的时候即为`这条分支的值`
> - `递归遍历左右子树`，将左右子树的值相加
>
> 注意两个边界情况：
>
> - 当`节点为 null`的时候，返回0
> - 当`节点无左右子树`的时候，即为叶节点的时候，返回当前已经计算的值sum

### **代码**

```c++
class Solution {
public:

    int dfs(TreeNode* root,int preSum){
        if(root == nullptr) return 0;
        int sum = preSum * 10 + root -> val;
        if(root -> left == nullptr && root -> right == nullptr) return sum;
        return dfs(root -> left,sum) + dfs(root -> right,sum);
    }

    int sumNumbers(TreeNode* root) {
        int sum =  dfs(root,0);
        return sum;
    }
};
```

### **复杂度分析**

> 时间复杂度为O(n)，空间复杂度为O(logN)