---
title: "Leetcode394 字符串解码"
date: 2020-11-10T21:43:48+08:00
tags:
- 算法
- leetcode
Categories:
- 数组队列栈
---



### **题目链接**

> https://leetcode-cn.com/problems/decode-string/

### **思路**

> 本题是括号匹配的问题，对于数字和字符都分别用一个栈来存放，当遇到`[`时，就将数字和字符串分别入栈，遇到`]`时就将数字和字符串出栈，并且对于此时的字符串栈的栈顶元素要循环加上此时保存的字符串，循环次数为数字栈的栈顶元素大小

### **代码**

```c++
class Solution {
public:
    string decodeString(string s) {
        stack<int> nums;
        stack<string> strs;
        int num = 0;
        string ret = "";
        for(int i = 0;i < s.size();i ++){
            if(s[i] >= '0' && s[i] <= '9'){
                num = num * 10 + s[i] - '0';
            }else if((s[i] >= 'a' && s[i] <= 'z') || (s[i] >= 'A' && s[i] <= 'Z')){
                ret += s[i];
            }else if(s[i] == '['){
                nums.push(num);
                num = 0;
                strs.push(ret);
                ret = "";
            }else if(s[i] == ']'){
                int times = nums.top();
                nums.pop();
                for(int i = 0;i < times;i ++){
                    strs.top() += ret;
                }
                ret = strs.top();
                strs.pop();
            }
        }

        return ret; 
    }
};
```

### **复杂度分析**

> 时间复杂度为：O(n) 空间复杂度为O(n)