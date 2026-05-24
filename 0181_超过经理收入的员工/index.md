# 力扣181. Employees Earning More Than Their Managers（超过经理收入的员工）


## 力扣181. Employees Earning More Than Their Managers（超过经理收入的员工）

编写 SQL 查询，找出所有收入比他们的经理多的员工。表：Employee（id, name, salary, managerId）。

示例 1：

![](../posts/01_学习/87_LeetCode/0181_超过经理收入的员工/img/0181-1-description.png)

```
输入：
Employee table:
| id | name  | salary | managerId |
|----|-------|--------|-----------|
| 1  | Joe   | 70000  | 3         |
| 2  | Henry | 80000  | 4         |
| 3  | Sam   | 60000  | null      |
| 4  | Max   | 90000  | null      |
输出：
| Employee |
|----------|
| Joe      |
```

提示：
- Employee 表包含所有员工及其上级的 id

