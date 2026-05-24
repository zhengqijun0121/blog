# 力扣171. Excel 表列序号


## 力扣171. Excel Sheet Column Number（Excel 表列序号）

给你一个字符串 `columnTitle`，表示 Excel 表格中的列名称，返回该列名称对应的列序号。

例如：
- `A` → 1
- `B` → 2
- `C` → 3
- ...
- `Z` → 26
- `AA` → 27
- `AB` → 28

示例 1：

![](../posts/01_学习/87_LeetCode/0171_Excel表列序号/img/0171-1-description.png)

```
输入：columnTitle = "A"
输出：1
```

示例 2：

![](../posts/01_学习/87_LeetCode/0171_Excel表列序号/img/0171-2-description.png)

```
输入：columnTitle = "AB"
输出：28
```

提示：
- `1 <= columnTitle.length <= 7`
- `columnTitle` 仅由大写英文组成

