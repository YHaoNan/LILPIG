---
title: Leetcode - Add Two Number
date: 2019-05-21 07:36:27
tags: [算法,leetcode]
categories: leetcode
---
### 描述
题目：  
>You are given two non-empty linked lists representing two non-negative integers. The digits are stored in reverse order and each of their nodes contain a single digit. Add the two numbers and return it as a linked list.  
You may assume the two numbers do not contain any leading zero, except the number 0 itself.

难度：Medium

示例：
```
Input: (2 -> 4 -> 3) + (5 -> 6 -> 4)
Output: 7 -> 0 -> 8
Explanation: 342 + 465 = 807.
```

链表的JavaAPI
```java
链表的Java API：

 public class ListNode {
    int val;
    ListNode next;
    ListNode(int x) { val = x; }
 }
 */
```
### 想法
由于这个步骤有些多，所以每一步用伪代码实现一下。

首先不考虑进位，不考虑两个链表长度不同，循环读出链表的值，并且相加成为一个新的链表。
```
f(l1,l2)
    r = null
    while(l1!=null)
        plus = l1.val + l2.val
        if(r == null)
            r = ListNode(plus)
        else
            r.next = ListNode(plus)
            r = result.next
        l1 = l1.next
        l2 = l2.next
```

我们实现了一个基本模型，现在考虑下进位问题。因为每位肯定是不可能超过10的正整数(包括0)，那么就算9和9相加都不到20，所以，进的位肯定是1或者0，而不可能是2。

所以我们并不用一个数来存储进位，直接用一个布尔变量记录上一次相加是否需要进位即可。
```
...
    while(l1!=null)
        plus = l1.val + l2.val + (n?1:0) # 这里加上了一个判断，如果需要进位就加1
        ...
            r = result.next
        
        # 下面判断是否进位，并做好标记。
        if(r.val>9)
            r.val %= 10 # 取个位数
            n = true
        else
            n = false

        l1 = l1.next
        l2 = l2.next
```
进位的问题解决了，我们再考虑下链表长度不同的问题。我们现在依靠l1的长度做这件事，也就是说如果l2比l1长，那么后面的数不会加到，如果比l1短，会引起异常(Null Pointer)。

那么我们把循环条件放松一些，如果l1和l2中有一个不等于null就行。也就是只要两个中有一个还有要加的数就让它继续循环。不过我们还需要多加一些判断的代码保证不会出异常。

```
...
    while(l1!=null || l2!=null)
        plus = (l1!=null?l1.val:0) + (l2!=null?l2.val:0) + (n?1:0)
        ...
            r = result.next
        
        # 下面判断是否进位，并做好标记。
        if(r.val>9)
            r.val %= 10 # 取个位数
            n = true
        else
            n = false

        l1 = (l1!=null?l1.next:null)
        l2 = (l2!=null?l2.next:null)
```
还有一个终极问题就是，如果最后一位需要进位怎么办？比如：
```
输入[5 -> 2 -> 1] [1 -> 2 -> 9]

输出[6 -> 4 -> 0 -> 1]
```
所以还要在后面判断一下。
```
...
        l2 = (l2!=null?l2.next:null)

if(n)
    r.next = ListNode(1)
```
这就万无一失了。
### 实现
Java
```Java
/**
* 时间复杂度 O(n)
* 空间复杂度 O(n)
*
*/
class Solution {
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        boolean isEqualsBiggerThanTen = false;
        
        ListNode result = null;
        ListNode head = null;
        
        while(l1!=null || l2!=null){
            int plus = (l1!=null?l1.val:0) + (l2!=null?l2.val:0) + (isEqualsBiggerThanTen?1:0);
            
            if(result == null){
                result = new ListNode(plus);
                head = result;
            }else{
                result.next = new ListNode(plus);
                result = result.next;
            }
            
            if(result.val > 9){
                result.val %= 10;
                isEqualsBiggerThanTen = true;
            }else
                isEqualsBiggerThanTen = false;
            l1 = (l1!=null?l1.next:null);
            l2 = (l2!=null?l2.next:null);
        }
        
        if(isEqualsBiggerThanTen){
            result.next = new ListNode(1);
        }
        return head;
    }
}
```
结果
```
Runtime: 2 ms, faster than 94.88% of Java online submissions for Add Two Numbers.
Memory Usage: 42.8 MB, less than 88.82% of Java online submissions for Add Two Numbers.
```