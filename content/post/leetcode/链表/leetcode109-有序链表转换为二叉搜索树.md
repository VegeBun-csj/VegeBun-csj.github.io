---
title: "Leetcode109 有序链表转换为二叉搜索树"
date: 2020-11-11T15:03:56+08:00
tags:
- 算法
- leetcode
Categories:
- 链表
---

### **题目链接**

> https://leetcode-cn.com/problems/convert-sorted-list-to-binary-search-tree/

### **思路**

> 将一个有序的链表转换为二叉搜索树，并且还要求，这个二叉搜索树为平衡二叉树，所以为了保持平衡，选取链表的中间节点为树的根节点，然后递归操作链表的左右区间为树的左右子树。这里有一个关键点就是如何得到链表的中间节点：可以通过快慢指针来得到，设一个快指针，一个慢指针，它们初始都是从头节点开始，但是快指针一次走两步，满指针一次走一步，最终等到快指针走到底的时候，满指针所指向的就是链表中间节点。然后以中间节点为界，划分除左右区间，再递归解决。

![img](https://cdn.jsdelivr.net/gh/VegeBun-csj/Images/{332105E2-16C7-684F-24D0-B0BC2D4168AD}.png)

### **代码**

```c++
class Solution {
public:
    TreeNode* sortedListToBST(ListNode* head) {
        if(head == nullptr) return nullptr;
        //如果只有一个节点，那么返回其值
        if(head -> next == nullptr) return new TreeNode(head -> val);

        //寻找链表的中间节点(快慢指针)
        ListNode* slow = head,*fast = head,*preslow = nullptr;
        while(fast != nullptr && fast->next != nullptr){
            preslow = slow;
            slow = slow -> next;
            fast = fast -> next -> next;
        }

        //当快指针走到头时，此时慢指针就是中间节点,此时就将中点的左右两边断开
        preslow -> next = nullptr;      
        TreeNode* root = new TreeNode(slow->val);       //创建树的根节点，即为中间节点值
        root -> left = sortedListToBST(head);
        root -> right = sortedListToBST(slow->next);
        return root;
    }
};
```

### **复杂度分析**

> 时间复杂度为O(nlogn)，空间复杂度为O(logn)