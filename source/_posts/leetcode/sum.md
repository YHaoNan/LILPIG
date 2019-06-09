---
title: Leetcode - Two Sum
date: 2019-05-20 18:24:27
tags: [算法,leetcode]
categories: leetcode
---
### 描述
题目：  
>Given an array of integers, return indices of the two numbers such that they add up to a specific target.  
You may assume that each input would have exactly one solution, and you may not use the same element twice.

难度：Easy

示例：
```
Given nums = [2, 7, 11, 15], target = 9,

Because nums[0] + nums[1] = 2 + 7 = 9,
return [0, 1].
```

### 想法
如果按照笨方法做，那么每一个数得循环一次，最起码是指数级别的时间复杂度。我想用常数级别的线性时间内完成，于是想到hashmap。

对于nums中的每个数，将它作为键，在数组中的下标作为值存储在Hashmap中，然后遍历一遍，从Hashmap中找是否存在`target-nums[i]`这个数，如果有，这个数和`nums[i]`相加就是`target`，直接返回。

### 实现
Java
```java
class Solution {
    public int[] twoSum(int[] nums, int target) {
        Map<Integer,Integer> bucket = new HashMap();
        for(int i=0;i<nums.length;i++){
            int lackNum = target-nums[i];
            if(bucket.get(lackNum)!=null)
                return new int[]{bucket.get(lackNum),i};
            else
                bucket.put(nums[i],i);
        }
        return null;
    }
}
```
Python3
```python
class Solution:
    def twoSum(self, nums: List[int], target: int) -> List[int]:
        bucket = {}
        for i,num in enumerate(nums):
            try:
                index = bucket[target-num]
                return [index,i]
            except:
                pass
            finally:
                bucket[num]=i
```

```
Java版本运行结果：
Runtime: 2 ms, faster than 99.33% of Java online submissions for Two Sum.
Memory Usage: 37.2 MB, less than 98.48% of Java online submissions for Two Sum.

Python版本运行结果：
Runtime: 28 ms, faster than 99.92% of Python3 online submissions for Two Sum.
Memory Usage: 14.4 MB, less than 31.65% of Python3 online submissions for Two Sum.
```
