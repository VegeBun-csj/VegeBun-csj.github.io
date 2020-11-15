---
title: "Leetcode821 字符最短距离"
date: 2020-11-10T21:43:08+08:00
tags:
- 算法
- leetcode
Categories:
- 数组队列栈
---

### **题目链接**

> https://leetcode-cn.com/problems/shortest-distance-to-a-character/

### **思路**

> 左右两边进行遍历，分别找字符串中每个字符距离左边C最近的距离和距离右边C最近的距离，然后取两者中的较小者

### **代码**

```c++
class Solution {
public:
    vector<int> shortestToChar(string S, char C) {
        vector<int> distance(S.size(),0);
        //pivot取得是很小的值，因为是被减的，如果没找到，距离就会很大
        int pivot = INT_MIN / 2;		//INT_MIN是一个非常小的数
        //从左向右遍历，寻找S中每一个字符左边距离C的最近距离，即i - pivot
        for(int i = 0;i < S.size();i ++){
            if(S[i] == C) pivot = i;
            //如果没找到C，距离就很大
            distance[i] = i - pivot;
        }
        //从右往左遍历，寻找S中每一个字符右边距离C的最近距离，即pivot - i
        //pivot取得是最大值，因为后面是pivot - i，如果没找到距离就特别大
        pivot = INT_MAX / 2;		//INT_MAX是一个非常大的数	
        for(int i = S.size() - 1;i >= 0;i --){
            if(S[i] == C) pivot = i;
            //取两边距离的较小者
            distance[i] = min(distance[i],pivot - i);
        }
        return distance;

    }
};
```

### **复杂度分析**

> 时间复杂度为O(n),空间复杂度为O(n)
>
> 执行用时：0 ms, 在所有 C++ 提交中击败了100.00%的用户
>
> 内存消耗：7.3 MB, 在所有 C++ 提交中击败了5.04%的用户