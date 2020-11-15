---
title: "Leetcode160 相交链表"
date: 2020-11-11T15:05:15+08:00
tags:
- 算法
- leetcode
Categories:
- 链表
---

### 题目链接

> https://leetcode-cn.com/problems/intersection-of-two-linked-lists/

### 思路

> 寻找相交的点。本题的解法非常巧妙，采用的双指针，即链表A一个指针ptr1进行遍历，链表B一个指针ptr2进行遍历； 两者一开始都是从各自的链表头部开始遍历（对于大多数情况，可能存在链表长度不一致的情况），当其中一个链表遍历结束，我们假设ptr1先遍历完了链表A，然后下一步将ptr1指向链表B，进行遍历；当ptr2遍历完链表B之后，将ptr2指向链表A，从头开始遍历。这样“轮着遍历”的精妙之处在于：当有相交的节点时，两个指针会同时指向该相交的节点(即它们在相交节点之前走过的路程是一样的)，此时就返回相交节点；当没有相交节点时，两个指针会将两个链表遍历完成，都为nullptr，表明两个链表没有相同的节点，最终返回空（所以这种方法的妙处在于，不管是走到相交的节点处还是遍历完两个链表之后没有相交节点，两个指针走过的路程都是一样的，并且它们最后退出遍历的条件都是相等的情况，即都指向相交节点或者都是nullptr）

![img](https://cdn.jsdelivr.net/gh/VegeBun-csj/Images/{D4EED0FC-6ABC-1EB8-BCCE-3C73619AC34D}.png)



![img](https://cdn.jsdelivr.net/gh/VegeBun-csj/Images/{F46E9F12-1B61-E417-6A12-796F5D52C4A1}.png)

### 代码

```c++
class Solution {
public:
    ListNode *getIntersectionNode(ListNode *headA, ListNode *headB) {
        if(headA == nullptr || headB == nullptr) return nullptr;
        ListNode* ptr1 = headA,*ptr2 = headB;

        //如果相交，那么ptr1和ptr2会在交点处都指向同一个节点，然后退出，此时ptr1和ptr2就是交点
        //如果不相交，两个指针都会遍历到最后，都指向nullptr的时候结束，此时就是返回Null
        while(ptr1 != ptr2){
            ptr1 = ptr1 == nullptr ? headB : ptr1 -> next;
            ptr2 = ptr2 == nullptr ? headA : ptr2 -> next;
        }
        return ptr1;
    }
};
```



### 复杂度分析

> 因为每个指针都要遍历两个链表，所以时间复杂度为O(n+m)，空间复杂度为O(1)