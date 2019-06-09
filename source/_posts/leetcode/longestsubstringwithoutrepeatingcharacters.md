---
title: Leetcode - Longest Substring Without Repeating Characters
date: 2019-05-22 18:52:27
tags: [算法,leetcode]
categories: leetcode
---
### 描述
>Given a string, find the length of the longest substring without repeating characters.

难度：Medium

示例：
```
Input: "abcabcbb"
Output: 3 
Explanation: The answer is "abc", with the length of 3. 

Input: "bbbbb"
Output: 1
Explanation: The answer is "b", with the length of 1.

Input: "pwwkew"
Output: 3
Explanation: The answer is "wke", with the length of 3. 
             Note that the answer must be a substring, "pwke" is a subsequence and not a substring.
```

### 想法
老实说，我的想法只有暴力破解，我又没敢写，想了好久也没想到，最后看了一个大佬写的东西。

算法中有一个东西叫滑动窗口，这个东西可以把很多复杂问题的时间复杂度搞到线性。

这个问题的解就用到了滑动窗口，看看它是咋工作的。

1. 初始化一个滑动窗口，有一个左值有一个右值(l和r)，默认它们是0,0
2. 遍历字符串中的每一个字符，r始终是当前遍历的字符所在的位置
3. 如果没发现重复的字符，l保持不动，如果发现了，l跳到该字符之前出现的位置的后一位
4. 重复2,3步，并且每次取窗口大小`r-l+1`，用每次的窗口大小确定最大长度

如果不懂就看[图解](https://blog.csdn.net/qq_40416052/article/details/82815116)

### 实现
```java
class Solution {
    public int lengthOfLongestSubstring(String s) {
        HashMap<Character,Integer> map = new HashMap<Character,Integer>();
        char[] chars = s.toCharArray();
        int maxLength = 0;
        for(int l=0,r=0;r<chars.length;r++){
            if (map.containsKey(chars[r])) l = Math.max(map.get(chars[r])+1,l);
            maxLength = Math.max(maxLength,r-l+1);
            map.put(chars[r],r);
        }
        return maxLength;
    }
}
```

结果：
```
Runtime: 6 ms, faster than 91.80% of Java online submissions for Longest Substring Without Repeating Characters.
Memory Usage: 35.3 MB, less than 99.98% of Java online submissions for Longest Substring Without Repeating Characters.
```