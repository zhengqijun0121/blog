# 力扣103. Binary Tree Zigzag Level Order Traversal（二叉树的锯齿形层序遍历）


## 力扣103. Binary Tree Zigzag Level Order Traversal（二叉树的锯齿形层序遍历）

给你二叉树的根节点 root，返回其节点值的锯齿形层序遍历。（即先从左往右，再从右往左进行下一层遍历，以此类推，层与层之间交替进行）。

示例 1：

![](../posts/01_学习/87_LeetCode/0103_二叉树的锯齿形层序遍历/img/0103-1-description.png)

```
输入：root = [3,9,20,null,null,15,7]
输出：[[3],[20,9],[15,7]]
解释：...
```

示例 2：

```
输入：root = [1]
输出：[[1]]
解释：...
```

示例 3：

```
输入：root = []
输出：[]
解释：...
```

提示：
- 树中节点数目在范围 [0, 2000] 内
- -100 <= Node.val <= 100

