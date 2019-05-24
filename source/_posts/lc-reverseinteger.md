---
title: Leetcode - Reverse Integer
date: 2019-05-24 17:36:27
tags: [算法,leetcode]
categories: leetcode
---
### 闲白
昨天那题给我干懵逼了。今天直接来个简单的。

### 描述
>Given a 32-bit signed integer, reverse digits of an integer.

难度：easy

示例：
```
Input: 123
Output: 321

Input: -123
Output: -321

Input: 120
Output: 21
```

注意：
>Note:   
>Assume we are dealing with an environment which could only store integers within the 32-bit signed integer range: [−2^31,  2^31 − 1]. For the purpose of this problem, assume that your function returns 0 when the reversed integer overflows.

### 分析
没啥好分析的，倒过来还不简单，就是要注意正负和int类型最大值反过来可能越界的问题。

有难度的地方就是负数和最大值，我把它们都当做正数处理就好了。

### 实现
```java
class Solution {
    public int reverse(int x) {
        if (x==Integer.MAX_VALUE||x==Integer.MIN_VALUE)
            return 0;
        boolean isNegative = x<0;

        x = Math.abs(x);

        int result=0;
        while (x!=0){
            if (result>Integer.MAX_VALUE/10)
                return 0;
            result*=10;
            result+=(x%10);
            x/=10;
        }
        result*=isNegative?-1:1;
        return result;
    }
}
```

结果：
```
Runtime: 1 ms, faster than 100.00% of Java online submissions for Reverse Integer.
Memory Usage: 32.7 MB, less than 67.28% of Java online submissions for Reverse Integer.
```