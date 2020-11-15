---
title: "Leetcode24 两两交换链表中的节点"
date: 2020-11-11T14:58:23+08:00
tags:
- 算法
- leetcode
Categories:
- 链表
---

### **题目描述**

给定一个链表，两两交换其中相邻的节点，并返回交换后的链表。

你不能只是单纯的改变节点内部的值，而是需要实际的进行节点交换。

- 例子：

  输入：head = [1,2,3,4]

  输出：[2,1,4,3]

  输入：head = []

  输出：[]

  输入：head = [1]

  输出：[1]

>
> 提示：
>
> 链表中节点的数目在范围 [0, 100] 内
> 0 <= Node.val <= 100

### **题目链接**

> https://leetcode-cn.com/problems/swap-nodes-in-pairs/

### **思路一**

> `迭代`：要求链表中的节点两两交换，为了使所有节点的操作一致
>
> - 需要添加一个`虚拟头节点dummpy`，然后设置两个指针`pre`和`cur`，分别指向`dumpy`和`head`，这两个指针的作用就是将`head`节点和`node1`节点进行交换。如图，这两个节点的交换涉及到三个节点`head`、`node1`、`cnn`，只需要改变`head`和`node1`的指针指向即可（但是需要注意的是单链表只有一个指针，在`head`断开指向`node1`的指针之前，需要先用一个指针指向`node1`，以便后面`node1`的操作，防止出现`断链`）
>
> - `head`指向后继的后继
>
> - `head`的后继`node1`断开指向`cnn`的指针，转而指向前驱`head`
>
> - 将`dummpy`指向`node1`（这一步不要忘记！！！）
>
>   这样，`head`和`node1`的交换就完成了
>
> - 然后将`pre`重新指向`node`，将`cur`重新指向`cnn`
>
> - 最终返回的是`dummpy`头节点，即一条链
>
> 所以可以看出来，`pre`和`cur`指针的作用就是将`cur`所指向的当前节点和其下一个节点进行交换

![img](https://cdn.jsdelivr.net/gh/VegeBun-csj/Images/{B0EF5083-1460-9B97-2030-913B0631F3CD}.png)

> 在链表的操作中，经常会出现`头节点`和`头指针`，很容易搞乱，它们的区别就是：`头指针`是链表中`必须有`的，因为它是指向链表的第一个节点的，如果一个链表中没有头节点，那么此时链表的第一个节点就是首节点head;`头节点不是必须`的，是因为**为了使链表的第一个节点和其他节点操作一致**，而引入了一个头节点dummpy，当一个链表有头节点的时候，链表的第一个节点是头节点dummpy，而第二个节点才是首节点head

### **代码(迭代)**

```c++
class Solution {
public:
    ListNode* swapPairs(ListNode* head) {
       if(head==nullptr || head->next ==nullptr) return head;
        
        //开辟一个头节点dummpy
        ListNode* dummpy = new ListNode(0);
        dummpy -> next = head;
        //创建两个指针，pre和cur
        ListNode *pre = dummpy,*cur = head;
        while(cur != nullptr && cur -> next != nullptr){
            //此时的序列为pre cur cur->next cnext-next......
            //记录cur的后继节点，因为后面cur要断开与后继节点的链接
            ListNode* cnext = cur -> next;
            cur -> next = cnext -> next;
            cnext -> next = cur;
            pre -> next = cnext;
            //交换之后的序列为pre cur->next cur cnext->next
            //交换一对节点之后，再将pre和cur重新指向新的节点
            //即pre重新指向cur,cur指向cur下一个节点（可以看出pre和cur两个指针是一起的，它们的作用就是将cur所指的当前的节点与cur的下一个节点进行交换）
            pre = cur;
            cur = cur -> next;
        }
		//最终返回的是dummpy
        return dummpy -> next;
    }
};
```

### **思路二**

> `递归`：写法更简洁，可以通过递归的思路去理解，首先递归的思路分为`三步骤`：
>
> - 递归`边界`
> - 递归当前层`做了什么`
> - 递归的`返回值`
>
> 本题中递归的边界就是当递归到节点的指针为null的时候；每一层递归需要做的就是交换，即`当前节点head`,下一个节点为`head->next`，和`已经交换过的链表部分`三部分，每次需要做的就是交换head和head->next；递归的返回值为当前已经处理完的链表（如果将p为head->next，那么返回的就应该是p，即p此时为第一个节点，代表了链表）
>
> - 递归切记不要陷入`递归地狱`，哈哈哈这个名字我自己起的，因为之前看Node里有一个回调地狱🤭，切记不要一层一层下去深究，会把自己绕晕的，只需要考虑本层的事情即可，其他层的道理其实一样
> - 操作的注意点也是链表交换防止`断链`，要将后面的节点用一个指针来指向，然后将两个节点进行交换

![img](https://cdn.jsdelivr.net/gh/VegeBun-csj/Images/{6B0FACB6-C0C6-312A-5EF8-CCF145B36819}.png)

### **代码(递归)**

```c++
class Solution {
public:
    ListNode* swapPairs(ListNode* head) {
        if(head==nullptr || head->next ==nullptr) return head;
        ListNode* p = head->next;
        head->next = swapPairs(p->next);
        p->next = head;
        return p;
    }
};
```

### **复杂度分析**

> 时间复杂度为O(n),空间复杂度为O(1)