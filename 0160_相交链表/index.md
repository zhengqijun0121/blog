# 力扣160. 相交链表


## 力扣160. Intersection of Two Linked Lists（相交链表）

给你两个单链表的头节点 headA 和 headB，请你找出并返回两个单链表相交的起始节点。如果两个链表不存在相交节点，返回 null。

示例 1：

![](../posts/01_学习/87_LeetCode/0160_相交链表/img/0160-1-description.png)

```
输入：intersectVal = 8, listA = [4,1,8,4,5], listB = [5,6,1,8,4,5], skipA = 2, skipB = 3
输出：Intersected at '8'
解释：相交节点的值为 8。
```

示例 2：

![](../posts/01_学习/87_LeetCode/0160_相交链表/img/0160-2-description.png)

```
输入：intersectVal = 2, listA = [1,9,1,2,4], listB = [3,2,4], skipA = 3, skipB = 1
输出：Intersected at '2'
解释：相交节点的值为 2。
```

示例 3：

![](../posts/01_学习/87_LeetCode/0160_相交链表/img/0160-3-description.png)

```
输入：intersectVal = 0, listA = [2,6,4], listB = [1,5], skipA = 3, skipB = 2
输出：null
解释：两个链表没有相交节点，返回 null。
```

提示：
- listA 中节点数目为 m
- listB 中节点数目为 n
- 1 <= m, n <= 3 * 10^4
- 1 <= Node.val <= 10^5
- 0 <= skipA <= m
- 0 <= skipB <= n
- 如果 listA 和 listB 没有交点，intersectVal 为 0

进阶：你能设计一个时间复杂度 O(m + n)、仅用 O(1) 内存的解决方案吗？

