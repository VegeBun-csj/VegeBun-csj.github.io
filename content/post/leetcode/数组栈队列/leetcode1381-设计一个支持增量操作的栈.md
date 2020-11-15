---
title: "Leetcode1381 设计一个支持增量操作的栈"
date: 2020-11-10T21:43:33+08:00
tags:
- 算法
- leetcode
Categories:
- 数组队列栈
---



### **题目链接**

> https://leetcode-cn.com/problems/design-a-stack-with-increment-operation/

### **思路：**

> 采用数组进行模拟

### **代码(cpp)**

```c++
class CustomStack {
public:
    int top;
    vector<int> simu_stack;     //用数组来模拟栈

    CustomStack(int maxSize) {
        simu_stack.resize(maxSize);
        top = -1;
    }
    
    void push(int x) {
        if(top != simu_stack.size() - 1){
            top ++;
            simu_stack[top] = x;
        }

    }
    
    int pop() {
        if(top != -1){
            //先top--
            top --;
            //然后返回栈顶，所以栈顶应该是没有--时的，所以需要+1
            return simu_stack[top + 1];
        }
        return -1;
    }
    
    void increment(int k, int val) {
        int num = min(k,top + 1);      //top + 1是因为top是数组下标
        for(int i = 0;i < num;++i){
            simu_stack[i] += val;
        }
    }
};
```

### **复杂度分析：**

> 时间复杂度：入栈和出栈都是O(1)，增加元素值的操作复杂度为O(n)；空间复杂度为O(n)

上面这种做法有一点不太好的就是增加其中栈顶某些元素值的时候复杂度比较高

------

### **优化（利用前缀和的思想）**

> 优化的思路主要是出于，本题的要求是增加的是栈底连续几个元素的值，遇到这种问题，应该想到前缀和，通过一个值不断累加，所以对于增加栈底的几个元素值时，可以不用循环，而是新开辟一个数组inc，用来记录增加值的操作，比如：`increament(2,3)`即将栈底的两个元素都增加3，那么我们就将inc[2] = 3，inc中的索引记录的是增加的栈底元素个数，值为增加的值。这样我们将要增加的值放在inc数组中，等到要pop取的时候我们再计算栈顶元素的值，即`simu_stack[top] + inc[top]`，然后将`inc[top]`的值传递到下一个元素`inc[top - 1]` ，然后将`inc[top] = 0`（下一次再重新赋值），等待下一次取的时候再`simu_stack[top] + inc[top]`,这样就完成了一段连续元素都增加的操作
>
> 对于push操作不变，pop

### **代码：**

```c++
class CustomStack {
public:
    int top;
    vector<int> simu_stack;     //用数组来模拟栈
    vector<int> inc;            //记录增加的值
    CustomStack(int maxSize) {
        simu_stack.resize(maxSize);
        inc.resize(maxSize);
        top = -1;
    }
    
    void push(int x) {
        if(top != simu_stack.size() - 1){
            top ++;
            simu_stack[top] = x;
        }
    }
    
    int pop() {
        if(top == -1){
            return -1;
        }
        int res = simu_stack[top] + inc[top];
        if(top != 0){
            inc[top - 1] += inc[top]; 
        }
        inc[top] = 0;
        top --;
        return res;
    }
    
    void increment(int k, int val) {
        int num = min(k-1,top);          //k-1是因为inc数组是以下标0开始的
        if(num >= 0){
            inc[num] += val;
        }
    }
};
```

### **复杂度分析**

> 时间复杂度：所有的操作都是O(1)，空间复杂度为O(n)