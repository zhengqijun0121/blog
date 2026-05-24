# 力扣733. 图像渲染


## 力扣733. Flood Fill（图像渲染）

将相邻的相同颜色的像素替换为新颜色。

示例 1：

![](../posts/01_学习/87_LeetCode/0733_图像渲染/img/0733-1-description.png)

```
输入：image = [[1,1,1],[1,1,0],[1,0,1]], sr = 1, sc = 1, newColor = 2
输出：[[2,2,2],[2,2,0],[2,0,1]]
解释：...
```

提示：
- m == image.length, n == image[i].length
- 1 <= m, n <= 50
- 0 <= image[i][j], newColor < 2^16
- 0 <= sr < m, 0 <= sc < n

