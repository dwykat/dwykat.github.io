---
layout: post
title: "LeetCode每日一题题解集"
description: "LeetCode每日一题题解集"
categories: [Python, Code]
tags: [leetcode, python]
redirect_from:
  - /2018/09/18/
---
**目录：**
* Kramdown table of contents
{:toc .toc}

* * * 

# LeetCode 71: Simplify Path 
**如果程序有一些情况需要回退，那么优先就考虑一下能不能使用栈。**

```python
#
# [71] Simplify Path
#
# https://leetcode.com/problems/simplify-path/description/
#
# algorithms
# Medium (26.95%)
# Total Accepted:    124.7K
# Total Submissions: 461.5K
# Testcase Example:  '"/home/"'
#
# Given an absolute path for a file (Unix-style), simplify it. 
# 
# For example,
# path = "/home/", => "/home"
# path = "/a/./b/../../c/", => "/c"
# path = "/a/../../b/../c//.//", => "/c"
# path = "/a//b////c/d//././/..", => "/a/b/c"
# 
# In a UNIX-style file system, a period ('.') refers to the current directory,
# so it can be ignored in a simplified path. Additionally, a double period
# ("..") moves up a directory, so it cancels out whatever the last directory
# was. For more information, look here:
# https://en.wikipedia.org/wiki/Path_(computing)#Unix_style
# 
# Corner Cases:
# 
# 
# Did you consider the case where path = "/../"?
# In this case, you should return "/".
# Another corner case is the path might contain multiple slashes '/' together,
# such as "/home//foo/".
# In this case, you should ignore redundant slashes and return "/home/foo".
# 


class Solution:
    def simplifyPath(self, path):
        """
        :type path: str
        :rtype: str
        """

        path = path.split('/')
        stack = []
        jump = ('', '.', '..')
        for item in path:
            if stack and item == '..':
                stack.pop()
            elif item not in jump:
                stack.append(item)

        return '/' + '/'.join(stack)

```

# LeetCode 53: maximum subarray 
*****典型动规**

```python
#
# [53] Maximum Subarray
#
# https://leetcode.com/problems/maximum-subarray/description/
#
# algorithms
# Easy (41.14%)
# Total Accepted:    367K
# Total Submissions: 890.6K
# Testcase Example:  '[-2,1,-3,4,-1,2,1,-5,4]'
#
# Given an integer array nums, find the contiguous subarray (containing at
# least one number) which has the largest sum and return its sum.
# 
# Example:
# 
# 
# Input: [-2,1,-3,4,-1,2,1,-5,4],
# Output: 6
# Explanation: [4,-1,2,1] has the largest sum = 6.
# 
# 
# Follow up:
# 
# If you have figured out the O(n) solution, try coding another solution using
# the divide and conquer approach, which is more subtle.
# 
#


class Solution:
    def maxSubArray(self, nums):
        """
        :type nums: List[int]
        :rtype: int
        """
        length = len(nums)
        if length < 2:
            return sum(nums)

        max_res = pre_res = nums[0]
        for i in range(1, length):
                pre_res = nums[i] if pre_res <= 0 else nums[i] + pre_res
                max_res = max(max_res, pre_res)

        return max_res
```

# LeetCode 28: Implement strStr()
**KMP算法**
![KMP](http://oq782gkz3.bkt.clouddn.com/kmp.JPG)



```python
#
# [28] Implement strStr()
#
# https://leetcode.com/problems/implement-strstr/description/
#
# algorithms
# Easy (29.86%)
# Total Accepted:    316K
# Total Submissions: 1.1M
# Testcase Example:  '"hello"\n"ll"'
#
# Implement strStr().
# 
# Return the index of the first occurrence of needle in haystack, or -1 if
# needle is not part of haystack.
# 
# Example 1:
# 
# 
# Input: haystack = "hello", needle = "ll"
# Output: 2
# 
# 
# Example 2:
# 
# 
# Input: haystack = "aaaaa", needle = "bba"
# Output: -1
# 
# 
# Clarification:
# 
# What should we return when needle is an empty string? This is a great
# question to ask during an interview.
# 
# For the purpose of this problem, we will return 0 when needle is an empty
# string. This is consistent to C's strstr() and Java's indexOf().
# 
#
class Solution:
    def strStr(self, haystack, needle):
        """
        :type haystack: str
        :type needle: str
        :rtype: int
        """
        if not needle:
            return 0
        h_length, n_length = len(haystack), len(needle)
        if h_length < n_length:
            return -1

        next_arr = self.get_next_arr(needle)
        i = j = 0
        while i < h_length and j < n_length:
            if haystack[i] == needle[j]:
                i += 1
                j += 1
            elif next_arr[j] == -1:
                i += 1
            else:
                j = next_arr[j]

        return i - j if j == n_length else -1

    def get_next_arr(self, needle):
        length = len(needle)
        if length < 2:
            return [-1]

        next_arr = [0 for i in range(length)]
        next_arr[0], next_arr[1] = -1, 0
        i, cur = 2, 0
        while i < length:
            if needle[cur] == needle[i-1]:
                next_arr[i] = cur + 1
                cur = next_arr[i]
                i += 1
            elif cur > 0:
                cur = next_arr[cur]
            else:
                next_arr[i] = 0
                i += 1 

        return next_arr

```

**刚刚随手写kmp的时候发现，上面代码`get_next_arr`函数while循环里最后一个else分支里少些了`i += 1`，这样竟然也通过了leetcode的提交。醉了。顺便还试验了下，本来在`strStr`函数里，我是可以直接把needle的长度传给`get_next_arr`函数的，但后来发现，这样的话运行时间反而会加长。。只能说函数多传一个参数比在函数里多建一个变量用时更长啊。**

# LeetCode 7: Reverse Integer


```python
#
# [7] Reverse Integer
#
# https://leetcode.com/problems/reverse-integer/description/
#
# algorithms
# Easy (24.43%)
# Total Accepted:    480.9K
# Total Submissions: 2M
# Testcase Example:  '123'
#
# Given a 32-bit signed integer, reverse digits of an integer.
# 
# Example 1:
# Input: 123
# Output: 321
# 
# Example 2:
# Input: -123
# Output: -321
# 
# Example 3:
# Input: 120
# Output: 21
# 
# Note:
# Assume we are dealing with an environment which could only store integers
# within the 32-bit signed integer range: [−231,  231 − 1]. For the purpose of
# this problem, assume that your function returns 0 when the reversed integer
# overflows.
 

class Solution:
    def reverse(self, x):
        """
        :type x: int
        :rtype: int
        """
        ans = str(x)[::-1] if x >= 0 else '-' + str(x)[:0:-1]
        ans = int(ans)
        if ans < - 2 ** 31 or ans > 2 ** 31 - 1:
            ans = 0

        return ans
```


# LeetCode 15: 3Sum

**有些时候，如果列表内元素需要重复访问的话，可以将其赋给变量，这个题把`nums[left]`, `nums[middle]`, `nums[right]`赋给变量比直接使用多打败了20%的提交的人。：）**

```python
#
# [15] 3Sum
#
# https://leetcode.com/problems/3sum/description/
#
# algorithms
# Medium (22.03%)
# Total Accepted:    383.4K
# Total Submissions: 1.7M
# Testcase Example:  '[-1,0,1,2,-1,-4]'
#
# Given an array nums of n integers, are there elements a, b, c in nums such
# that a + b + c = 0? Find all unique triplets in the array which gives the sum
# of zero.
# 
# Note:
# 
# The solution set must not contain duplicate triplets.
# 
# Example:
# 
# 
# Given array nums = [-1, 0, 1, 2, -1, -4],
# 
# A solution set is:
# [
# ⁠ [-1, 0, 1],
# ⁠ [-1, -1, 2]
# ]
# 
# 
#
class Solution:
    def threeSum(self, nums):
        """
        :type nums: List[int]
        :rtype: List[List[int]]
        """
        length, res = len(nums), []
        
        if length < 3:
            return res 

        nums.sort()
        
        for left in range(length-2):
            a = nums[left]
            if a > 0:
                break
            if left > 0 and a == nums[left-1]:
                continue

            middle = left + 1
            right = length - 1
            while middle < right:
                b, c = nums[middle], nums[right]
                cur_sum = a + b + c
                if cur_sum == 0:
                    res.append([a, b, c])
                    middle += 1
                    right -= 1
                    while middle < right and nums[middle] == nums[middle-1]:
                        middle += 1
                    while right > middle and nums[right] == nums[right+1]:
                        right -= 1
                elif cur_sum < 0:
                    middle += 1 
                else:
                    right -= 1

        return res

```


# LeetCode 21: Merge Two Sorted Lists

**能用`if else`区分情况就不要用两个`if`，会慢。**

```python
#
# [21] Merge Two Sorted Lists
#
# https://leetcode.com/problems/merge-two-sorted-lists/description/
#
# algorithms
# Easy (42.96%)
# Total Accepted:    409.2K
# Total Submissions: 948.5K
# Testcase Example:  '[1,2,4]\n[1,3,4]'
#
# Merge two sorted linked lists and return it as a new list. The new list
# should be made by splicing together the nodes of the first two lists.
# 
# Example:
# 
# Input: 1->2->4, 1->3->4
# Output: 1->1->2->3->4->4
# 
# 
#
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, x):
#         self.val = x
#         self.next = None

class Solution:
    def mergeTwoLists(self, l1, l2):
        """
        :type l1: ListNode
        :type l2: ListNode
        :rtype: ListNode
        """
        if l1 is None:
            return l2
        elif l2 is None:
            return l1
        
        if l1.val <= l2.val:
            head = l1
            l1 = l1.next
        else:
            head = l2
            l2 = l2.next

        cur = head

        while l1 and l2:
            if l1.val <= l2.val:
                cur.next = l1
                l1 = l1.next
                cur = cur.next
            else:
                cur.next = l2
                l2 = l2.next
                cur = cur.next

        while l1:
            cur.next = l1
            l1 = l1.next
            cur = cur.next

        while l2:
            cur.next = l2
            l2 = l2.next
            cur = cur.next

        return head

```


# LeetCode 20: Valid Parentheses

```python
#
# [20] Valid Parentheses
#
# https://leetcode.com/problems/valid-parentheses/description/
#
# algorithms
# Easy (34.54%)
# Total Accepted:    402.5K
# Total Submissions: 1.2M
# Testcase Example:  '"()"'
#
# Given a string containing just the characters '(', ')', '{', '}', '[' and
# ']', determine if the input string is valid.
# 
# An input string is valid if:
# 
# 
# Open brackets must be closed by the same type of brackets.
# Open brackets must be closed in the correct order.
# 
# 
# Note that an empty string is also considered valid.
# 
# Example 1:
# 
# 
# Input: "()"
# Output: true
# 
# 
# Example 2:
# 
# 
# Input: "()[]{}"
# Output: true
# 
# 
# Example 3:
# 
# 
# Input: "(]"
# Output: false
# 
# 
# Example 4:
# 
# 
# Input: "([)]"
# Output: false
# 
# 
# Example 5:
# 
# 
# Input: "{[]}"
# Output: true
# 
# 
#
class Solution:
    def isValid(self, s):
        """
        :type s: str
        :rtype: bool
        """
        if not s: 
            return True 
        elif len(s) % 2 == 1:
            return False
        else:
            check_dict = {'(':')', '{':'}', '[':']'}
            stack = []
            for item in s:
                if item in check_dict:
                    stack.append(item)
                else:
                    if not stack or check_dict[stack[-1]] != item:
                        return False
                    else:
                        stack.pop()

            return True if not stack else False

```

# LeetCode 300: Longest Increasing Subsequence

**1. 第一种最普遍解法，每次往前遍历所有已知`lis`数组元素，来更新`lis[i]`，时间复杂度`O(N**2)`。**

```python
#
# [300] Longest Increasing Subsequence
#
# https://leetcode.com/problems/longest-increasing-subsequence/description/
#
# algorithms
# Medium (39.22%)
# Total Accepted:    154.8K
# Total Submissions: 394.3K
# Testcase Example:  '[10,9,2,5,3,7,101,18]'
#
# Given an unsorted array of integers, find the length of longest increasing
# subsequence.
# 
# Example:
# 
# 
# Input: [10,9,2,5,3,7,101,18]
# Output: 4 
# Explanation: The longest increasing subsequence is [2,3,7,101], therefore the
# length is 4. 
# 
# Note: 
# 
# 
# There may be more than one LIS combination, it is only necessary for you to
# return the length.
# Your algorithm should run in O(n2) complexity.
# 
# 
# Follow up: Could you improve it to O(n log n) time complexity?
# 
#
class Solution:
    def lengthOfLIS(self, nums):
        """
        :type nums: List[int]
        :rtype: int
        """
        length = len(nums)
        if length < 2:
            return length
        
        res = [1 for i in range(length)]
        i = 1
        while i < length:
            temp = nums[i]
            res[i] = 1
            for j in range(i-1, -1, -1):
                if temp > nums[j]:
                    res[i] = max(res[i], res[j]+1)
            i += 1

        return max(res)

```

**2. 一种新思路的解法， 跟解法一比较除了使用`lis`数组来保存以对应位置元素结尾的最长递增序列的长度之外，额外添加了一个`max_v`数组，对应数组元素`max_v[i]`代表`i`长度的递增子序列里的最大值的最小值；有了这个数组之后，我们就可以从当前已知最长的递增子序列长度`cur_max`开始递减匹配（递减匹配过程中，用`j`作中间变量），那么如果当前元素`nums[i]`大于`max_v[j]`，那么`i`位置最长的递增子序列长度`lis[i]=j+1`，同时我们要根据`lis[i]`的值来决定更不更新`cur_max`；同时也要根据`num[i]`是否小于`max_v[j+1]`来确定更不更新最长递增子序列长度为`j+1`的那些递增子序列的最大值，此算法时间复杂度`O(n**2)`。**

```python
class Solution:
    def lengthOfLIS(self, nums):
        """
        :type nums: List[int]
        :rtype: int
        """
        length = len(nums)
        if length < 2:
            return length
        
        max_v = [0 for i in range(length+1)]

        max_v[0] = min(nums) - 1
        max_v[1] = nums[0]
        lis = [1 for i in range(length)]

        cur_max = 1
        for i in range(1, length):
            for j in range(cur_max, -1, -1):
                if nums[i] > max_v[j]:
                    lis[i] = j + 1
                    break
            # 如果更新了当前最大值，那么更新max_v
            if lis[i] > cur_max:
                cur_max = lis[i]
                max_v[cur_max] = nums[i]
            # 如果没有更新最大值，那么找到可能被i更新的j
            # 当nums[i] > max_v[j]也就是进入到了上面for循环中的if条件中
            # 如果进入了条件中，那么如果nums[i]比原来max_v[j+1]小的时候才有
            # 更新价值；
            elif nums[i] > max_v[j] and nums[i] < max_v[j+1]:
                max_v[j+1] = nums[i]
        
        return cur_max
```

********3. 上面的顺序匹配过程还可以改成二分，因为如果`i>j`, 那么一定有`max_v[i] > max_v[j]`，此算法时间复杂度`O(n*logn)`。**

```python
class Solution:
    def lengthOfLIS(self, nums):
        """
        :type nums: List[int]
        :rtype: int
        """
        length = len(nums)
        if length < 2:
            return length
        
        max_v = [0 for i in range(length+1)]

        max_v[0] = min(nums) - 1
        max_v[1] = nums[0]
        lis = [1 for i in range(length)]

        cur_max = 1
        for i in range(1, length):
            index = self.get_index(max_v[:cur_max+1], nums[i])
            if index >= 0:
                max_v[index+1] = nums[i]
                cur_max = max(cur_max, index+1)
                
        return cur_max

    def get_index(self, max_v, num):
        length = len(max_v)

        if num > max_v[-1]:
            return length - 1
        if num < max_v[0]:
            return -1

        low = 0
        high = length - 1
        while low <= high:
            middle = low + ((high - low) >> 1)
            if max_v[middle] < num and max_v[middle+1] > num:
                return middle
            elif max_v[middle] == num:
                return -1
            elif max_v[middle] > num:
                high = middle - 1
            else:
                low = middle + 1
```


# LeetCode 5: Longest Palindromic Substring

![manacher](http://oq782gkz3.bkt.clouddn.com/manacher.JPG)

```python
#
# [5] Longest Palindromic Substring
#
# https://leetcode.com/problems/longest-palindromic-substring/description/
#
# algorithms
# Medium (25.62%)
# Total Accepted:    370.8K
# Total Submissions: 1.4M
# Testcase Example:  '"babad"'
#
# Given a string s, find the longest palindromic substring in s. You may assume
# that the maximum length of s is 1000.
# 
# Example 1:
# 
# 
# Input: "babad"
# Output: "bab"
# Note: "aba" is also a valid answer.
# 
# 
# Example 2:
# 
# 
# Input: "cbbd"
# Output: "bb"
# 
# 
#
class Solution:
    def longestPalindrome(self, s):
        """
        :type s: str
        :rtype: str
        """
        if s is None:
            return ''

        length = len(s)
        if length < 2:
            return s
        s = self.str_trans(s)
        length = len(s)
        res = [1 for i in range(length)]
        i = 0
        while i < length:
            j, step = i, 1
            while j-step > -1 and j+step < length and s[j-step] == s[j+step]:
                res[i] += 2
                step += 1
            i += 1

        lp = max(res)
        lp_index = res.index(lp)
        lp = lp // 2
        lp = s[lp_index-lp:lp_index+lp+1]
        return lp.replace('#', '')

    def str_trans(self, s):
        s = list(s)
        for i in range(len(s)):
            s[i] = '#' + s[i]
        s.append('#')
        return ''.join(s)
```


**使用manacher算法的解法：**

```python
class Solution:
    def longestPalindrome(self, s):
        """
        :type s: str
        :rtype: str
        """
        if s is None:
            return ''

        length = len(s)
        if length < 2:
            return s

        s = self.str_trans(s)

        length = len(s)
        res = [1 for i in range(length)]
        right = c  = -1
        for i in range(length):
            res[i] = min(res[2*c-i], right-i) if right > i else 1
            while i-res[i] > -1 and i+res[i] < length:
                if s[i-res[i]] == s[i+res[i]]:
                    res[i] += 1
                else:
                    break
            res[i] -= 1

            if i + res[i] > right:
                right = i + res[i]
                c = i

        lp = max(res)
        lp_index = res.index(lp)
        lp = s[lp_index-lp:lp_index+lp+1]
        return lp.replace('#', '')

    def str_trans(self, s):
        s = list(s)
        for i in range(len(s)):
            s[i] = '#' + s[i]
        s.append('#')
        return ''.join(s)

```
