# 力扣106. 从中序与后序遍历序列构造二叉树


## 力扣106. Construct Binary Tree from Inorder and Postorder Traversal（从中序与后序遍历序列构造二叉树）

给定两个整数数组 inorder 和 postorder，其中 inorder 是二叉树的中序遍历，postorder 是同一棵树的后序遍历，请构造二叉树并返回其根节点。

示例 1：

![](../posts/01_学习/87_LeetCode/0106_从中序与后序遍历序列构造二叉树/img/0106-1-description.png)

```
输入：inorder = [9,3,15,20,7], postorder = [9,15,7,20,3]
输出：[3,9,20,null,null,15,7]
解释：...
```

示例 2：

```
输入：inorder = [-1], postorder = [-1]
输出：[-1]
解释：...
```

提示：
- 1 <= inorder.length <= 3000
- postorder.length == inorder.length
- -3000 <= Node.val <= 3000

