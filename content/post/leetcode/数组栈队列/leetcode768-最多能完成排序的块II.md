---
title: "Leetcode768 最多能完成排序的块II"
date: 2020-11-10T21:44:32+08:00
tags:
- 算法
- leetcode
Categories:
- 数组队列栈
---

### **题目链接**

> https://leetcode-cn.com/problems/max-chunks-to-make-sorted-ii/

### **思路**

> 题目要求最大的分块数目，本身题目隐含的意思就是块内是可以是无序的，但是每个块整体上是递增的，也就是说每个块中的最大值是递增的，一个块中有一个最大值，即有几个极大值，就有几个块，也就是最终求有多少个这样的极大值。可以使用单调栈来解，执行流程为：当栈为空或者栈定元素小于即将入栈的元素时，将该元素入栈；否则就将栈中元素一一出栈与该元素比较，如果除了栈顶，有栈中元素大于即将入栈的元素，那么就将栈中元素出栈，因为即将进栈的元素比较小，必须要将其作为栈顶元素所在块的元素才行，否则不满足块的整体升序，即最终结果是一个单调递增栈

### **代码**

```c++
class Solution {
public:
    int maxChunksToSorted(vector<int>& arr) {
        stack<int> stk;
        for(int i = 0;i < arr.size();i ++){
            if(stk.empty() || stk.top() <= arr[i]){
                stk.push(arr[i]);
            }else{
                int temp = stk.top();
                while(!stk.empty() && stk.top() > arr[i]) {
                    stk.pop();
                }
                stk.push(temp);
            }
        }
        return stk.size();
    }
};
```

### **复杂度分析**

> 时间复杂度为O(n)，空间复杂度为O(n)