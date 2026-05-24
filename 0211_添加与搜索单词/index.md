# 力扣211. Design Add and Search Words Data Structure（添加与搜索单词）


## 力扣211. Design Add and Search Words Data Structure（添加与搜索单词）

设计一个支持添加单词和搜索单词的数据结构，搜索支持 '.' 通配符。

示例 1：

![](../posts/01_学习/87_LeetCode/0211_添加与搜索单词/img/0211-1-description.png)

```
输入：
["WordDictionary","addWord","addWord","addWord","search","search","search","search"]
[[],["bad"],["dad"],["mad"],["pad"],["bad"],[".ad"],["b.."]]
输出：
[null,null,null,null,false,true,true,true]
```

提示：
- 1 <= word.length <= 25
- addWord 中的 word 由小写英文字母组成
- search 中的 word 由 '.' 或小写英文字母组成
- 最多调用 10^4 次 addWord 和 search

