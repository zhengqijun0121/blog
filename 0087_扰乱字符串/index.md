# 力扣87. Scramble String（扰乱字符串）


## 力扣87. Scramble String（扰乱字符串）

我们可以将一个字符串通过递归方式将其分割成两个非空的子字符串，然后交换这两个子字符串的位置来打乱字符串。输入两个字符串 s1 和 s2，判断 s2 是否为 s1 的扰乱字符串。

示例 1：

```
输入：s1 = "great", s2 = "rgeat"
输出：true
```

示例 2：

```
输入：s1 = "abcde", s2 = "caebd"
输出：false
```

提示：
- s1.length == s2.length
- 1 <= s1.length <= 30
- s1 和 s2 由小写英文字母组成

