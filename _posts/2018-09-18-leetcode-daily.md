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

# LeetCode 71: Simplify Path {#leetcode71}
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

# LeetCode 53: maximum subarray  {#leetcode53}
**典型动归**

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

# LeetCode 28: Implement strStr() {#leetcode28}
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

        return next_arr

```
