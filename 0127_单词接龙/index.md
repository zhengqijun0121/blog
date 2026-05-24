# 力扣127. Word Ladder（单词接龙）


## 力扣127. Word Ladder（单词接龙）

字典 wordList 中从单词 beginWord 到 endWord 的转换序列是一个序列，相邻单词仅一个字母不同。返回从 beginWord 到 endWord 的最短转换序列的长度。如果不存在这样的转换序列，返回 0。

示例 1：

![](../posts/01_学习/87_LeetCode/0127_单词接龙/img/0127-1-description.png)

```
输入：beginWord = "hit", endWord = "cog", wordList = ["hot","dot","dog","lot","log","cog"]
输出：5
解释：一个最短转换序列是 "hit" -> "hot" -> "dot" -> "dog" -> "cog"，返回它的长度 5。
```

提示：
- 1 <= beginWord.length <= 10
- endWord.length == beginWord.length
- 1 <= wordList.length <= 5000
- wordList 中的所有单词长度与 beginWord 相同
- beginWord、endWord 和 wordList 中的单词均由小写英文字母组成

