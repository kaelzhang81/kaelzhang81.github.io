---
layout: post
title: [leetcode]longest_common_prefix
category: leetcode
---

#####需求描述:

Write a function to find the longest common prefix string amongst an array of strings.

写一个函数获取字符串数组中最长的字符串前缀

***
#####pythonic实现:
~~~python
class Solution:
    def longestCommonPrefix(self, strs):
        ret, l = '', len(strs)
        for t in zip(*strs):
            if t != tuple(t[0] * l):
                break
            ret += t[0]
        return ret
~~~
