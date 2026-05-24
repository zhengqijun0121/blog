# 力扣157. Read N Characters Given Read4（用 Read4 读取 N 个字符）


## 力扣157. Read N Characters Given Read4（用 Read4 读取 N 个字符）

给你一个文件，并且该文件只能通过给定的 read4 方法来读取，请实现一个方法使其能够读取 n 个字符。read4 方法可以从文件中读取 4 个字符，并将它们存入缓存数组 buf 中。此题是会员题。

示例 1：

![](../posts/01_学习/87_LeetCode/0157_用Read4读取N个字符/img/0157-1-description.png)

```
输入：file = "abc", n = 4
输出：3
解释：执行 read(buf, 4) 后，buf 中包含 "abc"，文件共 3 个字符，因此返回 3。
```

示例 2：

![](../posts/01_学习/87_LeetCode/0157_用Read4读取N个字符/img/0157-2-description.png)

```
输入：file = "abcde", n = 5
输出：5
解释：执行 read(buf, 5) 后，buf 中包含 "abcde"，文件共 5 个字符，因此返回 5。
```

示例 3：

![](../posts/01_学习/87_LeetCode/0157_用Read4读取N个字符/img/0157-3-description.png)

```
输入：file = "abcdABCD1234", n = 12
输出：12
解释：执行 read(buf, 12) 后，buf 中包含 "abcdABCD1234"，文件共 12 个字符，因此返回 12。
```

提示：
- 1 <= n <= 1000
- 你只能通过调用 read4 方法来读取文件

