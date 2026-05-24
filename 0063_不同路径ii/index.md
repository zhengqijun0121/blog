# 力扣63. Unique Paths II（不同路径 II）


## 力扣63. Unique Paths II（不同路径 II）

一个机器人位于一个 m×n 网格的左上角。机器人每次只能向下或者向右移动一步。网格中的障碍物和空位置分别用 1 和 0 来表示。机器人试图达到网格的右下角，问总共有多少条不同的路径？

示例 1：

![](../posts/01_学习/87_LeetCode/0063_不同路径II/img/0063-1-description.png)

```
输入：obstacleGrid = [[0,0,0],[0,1,0],[0,0,0]]
输出：2
```

示例 2：

![](../posts/01_学习/87_LeetCode/0063_不同路径II/img/0063-2-description.png)

```
输入：obstacleGrid = [[0,1],[0,0]]
输出：1
```

提示：
- m == obstacleGrid.length
- n == obstacleGrid[i].length
- 1 <= m, n <= 100
- obstacleGrid[i][j] 为 0 或 1

