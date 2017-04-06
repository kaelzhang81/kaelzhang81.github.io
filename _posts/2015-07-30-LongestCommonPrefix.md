---
layout:     post
title:      "longest_common_prefix"
subtitle:   "longest_common_prefix"
date:       2015-07-30 22:00:00
author:     "Kaelzhang"
header-img: "img/hearder/docker_concepts.png"
catalog:    true
tags:
    - leetcode
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

