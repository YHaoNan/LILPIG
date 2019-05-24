---
title: Leetcode - Median of Two Sorted Arrays
date: 2019-05-22 18:52:27
tags: [算法,leetcode]
categories: leetcode
---

### 描述
>There are two sorted arrays nums1 and nums2 of size m and n respectively.  
>
>Find the median of the two sorted arrays. The overall run time complexity should be O(log (m+n)).
>
>You may assume nums1 and nums2 cannot be both empty.

难度：hard

示例：
```
nums1 = [1, 3]
nums2 = [2]

The median is 2.0


nums1 = [1, 2]
nums2 = [3, 4]

The median is (2 + 3)/2 = 2.5
```
### 想法
直接想到新开一个长度刚好能放进去给的两个数组的数组。然后因为所给数组有序，我们只需要保证它插入大数组时有序就行了，然后一个有序数组的中位数就非常好求了。
```java
class Solution {
    public double findMedianSortedArrays(int[] nums1, int[] nums2) {
        int[] arr = new int[nums1.length+nums2.length];

        int iof1 = 0 , iof2 = 0 , iofarr = 0;

        while(iof1<nums1.length && iof2<nums2.length){
            if(nums1[iof1]<nums2[iof2]) arr[iofarr++] = nums1[iof1++];
            else arr[iofarr++] = nums2[iof2++];
        }

        if(iof1<nums1.length)
            for(;iof1<nums1.length;iof1++)
                arr[iofarr++]=nums1[iof1];
        if(iof2<nums2.length)
            for(;iof2<nums2.length;iof2++)
                arr[iofarr++]=nums2[iof2];

        float result;

        int divTwo = arr.length/2;
        if (arr.length%2==0)
            result = (arr[divTwo] + arr[divTwo-1]) / 2.0f;
        else
            result = arr[divTwo];

        return result;
        
        
    }
}
```
结果：
```
Runtime: 2 ms, faster than 100.00% of Java online submissions for Median of Two Sorted Arrays.
Memory Usage: 45.7 MB, less than 95.07% of Java online submissions for Median of Two Sorted Arrays.
```
结果吓到我了，这可是个hard难度的题啊，被我直接超过了`100%`？不太可能吧。。。

回头再看看题目，哦woc，要求时间复杂度在`O(log(m+n))`，我这个时间复杂度是`O(max(m,n))`，虽然快于要求的，但是显然不符合题意。

我感觉这题对于我太难了，我先去补补课，这几天先做easy和medium的题吧，过一段再回来补这道题。

讲真的，我开创了追溯原点式学习法。。。啊哈哈哈😂