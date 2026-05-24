# 力扣97. 交错字符串


## 力扣97. Interleaving String（交错字符串）

给定三个字符串 s1、s2、s3，请你帮忙验证 s3 是否是由 s1 和 s2 交错组成的。交错组成是指 s3 由 s1 和 s2 按顺序交错合并而成，且 s3 中每个字符的顺序与在 s1 和 s2 中的原始顺序一致。

示例 1：

![](../posts/01_学习/87_LeetCode/0097_交错字符串/img/0097-1-description.png)

```
输入：s1 = "aabcc", s2 = "dbbca", s3 = "aadbbcbcac"
输出：true
解释：...
```

示例 2：

```
输入：s1 = "aabcc", s2 = "dbbca", s3 = "aadbbbaccc"
输出：false
解释：...
```

提示：
- 0 <= s1.length, s2.length <= 100
- 0 <= s3.length <= 200
- s1、s2 和 s3 由小写英文字母组成

进阶：你能只用 O(s2.length) 的额外内存空间来解决吗？

