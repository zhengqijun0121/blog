# 力扣32. Longest Valid Parentheses（最长有效括号）


## 力扣32. Longest Valid Parentheses（最长有效括号）

给你一个只包含 '(' 和 ')' 的字符串，找出最长有效（格式正确且连续）括号子串的长度。

示例 1：

![](../posts/01_学习/87_LeetCode/0032_最长有效括号/img/0032-1-description.png)

```
输入：s = "(()"
输出：2
解释：最长有效括号子串是 "()"。
```

示例 2：

![](../posts/01_学习/87_LeetCode/0032_最长有效括号/img/0032-2-description.png)

```
输入：s = ")()())"
输出：4
解释：最长有效括号子串是 "()()"。
```

示例 3：

![](../posts/01_学习/87_LeetCode/0032_最长有效括号/img/0032-3-description.png)

```
输入：s = ""
输出：0
```

提示：
- 0 <= s.length <= 3 * 10^4
- s[i] 为 '(' 或 ')'

