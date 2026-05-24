# 力扣85. 最大矩形


## 力扣85. Maximal Rectangle（最大矩形）

给定一个仅包含 0 和 1、大小为 rows×cols 的二维二进制矩阵，找出只包含 1 的最大矩形，并返回其面积。

示例 1：

![](../posts/01_学习/87_LeetCode/0085_最大矩形/img/0085-1-description.png)

```
输入：matrix = [["1","0","1","0","0"],["1","0","1","1","1"],["1","1","1","1","1"],["1","0","0","1","0"]]
输出：6
```

示例 2：

![](../posts/01_学习/87_LeetCode/0085_最大矩形/img/0085-2-description.png)

```
输入：matrix = [["0"]]
输出：0
```

提示：
- rows == matrix.length
- cols == matrix[0].length
- 1 <= rows, cols <= 200

