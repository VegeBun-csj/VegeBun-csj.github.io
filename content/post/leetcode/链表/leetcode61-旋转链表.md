---
title: "Leetcode61 旋转链表"
date: 2020-11-11T15:01:50+08:00
tags:
- 算法
- leetcode
Categories:
- 链表
---

### **题目描述**

给定一个链表，旋转链表，将链表每个节点向右移动 k 个位置，其中 k 是非负数。

示例 1:

> 输入: 1->2->3->4->5->NULL, k = 2
>
> 输出: 4->5->1->2->3->NULL

解释:

> 向右旋转 1 步: 5->1->2->3->4->NULL
>
> 向右旋转 2 步: 4->5->1->2->3->NULL

示例 2:

> 输入: 0->1->2->NULL, k = 4
>
> 输出: 2->0->1->NULL

解释:

> 向右旋转 1 步: 2->0->1->NULL
>
> 向右旋转 2 步: 1->2->0->NULL
>
> 向右旋转 3 步: 0->1->2->NULL
>
> 向右旋转 4 步: 2->0->1->NULL

### **题目链接**

> https://leetcode-cn.com/problems/rotate-list/

### **思路**

> **旋转实际上就是确定新的链表头和链表尾**；如果说将一个链表每个节点向右移动k，那么新的链表的表头节点就是从后往前数第k个节点(如果是k>n的话就是k%n)。在单链表中，只有next指针，所以，如果要找到新链表的表头节点，我们就需要找到它的前一个节点（即新链表的表尾节点），新链表的表尾节点从左往右数第`n-k`个(如果k>n，那么就是`n - k%n)；但是从头开始找的话只需要移动`n - k % n -1 `次就能找到新的表尾节点。
>
> 其中还涉及到一个就是`计算链表节点个数`，就是普通的遍历，当遍历到最后的时候，我们不仅能得到链表的节点数目，还能得到旧链表的表尾节点。
>
> 对于第一种代码的写法(官方写的)：就是在找到表尾结点之后，将表尾节点指向表头节点，形成一个闭环；然后从头开始遍历寻找新的表尾节点，再得到新的表头节点，然后将表头和表尾断开即可
>
> 对于第二种代码的写法：不使用所谓的链表闭环，只是记录旧节点的位置，当最后找到新的链表头和尾的时候，将其接到旧链表的表头
>
> 注意一个判断条件：`k % n`是否为 0 ，如果为0那么不需要进行移动，因为还是移动的结果还是本身，直接发返回即可。
>
> 所以综合来看，并不需要什么所谓的链表的环，所要得到的本质就是两个：确定**新的表头和新的表尾**(先得到新表尾，再得到新的表头)

![img](https://cdn.jsdelivr.net/gh/VegeBun-csj/Images/{F840FAF1-A1DC-58EA-E5CE-5E27877337C4}.png)

### **代码**

```c++
class Solution {
public:
    ListNode* rotateRight(ListNode* head, int k) {
        if(head == nullptr || head -> next == nullptr ) return head;

        ListNode* oldNode = head;
        int n;  //用来记录链表长度
        for(n = 1;oldNode->next!=nullptr;n ++){     //一开始就是从第一个节点开始计数
            oldNode = oldNode->next;
        }
        //如果k % n 为0，那么就不需要进行移动
        if(k % n == 0) return head;
        
        //同时获取到旧链表的最后一个节点
        oldNode->next = head;
        //此时已经形成了闭环

        //接下来就找新的表尾
        ListNode* newNode = head;
        for(int i = 0;i < n - k % n - 1;i ++){     //找到第n-k%n-1的位置的元素，即为链表尾
            newNode = newNode->next;
        }
        //新的表头
        ListNode* newHead = newNode->next;
        
        //断开环(表尾置空)
        newNode->next = nullptr;
        return newHead;

    // 如果k为2，也就是说从队尾开始找到第2个节点，题目中测试用例即4就会称为新的list的head

    }
};
```



另一种代码类似写法

```c++
class Solution {
public:
    ListNode* rotateRight(ListNode* head, int k) {
        if(head == nullptr || head -> next == nullptr ) return head;
        
        ListNode* oldNode = head;
        int n;
        for(n = 1;oldNode -> next != nullptr;n ++) oldNode = oldNode -> next;
        //同时这里的oldNode为旧链表的尾节点

        if(k % n == 0) return head;
        //寻找新的尾节点
        ListNode* newTail = head;
        for(int i = 0;i < n - k % n -1;i ++)    newTail = newTail -> next;
        
        //找到新的头节点
        ListNode* newHead = newTail -> next; 
        //新的尾节点的next置空
        newTail -> next = nullptr;

        //将老的尾节点接到老的头节点，形成一个链表
        oldNode -> next = head;

        return newHead;
    }
};
```

### **复杂度分析**

> 时间复杂度为O(n)，空间复杂度为O(1)