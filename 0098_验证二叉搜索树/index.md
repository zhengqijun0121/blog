# 力扣98. Validate Binary Search Tree（验证二叉搜索树）


## 力扣98. Validate Binary Search Tree（验证二叉搜索树）

给你一个二叉树的根节点 root，判断其是否是一个有效的二叉搜索树。有效二叉搜索树定义如下：节点的左子树只包含小于当前节点的数，节点的右子树只包含大于当前节点的数，所有左子树和右子树自身必须也是二叉搜索树。

示例 1：

![](../posts/01_学习/87_LeetCode/0098_验证二叉搜索树/img/0098-1-description.png)

```
输入：root = [2,1,3]
输出：true
解释：...
```

示例 2：

![](../posts/01_学习/87_LeetCode/0098_验证二叉搜索树/img/0098-2-description.png)

```
输入：root = [5,1,4,null,null,3,6]
输出：false
解释：...
```

提示：
- 树中节点数目范围在 [1, 10^4] 内
- -2^31 <= Node.val <= 2^31 - 1

