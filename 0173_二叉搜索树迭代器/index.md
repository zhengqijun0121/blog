# 力扣173. 二叉搜索树迭代器


## 力扣173. Binary Search Tree Iterator（二叉搜索树迭代器）

实现一个 `BSTIterator` 类，表示一个按**中序遍历**二叉搜索树的迭代器：

- `BSTIterator(TreeNode root)`：初始化迭代器，给定二叉搜索树的根节点
- `int next()`：返回下一个最小的数
- `boolean hasNext()`：返回是否还有下一个数

示例 1：

![](../posts/01_学习/87_LeetCode/0173_二叉搜索树迭代器/img/0173-1-description.png)

```
输入：
["BSTIterator", "next", "next", "hasNext", "next", "hasNext"]
[[[7,3,15,null,null,9,20]], [], [], [], [], []]
输出：
[null, 3, 7, true, 9, false]

解释：
BSTIterator bSTIterator = new BSTIterator([7,3,15,null,null,9,20]);
bSTIterator.next();    // 返回 3
bSTIterator.next();    // 返回 7
bSTIterator.hasNext(); // 返回 True
bSTIterator.next();    // 返回 9
bSTIterator.hasNext(); // 返回 False
```

提示：
- 树中节点数目在范围 `[1, 10^5]` 内
- `0 <= Node.val <= 10^6`
- 最多调用 `10^5` 次 `next()` 和 `hasNext()`

