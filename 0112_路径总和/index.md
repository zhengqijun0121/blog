# 力扣112. Path Sum（路径总和）


## 力扣112. Path Sum（路径总和）

给你二叉树的根节点 root 和一个表示目标和的整数 targetSum，判断该树中是否存在根节点到叶子节点的路径，这条路径上所有节点值相加等于目标和 targetSum。

示例 1：

![](../posts/01_学习/87_LeetCode/0112_路径总和/img/0112-1-description.png)

```
输入：root = [5,4,8,11,null,13,4,7,2,null,null,null,1], targetSum = 22
输出：true
解释：根节点到叶子节点路径 5 → 4 → 11 → 2 和为 22。
```

示例 2：

![](../posts/01_学习/87_LeetCode/0112_路径总和/img/0112-2-description.png)

```
输入：root = [1,2,3], targetSum = 5
输出：false
```

示例 3：

```
输入：root = [], targetSum = 0
输出：false
```

提示：
- 树中节点数目在范围 [0, 5000] 内
- -1000 <= Node.val <= 1000
- -1000 <= targetSum <= 1000

