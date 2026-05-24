# 力扣165. 比较版本号


## 力扣165. Compare Version Numbers（比较版本号）

给你两个版本号字符串 version1 和 version2，请你比较它们。版本号由以点 `.` 分隔的修订号组成。

返回规则：如果 version1 > version2 返回 1，如果 version1 < version2 返回 -1，否则返回 0。

示例 1：

![](../posts/01_学习/87_LeetCode/0165_比较版本号/img/0165-1-description.png)

```
输入：version1 = "1.01", version2 = "1.001"
输出：0
解释：忽略前导零，"01" 和 "001" 都表示相同的整数 "1"。
```

示例 2：

![](../posts/01_学习/87_LeetCode/0165_比较版本号/img/0165-2-description.png)

```
输入：version1 = "1.0", version2 = "1.0.0"
输出：0
解释：version1 没有第三级修订号，意味着其第三级修订号默认为 "0"。
```

示例 3：

![](../posts/01_学习/87_LeetCode/0165_比较版本号/img/0165-3-description.png)

```
输入：version1 = "0.1", version2 = "1.1"
输出：-1
```

提示：

- 1 <= version1.length, version2.length <= 500
- 版本号仅包含数字和字符 `.`
- 版本号至少包含一个数字

