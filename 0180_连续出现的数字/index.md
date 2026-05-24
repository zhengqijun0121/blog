# 力扣180. 连续出现的数字


## 力扣180. Consecutive Numbers（连续出现的数字）

编写 SQL 查询，查找 `Logs` 表中所有至少连续出现三次的数字。

表结构：

**Logs**
| Column Name | Type |
|-------------|------|
| id          | int  |
| num         | int  |

id 是主键。

示例 1：

![](../posts/01_学习/87_LeetCode/0180_连续出现的数字/img/0180-1-description.png)

```
输入：
Logs 表：
| id | num |
|----|-----|
| 1  | 1   |
| 2  | 1   |
| 3  | 1   |
| 4  | 2   |
| 5  | 1   |
| 6  | 2   |
| 7  | 2   |

输出：
| ConsecutiveNums |
|-----------------|
| 1               |
```

