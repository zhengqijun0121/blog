# 力扣184. Department Highest Salary（部门工资最高的员工）


## 力扣184. Department Highest Salary（部门工资最高的员工）

编写 SQL 查询，找出每个部门中薪资最高的员工。表：Employee（id, name, salary, departmentId）和 Department（id, name）。

示例 1：

![](../posts/01_学习/87_LeetCode/0184_部门工资最高的员工/img/0184-1-description.png)

```
输入：
Employee table:
| id | name  | salary | departmentId |
|----|-------|--------|--------------|
| 1  | Joe   | 70000  | 1            |
| 2  | Jim   | 90000  | 1            |
| 3  | Henry | 80000  | 2            |
Department table:
| id | name  |
|----|-------|
| 1  | IT    |
| 2  | Sales |
输出：
| Department | Employee | Salary |
|------------|----------|--------|
| IT         | Jim      | 90000  |
| Sales      | Henry    | 80000  |
```

提示：
- 每个部门可能有多名员工同时拥有最高薪资

