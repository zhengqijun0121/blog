# 力扣187. 重复的DNA序列


## 力扣187. Repeated DNA Sequences（重复的DNA序列）

DNA 序列由一系列核苷酸组成，缩写为 'A'、'C'、'G' 和 'T'。例如，"ACGAATTCCG" 是一个 DNA 序列。给定一个表示 DNA 序列的字符串 s，返回所有在 DNA 分子中出现不止一次的长度为 10 的子序列。

示例 1：

![](../posts/01_学习/87_LeetCode/0187_重复的DNA序列/img/0187-1-description.png)

```
输入：s = "AAAAACCCCCAAAAACCCCCCAAAAAGGGTTT"
输出：["AAAAACCCCC","CCCCCAAAAA"]
```

示例 2：

![](../posts/01_学习/87_LeetCode/0187_重复的DNA序列/img/0187-2-description.png)

```
输入：s = "AAAAAAAAAAAAA"
输出：["AAAAAAAAAA"]
```

提示：
- 1 <= s.length <= 10^5
- s[i] 是 'A'、'C'、'G' 或 'T'

