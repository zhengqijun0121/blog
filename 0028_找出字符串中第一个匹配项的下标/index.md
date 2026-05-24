# 力扣28. 找出字符串中第一个匹配项的下标


## 力扣28. Find the Index of the First Occurrence in a String（找出字符串中第一个匹配项的下标）

给你两个字符串 haystack 和 needle，请你在 haystack 字符串中找出 needle 字符串的第一个匹配项的下标（下标从 0 开始）。如果 needle 不是 haystack 的一部分，则返回 -1。

示例 1：

![](../posts/01_学习/87_LeetCode/0028_找出字符串中第一个匹配项的下标/img/0028-1-description.png)

```
输入：haystack = "sadbutsad", needle = "sad"
输出：0
解释："sad" 在下标 0 和 6 处匹配，第一个匹配项的下标是 0。
```

示例 2：

![](../posts/01_学习/87_LeetCode/0028_找出字符串中第一个匹配项的下标/img/0028-2-description.png)

```
输入：haystack = "leetcode", needle = "leeto"
输出：-1
解释："leeto" 没有在 "leetcode" 中出现，返回 -1。
```

提示：
- 1 <= haystack.length, needle.length <= 10^4
- haystack 和 needle 仅由小写英文字符组成

