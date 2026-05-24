# 力扣150. Evaluate Reverse Polish Notation（逆波兰表达式求值）


## 力扣150. Evaluate Reverse Polish Notation（逆波兰表达式求值）

给你一个字符串数组 tokens，表示一个根据逆波兰表示法表示的算术表达式。请你计算该表达式并返回一个整数。有效的运算符为 '+'、'-'、'*' 和 '/' 。

示例 1：

![](../posts/01_学习/87_LeetCode/0150_逆波兰表达式求值/img/0150-1-description.png)

```
输入：tokens = ["2","1","+","3","*"]
输出：9
解释：((2 + 1) * 3) = 9
```

示例 2：

![](../posts/01_学习/87_LeetCode/0150_逆波兰表达式求值/img/0150-2-description.png)

```
输入：tokens = ["4","13","5","/","+"]
输出：6
解释：(4 + (13 / 5)) = 6
```

提示：
- 1 <= tokens.length <= 10^4
- tokens[i] 是一个算数运算符（"+"、"-"、"*" 或 "/"），或范围在 [-200, 200] 内的一个整数

