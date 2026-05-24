# 力扣65. Valid Number（有效数字）


## 力扣65. Valid Number（有效数字）

给定一个字符串 s，返回 s 是否是一个有效数字。有效数字（按顺序）可以分成以下几个部分：一个小数或者整数，可选的 e 或 E 加上一个有符号整数。

示例 1：

![](../posts/01_学习/87_LeetCode/0065_有效数字/img/0065-1-description.png)

```
输入：s = "0"
输出：true
```

示例 2：

![](../posts/01_学习/87_LeetCode/0065_有效数字/img/0065-2-description.png)

```
输入：s = "e"
输出：false
```

示例 3：

![](../posts/01_学习/87_LeetCode/0065_有效数字/img/0065-3-description.png)

```
输入：s = "."
输出：false
```

提示：
- 1 <= s.length <= 20
- s 仅含英文字母（大写和小写）、数字（0-9）、加号 '+'、减号 '-'、或者空格 ' '

