# 力扣142. 环形链表


## 力扣142. 环形链表 II

给定一个链表的头节点 `head`，返回链表开始入环的第一个节点。如果链表无环，则返回 `null`。

如果链表中有某个节点，可以通过连续跟踪 `next` 指针再次到达，则链表中存在环。为了表示给定链表中的环，评测系统内部使用整数 `pos` 来表示链表尾连接到链表中的位置（`索引从 0 开始`）。如果 `pos` 是 `-1`，则在该链表中没有环。

{{< admonition >}}
注意：`pos` 不作为参数进行传递，仅仅是为了标识链表的实际情况。不允许修改链表。
{{< /admonition >}}

示例 1：

{{< image src="../posts/01_学习/88_LeetCode/0142_环形链表/img/142-1-circularlinkedlist.png" width="531" height="171" >}}

```
输入：head = [3, 2, 0, -4], pos = 1
输出：返回索引为 1 的链表节点
解释：链表中有一个环，其尾部连接到第二个节点。
```

示例 2：

{{< image src="../posts/01_学习/88_LeetCode/0142_环形链表/img/142-2-circularlinkedlist_test2.png" width="201" height="105" >}}

```
输入：head = [1, 2], pos = 0
输出：返回索引为 0 的链表节点
解释：链表中有一个环，其尾部连接到第一个节点。
```

示例 3：

{{< image src="../posts/01_学习/88_LeetCode/0142_环形链表/img/142-3-circularlinkedlist_test3.png" width="65" height="65" >}}

```
输入：head = [1], pos = -1
输出：返回 null
解释：链表中没有环。
```

{{< admonition >}}
提示：
- 链表中节点的数目范围在范围 [0, 104] 内
- -105 <= Node.val <= 105
- pos 的值为 -1 或者链表中的一个有效索引
{{< /admonition >}}

进阶：你是否可以使用 O(1) 空间解决此题？

{{< admonition quote >}}
来源：力扣（LeetCode）

链接：[https://leetcode-cn.com/problems/linked-list-cycle-ii](https://leetcode-cn.com/problems/linked-list-cycle-ii)

著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。
{{< /admonition >}}

----

## 哈希表

### 思路与算法

遍历链表中的每个节点，并记录下来。一旦遇到之前保存过的节点，就可以判定为环形链表。为了方便查询，使用哈希表结构实现。

### 代码实现

1. C++ 实现

```c++
class Solution {
public:
    ListNode *detectCycle(ListNode *head) {
        unordered_set<ListNode *> visited;
        while (head != nullptr) {
            if (visited.count(head)) {
                return head;
            }
            visited.insert(head);
            head = head->next;
        }
        return nullptr;
    }
};
```

2. Java 实现

```java
public class Solution {
    public ListNode detectCycle(ListNode head) {
        ListNode pos = head;
        Set<ListNode> visited = new HashSet<ListNode>();
        while (pos != null) {
            if (visited.contains(pos)) {
                return pos;
            } else {
                visited.add(pos);
            }
            pos = pos.next;
        }
        return null;
    }
}
```

### 复杂度分析

时间复杂度：O(N)，其中 N 为链表中节点的数目。需要访问链表中的每一个节点。
空间复杂度：O(N)，其中 N 为链表中节点的数目。需要将链表中的每个节点都保存在哈希表当中。

----

## 快慢指针

### 思路与算法

使用两个指针，`fast` 与 `slow`。它们起始都位于链表的头部。随后，`slow` 指针每次向后移动一个位置，而 `fast` 指针向后移动两个位置。如果链表中存在环，则 `fast` 指针最终将再次与 `slow` 指针在环中相遇。

### 代码实现

1. C++ 实现

```c++
class Solution {
public:
    ListNode *detectCycle(ListNode *head) {
        ListNode *slow = head;
        ListNode *fast = head;
        while (fast != nullptr) {
            slow = slow->next;
            if (fast->next == nullptr) {
                return nullptr;
            }
            fast = fast->next->next;
            if (fast == slow) {
                ListNode *ptr = head;
                while (ptr != slow) {
                    ptr = ptr->next;
                    slow = slow->next;
                }
                return ptr;
            }
        }
        return nullptr;
    }
};
```

2. Java 实现

```java
public class Solution {
    public ListNode detectCycle(ListNode head) {
        if (head == null) {
            return null;
        }
        ListNode slow = head;
        ListNode fast = head;
        while (fast != null) {
            slow = slow.next;
            if (fast.next != null) {
                fast = fast.next.next;
            } else {
                return null;
            }
            if (fast == slow) {
                ListNode ptr = head;
                while (ptr != slow) {
                    ptr = ptr.next;
                    slow = slow.next;
                }
                return ptr;
            }
        }
        return null;
    }
}
```

### 复杂度分析

- 时间复杂度：O(N)，其中 N 为链表中节点的数目。在最初判断快慢指针是否相遇时，slow 指针走过的距离不会超过链表的总长度；随后寻找入环点时，走过的距离也不会超过链表的总长度。
- 空间复杂度：O(1)。只使用了三个指针。

----

## 快慢指针的数学分析

### 快慢指针的相遇问题

如下图所示，设链表中环外部分的长度为 `a`。`slow` 指针进入环后，又走了 `b` 的距离与 `fast` 相遇。此时，`fast` 指针已经走完了环的 `n` 圈，因此它走过的总距离为 `a + n(b + c) + b = a + (n + 1)b + nc`。

{{< image src="../posts/01_学习/88_LeetCode/0142_环形链表/img/142-4-fig1.png" width="608" height="342" >}}

设 `fast` 指针走过的距离都为 `slow` 指针的 `x` 倍，因此有 `a + (n + 1)b + nc = x(a + b)` ⟹  `(b + c)n = (x − 1)(a + b)`。即从相遇点到入环点的距离加上 `n` 圈的环长，恰好等于从链表头部到入环点的距离的 `x - 1` 倍。

若 `x = 2`，快指针步长为慢指针的 2 倍，`(b + c)n = a + b` 方程组有解。若 `x = 3`，快指针步长为慢指针的 3 倍，`(b + c)n = 2(a + b)` 方程组有解。

### 循环圈的起始位置问题

由 `(b + c)n = (x - 1)(a + b)` 方程可知从相遇点到入环点的距离是环长的倍数。因此，当发现 `slow` 与 `fast` 相遇时，我们再额外使用一个指针 `ptr`。起始位置指向链表头部，随后它和 `slow` 每次向后移动一个位置。最终，它们会在入环点相遇。

----


