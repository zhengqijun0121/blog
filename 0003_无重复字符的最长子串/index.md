# 力扣3. Longest Substring Without Repeating Characters（无重复字符的最长子串）


## 力扣3. Longest Substring Without Repeating Characters（无重复字符的最长子串）

给定一个字符串 s ，请你找出其中不含有重复字符的最长子串的长度。

示例 1：

![](../posts/01_学习/87_LeetCode/0003_无重复字符的最长子串/img/0003-1-description.png)

```
输入：s = "abcabcbb"
输出：3
解释：因为无重复字符的最长子串是 "abc"，所以其长度为 3。
```

示例 2：

![](../posts/01_学习/87_LeetCode/0003_无重复字符的最长子串/img/0003-2-description.png)

```
输入：s = "bbbbb"
输出：1
解释：因为无重复字符的最长子串是 "b"，所以其长度为 1。
```

示例 3：

![](../posts/01_学习/87_LeetCode/0003_无重复字符的最长子串/img/0003-3-description.png)

```
输入：s = "pwwkew"
输出：3
解释：因为无重复字符的最长子串是 "wke"，所以其长度为 3。请注意，你的答案必须是子串的长度，"pwke" 是一个子序列，不是子串。
```

提示：
- 0 <= s.length <= 5 * 10^4
- s 由英文字母、数字、符号和空格组成

