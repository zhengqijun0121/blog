# 力扣151. 反转字符串中的单词


## 力扣151. Reverse Words in a String（反转字符串中的单词）

给你一个字符串 s，请你反转字符串中单词的顺序。单词是由非空格字符组成的字符串。s 中使用至少一个空格将字符串中的单词分隔开。返回单词顺序颠倒且单词之间用单个空格连接的结果字符串。

示例 1：

![](../posts/01_学习/87_LeetCode/0151_反转字符串中的单词/img/0151-1-description.png)

```
输入：s = "the sky is blue"
输出："blue is sky the"
```

示例 2：

![](../posts/01_学习/87_LeetCode/0151_反转字符串中的单词/img/0151-2-description.png)

```
输入：s = "  hello world  "
输出："world hello"
解释：反转后的字符串中不能存在前导空格和尾随空格。
```

示例 3：

![](../posts/01_学习/87_LeetCode/0151_反转字符串中的单词/img/0151-3-description.png)

```
输入：s = "a good   example"
输出："example good a"
解释：如果两个单词间有多余的空格，请你将反转后单词间的空格减少到只含一个。
```

提示：
- 1 <= s.length <= 10^4
- s 包含英文大小写字母、数字和空格
- s 中至少存在一个单词

进阶：请你尝试使用 O(1) 额外空间复杂度的原地解法。

