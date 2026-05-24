# 力扣141. 环形链表


## 力扣141. Linked List Cycle（环形链表）

给你一个链表的头节点 head，判断链表中是否有环。如果链表中有某个节点，可以通过连续跟踪 next 指针再次到达，则链表中存在环。返回 true 表示链表中有环，false 表示没有。

示例 1：

![](../posts/01_学习/87_LeetCode/0141_环形链表/img/0141-1-description.png)

```
输入：head = [3,2,0,-4], pos = 1
输出：true
解释：...
```

示例 2：

![](../posts/01_学习/87_LeetCode/0141_环形链表/img/0141-2-description.png)

```
输入：head = [1,2], pos = 0
输出：true
```

示例 3：

![](../posts/01_学习/87_LeetCode/0141_环形链表/img/0141-3-description.png)

```
输入：head = [1], pos = -1
输出：false
```

提示：
- 链表中节点数目范围在 [0, 10^4] 内
- -10^5 <= Node.val <= 10^5

进阶：你能用 O(1) 内存解决此问题吗？

