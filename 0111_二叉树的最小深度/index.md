# 力扣111. Minimum Depth of Binary Tree（二叉树的最小深度）


## 力扣111. Minimum Depth of Binary Tree（二叉树的最小深度）

给定一个二叉树，找出其最小深度。最小深度是从根节点到最近叶子节点的最短路径上的节点数量。叶子节点是指没有子节点的节点。

示例 1：

![](../posts/01_学习/87_LeetCode/0111_二叉树的最小深度/img/0111-1-description.png)

```
输入：root = [3,9,20,null,null,15,7]
输出：2
解释：最小深度为 2（根节点 3 → 9）。
```

示例 2：

![](../posts/01_学习/87_LeetCode/0111_二叉树的最小深度/img/0111-2-description.png)

```
输入：root = [2,null,3,null,4,null,5,null,6]
输出：5
```

提示：
- 树中节点数目在范围 [0, 10^5] 内
- -1000 <= Node.val <= 1000

