# 力扣68. Text Justification（文本左右对齐）


## 力扣68. Text Justification（文本左右对齐）

给定一个单词数组 `words` 和一个长度 `maxWidth`，重新排版单词，使其成为每行恰好有 `maxWidth` 个字符、且左右两端对齐的文本。

你应该使用「贪心算法」来放置给定的单词；也就是说，尽可能多地往每行中放置单词。必要时可用空格 `' '` 填充，使得每行恰好有 `maxWidth` 个字符。

要求尽可能均匀分配单词间的空格数量。如果某一行单词间的空格不能均匀分配，则左侧放置的空格数要多于右侧的空格数。

文本的最后一行应为左对齐，且单词之间不插入额外的空格。

示例 1：

![](../posts/01_学习/87_LeetCode/0068_文本左右对齐/img/0068-1-description.png)

```
输入：words = ["This", "is", "an", "example", "of", "text", "justification."], maxWidth = 16
输出：
[
   "This    is    an",
   "example  of text",
   "justification.  "
]
```

示例 2：

![](../posts/01_学习/87_LeetCode/0068_文本左右对齐/img/0068-2-description.png)

```
输入：words = ["What","must","be","acknowledgment","shall","be"], maxWidth = 16
输出：
[
  "What   must   be",
  "acknowledgment  ",
  "shall be        "
]
解释：注意最后一行的格式应为 "shall be    "，而不是 "shall     be"。
     因为最后一行应为左对齐，而不是左右两端对齐。
     第二行同样为左对齐，这是因为这行只包含一个单词。
```

提示：
- 1 <= words.length <= 300
- 1 <= words[i].length <= 20
- words[i] 由小写英文字母和符号组成
- 1 <= maxWidth <= 100
- words[i].length <= maxWidth

