# 力扣146. LRU 缓存


## 力扣146. LRU Cache（LRU 缓存）

请你设计并实现一个满足 LRU（最近最少使用）缓存约束的数据结构。实现 LRUCache 类：LRUCache(capacity) 以正整数作为容量初始化，get(key) 如果关键字 key 存在于缓存中则返回关键字的值否则返回 -1，put(key, value) 如果关键字 key 已经存在则变更其值，否则插入该组。当缓存容量达到上限时，它应该在写入新数据之前删除最久未使用的数据值。

示例 1：

![](../posts/01_学习/87_LeetCode/0146_LRU缓存/img/0146-1-description.png)

```
输入：
["LRUCache","put","put","get","put","get","put","get","get","get"]
[[2],[1,1],[2,2],[1],[3,3],[2],[4,4],[1],[3],[4]]
输出：[null,null,null,1,null,-1,null,-1,3,4]
解释：
LRUCache lRUCache = new LRUCache(2);
lRUCache.put(1, 1); // 缓存是 {1=1}
lRUCache.put(2, 2); // 缓存是 {1=1, 2=2}
lRUCache.get(1);    // 返回 1
lRUCache.put(3, 3); // 该操作会使得关键字 2 作废，缓存是 {1=1, 3=3}
lRUCache.get(2);    // 返回 -1 (未找到)
lRUCache.put(4, 4); // 该操作会使得关键字 1 作废，缓存是 {4=4, 3=3}
lRUCache.get(1);    // 返回 -1 (未找到)
lRUCache.get(3);    // 返回 3
lRUCache.get(4);    // 返回 4
```

提示：
- 1 <= capacity <= 3000
- 0 <= key <= 10^4
- 0 <= value <= 10^5
- 最多调用 2*10^5 次 get 和 put

进阶：你能在 O(1) 时间复杂度内完成这两种操作吗？

