---
title: "Leetcode01 两数之和"
date: 2020-11-13T14:36:06+08:00
tags:
- 算法
- leetcode
Categories:
- hash表
---

### **题目描述**

给定一个整数数组 *nums* 和一个目标值 *target*，请你在该数组中找出和为目标值的那 两个 整数，并返回他们的数组下标。

你可以假设每种输入只会对应一个答案。但是，数组中同一个元素不能使用两遍。

示例:

> 给定 nums = [2, 7, 11, 15], target = 9
>
> 因为 nums[0] + nums[1] = 2 + 7 = 9
> 所以返回 [0, 1]

### **题目链接**

> https://leetcode-cn.com/problems/two-sum/

###  **思路一**

> 第一种：`暴力解法`，利用`双指针`，直接循环两次遍历数组

### **代码一**

```c++
class Solution {
public:
    vector<int> twoSum(vector<int>& nums, int target) {
        int i,j;
        for(i=0;i<nums.size()-1;i++)
        {
            for(j=i+1;j<nums.size();j++)
            {
                if(nums[i]+nums[j]==target)
                {
                   return {i,j};
                }
            }
        }
        return {i,j};
    }
};
```

### **思路二**

> 第二种：**`hash表`，将`数组中的值`作为hash表的`key`，将这个值的下标作为hash表的`value`**。遍历数组，比如当前遍历的元素为`a`，然后在hash表中寻找是否有符合`target-a`的`key1`，如果没有，就将当前遍历的元素`a`及`其下标`插入hash表中，如果有，就**返回key1对应的值(也就是`key1的下标`)和当前遍历的元素的下标**

![img](https://cdn.jsdelivr.net/gh/VegeBun-csj/Images/{97C6B69A-8F17-96A9-F70A-E6E944EAC366}.png)

### **代码二**

```c++
class Solution {
public:
    vector<int> twoSum(vector<int>& nums, int target) {
        unordered_map<int,int> nums_map;
        int i;
        for(i = 0;i < nums.size();i ++){
            if(nums_map.count(target - nums[i]) != 0) return {nums_map[target - nums[i]],i};
            else nums_map.insert(pair<int,int>(nums[i],i));
        }
        return {nums_map[i],i};
    }
};
```

### **复杂度分析**

> 第一种暴力解法：时间复杂度为O(n^2)，空间复杂度为O(1)
>
> 第二种hash表：时间复杂度为O(n)，空间复杂度为O(n)