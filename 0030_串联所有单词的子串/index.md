# 力扣30. 串联所有单词的子串


## 力扣30. Substring with Concatenation of All Words（串联所有单词的子串）

给定一个字符串 s 和一个字符串数组 words。words 中所有字符串长度相同。s 中的串联子串是指一个包含 words 中所有字符串以任意顺序排列连接起来的子串。例如，如果 words = ["ab","cd","ef"]，那么 "abcdef"、"abefcd"、"cdabef"、"cdefab"、"efabcd" 和 "efcdab" 都是串联子串。返回所有串联子串在 s 中的开始索引。

示例 1：

![](../posts/01_学习/87_LeetCode/0030_串联所有单词的子串/img/0030-1-description.png)

```
输入：s = "barfoothefoobarman", words = ["foo","bar"]
输出：[0,9]
解释：从索引 0 和 9 开始的子串分别是 "barfoo" 和 "foobar"，它们都是 words 中所有单词以任意顺序排列连接起来的串联子串。
```

示例 2：

![](../posts/01_学习/87_LeetCode/0030_串联所有单词的子串/img/0030-2-description.png)

```
输入：s = "wordgoodgoodgoodbestword", words = ["word","good","best","word"]
输出：[]
解释：words 中每个单词只能使用一次，"wordgoodgoodgoodbestword" 中不存在包含所有单词的串联子串。
```

示例 3：

![](../posts/01_学习/87_LeetCode/0030_串联所有单词的子串/img/0030-3-description.png)

```
输入：s = "barfoofoobarthefoobarman", words = ["bar","foo","the"]
输出：[6,9,12]
解释：从索引 6、9 和 12 开始的子串分别是 "foobarthe"、"barthefoo" 和 "thefoobar"，它们都是 words 中所有单词以任意顺序排列连接起来的串联子串。
```

提示：
- 1 <= s.length <= 10^4
- 1 <= words.length <= 5000
- 1 <= words[i].length <= 30
- s 和 words[i] 仅由小写英文字母组成

