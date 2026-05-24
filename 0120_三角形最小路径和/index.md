# 力扣120. Triangle（三角形最小路径和）


## 力扣120. Triangle（三角形最小路径和）

给定一个三角形 triangle，找出自顶向下的最小路径和。每一步只能移动到下一行中相邻的节点上。相邻的节点指下标与上一层节点下标相同或等于上一层节点下标+1的两个节点。

示例 1：

![](../posts/01_学习/87_LeetCode/0120_三角形最小路径和/img/0120-1-description.png)

```
输入：triangle = [[2],[3,4],[6,5,7],[4,1,8,3]]
输出：11
解释：自顶向下的最小路径和为 11（2 → 3 → 5 → 1）。
```

示例 2：

```
输入：triangle = [[-10]]
输出：-10
```

提示：
- 1 <= triangle.length <= 200
- triangle[0].length == 1
- triangle[i].length == triangle[i-1].length + 1
- -10^4 <= triangle[i][j] <= 10^4

