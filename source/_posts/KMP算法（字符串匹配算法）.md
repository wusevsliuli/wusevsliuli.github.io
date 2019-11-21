---
title: KMP算法（字符串匹配算法）
date: 2019-09-06 18:25:43
categories:
 - 算法
tags:
 - 算法
---
最近各种文本类匹配搜索的数据分析任务，写解析器写的头都大了，KMP算法算是非常著名的算法了，这里写一下算法的解析。
<escape><!-- more --></escape>

### 题目
给定一个 haystack 字符串和一个 needle 字符串，在 haystack 字符串中找出 needle 字符串出现的第一个位置 (从0开始)。如果不存在，则返回  -1。

示例 1:
>输入: haystack = "hello", needle = "ll"  
>输出: 2

示例 2:
>输入: haystack = "aaaaa", needle = "bba"  
>输出: -1

说明:

当 needle 是空字符串时，我们应当返回什么值呢？这是一个在面试中很好的问题。

来源：[力扣（LeetCode）](https://leetcode-cn.com/problems/implement-strstr)

### 解题
##### 暴力匹配算法
```python
# 暴力匹配算法
def str_find(S, P):
    if not P:
        return 0
    i = 0  # S 中匹配 P 的第一个字符的位置，也是最终的返回值
    j = 0  # S 中匹配 P 的位置，此时比较的是 S[i+j] ?= P[j]
    while i + len(P) <= len(S):
        if S[i + j] == P[j]:
            # 相等情况下，j向前移动一位
            j += 1
            if j == len(P):
                return i
        else:
            # 不相等的情况下重置j, 然后i向前移动一位
            i += 1
            j = 0
    return -1
```
##### KMP算法  
KMP算法是基于暴力匹配算法的优化  
文本字符串串为 **S**, 模式串(需要匹配的字符串)为 **P**  
* 假设现在文本串S匹配到 i 位置，模式串P匹配到 j 位置  
    * 如果j = -1，或者当前字符匹配成功（即S[i] == P[j]），都令i++，j++，继续匹配下一个字符；  
    * 如果j != -1，且当前字符匹配失败（即S[i] != P[j]），则令 i 不变，j = next[j]   
        此举意味着失配时，模式串P相对于文本串S向右移动了j - next [j] 位。换言之，当匹配失败时，模式串向右移动的位数为：失配字符所在位置 - 失配字符对应的next 值（next 数组的求解会在下文的3.3.3节中详细阐述），即移动的实际位数为：j - next[j]，且此值大于等于1。  