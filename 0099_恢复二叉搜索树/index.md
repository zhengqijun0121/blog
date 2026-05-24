# 力扣99. Recover Binary Search Tree（恢复二叉搜索树）


## 力扣99. Recover Binary Search Tree（恢复二叉搜索树）

给你二叉搜索树的根节点 root，该树中的恰好两个节点的值被错误地交换。请在不改变其结构的情况下，恢复这棵树。

示例 1：

![](../posts/01_学习/87_LeetCode/0099_恢复二叉搜索树/img/0099-1-description.png)

```
输入：root = [1,3,null,null,2]
输出：[3,1,null,null,2]
解释：...
```

示例 2：

![](../posts/01_学习/87_LeetCode/0099_恢复二叉搜索树/img/0099-2-description.png)

```
输入：root = [3,1,4,null,null,2]
输出：[2,1,4,null,null,3]
解释：...
```

提示：
- 树上节点的数目在范围 [2, 1000] 内
- -2^31 <= Node.val <= 2^31 - 1

进阶：使用 O(n) 空间复杂度的解法很容易实现。你能想出一个只使用 O(1) 空间的解决方案吗？

