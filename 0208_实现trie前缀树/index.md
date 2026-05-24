# 力扣208. 实现 Trie 前缀树


## 力扣208. Implement Trie (Prefix Tree)（实现 Trie 前缀树）

实现一个 Trie 类，包含 insert、search 和 startsWith 方法。

示例 1：

![](../posts/01_学习/87_LeetCode/0208_实现Trie前缀树/img/0208-1-description.png)

```
输入：
["Trie", "insert", "search", "search", "startsWith", "insert", "search"]
[[], ["apple"], ["apple"], ["app"], ["app"], ["app"], ["app"]]
输出：
[null, null, true, false, true, null, true]
```

提示：
- 1 <= word.length, prefix.length <= 2000
- word 和 prefix 仅由小写英文字母组成
- insert、search 和 startsWith 调用次数总计不超过 3 * 10^4 次

