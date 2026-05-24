# 力扣241. 为运算表达式设计优先级


## 力扣241. Different Ways to Add Parentheses（为运算表达式设计优先级）

给你一个由数字和运算符组成的字符串 expression，按不同优先级为表达式加括号并返回所有可能的结果。

示例 1：

![](../posts/01_学习/87_LeetCode/0241_为运算表达式设计优先级/img/0241-1-description.png)

```
输入：expression = "2-1-1"
输出：[0,2]
解释：((2-1)-1) = 0, (2-(1-1)) = 2
```

提示：
- 1 <= expression.length <= 20
- expression 由数字和运算符（'+'、'-'、'*'）组成
- expression 中的所有整数都在 [0, 99] 范围内

