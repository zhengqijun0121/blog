# 力扣235. Lowest Common Ancestor of a Binary Search Tree（二叉搜索树的最近公共祖先）


## 力扣235. Lowest Common Ancestor of a Binary Search Tree（二叉搜索树的最近公共祖先）

给定一个二叉搜索树，找到该树中两个指定节点的最近公共祖先。

示例 1：

![](../posts/01_学习/87_LeetCode/0235_二叉搜索树的最近公共祖先/img/0235-1-description.png)

```
输入：root = [6,2,8,0,4,7,9,null,null,3,5], p = 2, q = 8
输出：6
解释：节点 2 和节点 8 的最近公共祖先是 6。
```

提示：
- 树中节点数目在 [2, 10^5] 内
- -10^9 <= Node.val <= 10^9
- 所有 Node.val 互不相同
- p != q
- p 和 q 均存在于给定的二叉搜索树中

