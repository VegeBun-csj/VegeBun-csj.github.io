---
title: "Leetcode142 环形链表II"
date: 2020-11-11T15:08:34+08:00
tags:
- 算法
- leetcode
Categories:
- 链表
---

### **题目链接**

> https://leetcode-cn.com/problems/linked-list-cycle-ii/

### **思路**

> 使用双指针，一个快指针，一个慢指针，需要进行数学推导证明，记住这个规律即可：设置一个快指针，一个慢指针，都一开始从链表head开始，快指针一次走两步，慢指针一次走1步，当它们第一次相遇的时候，将快指针重新指向head，然后和慢指针继续一次走一步，当他们再次相遇的时候，所指向的即为环的入口点
>
> 本题还有一个注意点就是在两个指针移动的过程中，要用快指针来判断是否指向为空，只要快指针的next或者next->next指向为空，就说明没有环，即退出；用快指针进行判断的原因就是它走的比较快，适合`“探路”`，而慢指针就不行，因为它只能判断next是否为空，而不能判断next->next是否为空

![img](https://cdn.jsdelivr.net/gh/VegeBun-csj/Images/{F8EEE54D-47A1-239D-6D27-BACDA2BC4C37}.png)

### **代码**

```c++
class Solution {
public:
    ListNode *detectCycle(ListNode *head) {
        if(head == nullptr || head -> next == nullptr) return nullptr;
        ListNode* ptr1 = head;
        ListNode* ptr2 = head;
        do{
            //ptr2必须进行判断，因为如果ptr2指向出现了空，或者它的下下个为空，说明不存在环
            if(ptr2 -> next != nullptr && ptr2 -> next -> next != nullptr){
                ptr2 = ptr2 -> next -> next;
            }else{
                return nullptr;
            }
            ptr1 = ptr1 -> next;
        }while(ptr1 != ptr2);

        ptr2 = head;
        while(ptr1 != ptr2){
            ptr2 = ptr2 -> next;
            ptr1 = ptr1 -> next;
        }
        return ptr1;
    }
};
```

### **复杂度分析**

> 时间复杂度为O(n)，空间复杂度为O(1)