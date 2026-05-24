# 力扣134. Gas Station（加油站）


## 力扣134. Gas Station（加油站）

在一条环路上有 n 个加油站，其中第 i 个加油站有汽油 gas[i] 升。你有一辆油箱容量无限的汽车，从第 i 个加油站开往第 i+1 个加油站需要消耗汽油 cost[i] 升。如果你可以绕环路行驶一周，则返回出发时加油站的编号，否则返回 -1。

示例 1：

![](../posts/01_学习/87_LeetCode/0134_加油站/img/0134-1-description.png)

```
输入：gas = [1,2,3,4,5], cost = [3,4,5,1,2]
输出：3
解释：...
```

示例 2：

![](../posts/01_学习/87_LeetCode/0134_加油站/img/0134-2-description.png)

```
输入：gas = [2,3,4], cost = [3,4,3]
输出：-1
```

提示：
- gas.length == n
- cost.length == n
- 1 <= n <= 10^5
- 0 <= gas[i], cost[i] <= 10^4

