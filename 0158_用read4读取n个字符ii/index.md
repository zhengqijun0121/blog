# 力扣158. 用 Read4 读取 N 个字符 II


## 力扣158. Read N Characters Given Read4 II - Call Multiple Times（用 Read4 读取 N 个字符 II）

与 157 题类似，但此题的 read 方法可能会被多次调用。你需要确保多次调用之间能正确读取后续字符。此题是会员题。

示例 1：

![](../posts/01_学习/87_LeetCode/0158_用Read4读取N个字符II/img/0158-1-description.png)

```
输入：file = "abc", n = 4
输出：3
```

示例 2：

![](../posts/01_学习/87_LeetCode/0158_用Read4读取N个字符II/img/0158-2-description.png)

```
输入：file = "abcde", n = [1, 4]
输出：[1, 4]
解释：第一次调用 read(buf, 1) 读取 'a'；第二次调用 read(buf, 4) 读取 'bcde'。
```

示例 3：

![](../posts/01_学习/87_LeetCode/0158_用Read4读取N个字符II/img/0158-3-description.png)

```
输入：file = "abcABCD1234", n = [5, 7]
输出：[5, 7]
```

提示：
- 1 <= n <= 1000
- read 函数可能会被多次调用

