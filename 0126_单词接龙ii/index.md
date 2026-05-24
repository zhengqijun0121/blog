# 力扣126. Word Ladder II（单词接龙 II）


## 力扣126. Word Ladder II（单词接龙 II）

按字典 wordList 完成从单词 beginWord 到单词 endWord 的转换。一个转换序列是每个相邻单词仅一个字母不同的序列。返回所有从 beginWord 到 endWord 的最短转换序列。所有单词长度相同，且由小写字母组成。

示例 1：

![](../posts/01_学习/87_LeetCode/0126_单词接龙II/img/0126-1-description.png)

```
输入：beginWord = "hit", endWord = "cog", wordList = ["hot","dot","dog","lot","log","cog"]
输出：[["hit","hot","dot","dog","cog"],["hit","hot","lot","log","cog"]]
解释：存在两个最短转换序列："hit" -> "hot" -> "dot" -> "dog" -> "cog" 和 "hit" -> "hot" -> "lot" -> "log" -> "cog"，长度均为 5。
```

提示：
- 1 <= beginWord.length <= 5
- endWord.length == beginWord.length
- 1 <= wordList.length <= 500
- wordList 中的所有单词长度与 beginWord 相同
- beginWord、endWord 和 wordList 中的单词均由小写英文字母组成

