---
title: "Leetcode206 反转链表"
date: 2020-11-12T14:58:23+16:00
tags:
- 算法
- leetcode
Categories:
- 链表
---

### **题目描述**

反转一个单链表。

**示例:**

```
输入: 1->2->3->4->5->NULL
输出: 5->4->3->2->1->NULL
```

### **题目链接**

> https://leetcode-cn.com/problems/reverse-linked-list/
>

### **思路一**

> `迭代`：使用双指针，同样为了操作的一致性，需要创建一个虚拟节点`dummpy`，`原地反转`链表
>
> 一个指向头节点的指针`prev`，一个指向pre->next的指针`cur；`每次要做的就是将`pre`的指针指向`cur的下一个节点`，然后将`cur`指向`dummpy的下一个节点`，最后再将`dummpy`指向`cur的下一个节点`。
>
> 主要分为四步(以step1为例)：
>
> - pre指向的节点head断开指向后继的指针，转而指向后继的后继node1节点
> - cur指向的节点断开指向后继的指针，转而指向前驱head节点
> - dummpy虚拟节点断开其指向后继head的指针，转而指向node1节点
> - 将cur指针重置为指向pre指针所指节点的下一个节点node2
>
> 重复上述步骤，直到cur指向为null的时候，即下图的step4，此时就已经反转结束了
>
>
> 整个循环不停反转过程中`pre`一直是指向`head`节点的，而`cur`是不断变化的，不断更新成为	为dummpy的下一个节点，最终原来的链表尾成为dummpy指向的节点，即首节点，而原来的首节点`head`成为尾节点。

![img](https://cdn.jsdelivr.net/gh/VegeBun-csj/Images/{F7632B14-6615-88E4-7BB7-476545E0B778}.png)

### **代码**

```c++
class Solution {
public:
    ListNode* reverseList(ListNode* head) {
        if(head == nullptr || head -> next == nullptr) return head;
        //新建一个指向首节点的虚拟节点
        ListNode* dummpy = new ListNode(0);
        dummpy -> next = head;
        //设置两个指针，分别指向头节点和其下一个节点
        ListNode* prev = head;
        ListNode* cur = prev -> next;
        while(cur != nullptr){
            //反转dummpy的下一个节点和cur当前指向的节点
            prev -> next = cur -> next;
            cur -> next = dummpy -> next;
            dummpy -> next = cur;
            //pre不变，cur指向下一个pre的下一个节点
            cur = prev -> next;
        }
        return dummpy -> next;
    }
};
```



### **思路二**

> `递归`
>
> - 递归边界：当递归到最后为null时
> - 递归每层干的事：将当前节点指向当前节点的下下个节点，并将当前节点的下一个节点指向当前节点。
> - 递归返回值：返回的是已经变为倒序的链表

### **代码**

```c++
class Solution {
public:
    ListNode* reverseList(ListNode* head) {
        if(head == nullptr || head -> next == nullptr) return head;
        ListNode* p = reverseList(head ->next);
        head -> next -> next = head;
        head ->next = nullptr;
        return p;
    }
};
```

### **复杂度分析**

> 时间复杂度为O(n)，空间复杂度为O(1)