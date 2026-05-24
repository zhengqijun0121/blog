# 力扣177. Nth Highest Salary（第 N 高的薪水）


## 力扣177. Nth Highest Salary（第 N 高的薪水）

编写 SQL 查询，查找 `Employee` 表中第 `n` 高的薪水。如果不存在第 `n` 高的薪水，返回 `null`。

表结构：

**Employee**
| Column Name | Type |
|-------------|------|
| id          | int  |
| salary      | int  |

id 是主键。

示例 1：

![](../posts/01_学习/87_LeetCode/0177_第N高的薪水/img/0177-1-description.png)

```
输入：
Employee 表：
| id | salary |
|----|--------|
| 1  | 100    |
| 2  | 200    |
| 3  | 300    |

n = 2

输出：
| getNthHighestSalary(2) |
|------------------------|
| 200                    |
```

