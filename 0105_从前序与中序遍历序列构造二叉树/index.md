# 力扣105. Construct Binary Tree from Preorder and Inorder Traversal（从前序与中序遍历序列构造二叉树）


## 力扣105. Construct Binary Tree from Preorder and Inorder Traversal（从前序与中序遍历序列构造二叉树）

给定两个整数数组 preorder 和 inorder，其中 preorder 是二叉树的先序遍历，inorder 是同一棵树的中序遍历，请构造二叉树并返回其根节点。

示例 1：

![](../posts/01_学习/87_LeetCode/0105_从前序与中序遍历序列构造二叉树/img/0105-1-description.png)

```
输入：preorder = [3,9,20,15,7], inorder = [9,3,15,20,7]
输出：[3,9,20,null,null,15,7]
解释：...
```

示例 2：

```
输入：preorder = [-1], inorder = [-1]
输出：[-1]
解释：...
```

提示：
- 1 <= preorder.length <= 3000
- inorder.length == preorder.length
- -3000 <= Node.val <= 3000

