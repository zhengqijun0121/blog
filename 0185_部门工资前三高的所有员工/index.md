# 力扣185. Department Top Three Salaries（部门工资前三高的所有员工）


## 力扣185. Department Top Three Salaries（部门工资前三高的所有员工）

编写 SQL 查询，找出每个部门中薪资排名前三的员工。表：Employee（id, name, salary, departmentId）和 Department（id, name）。

示例 1：

![](../posts/01_学习/87_LeetCode/0185_部门工资前三高的所有员工/img/0185-1-description.png)

```
输入：
Employee table:
| id | name  | salary | departmentId |
|----|-------|--------|--------------|
| 1  | Joe   | 85000  | 1            |
| 2  | Henry | 80000  | 2            |
| 3  | Sam   | 60000  | 2            |
| 4  | Max   | 90000  | 1            |
| 5  | Janet | 69000  | 1            |
| 6  | Randy | 85000  | 1            |
| 7  | Will  | 70000  | 1            |
Department table:
| id | name  |
|----|-------|
| 1  | IT    |
| 2  | Sales |
输出：
| Department | Employee | Salary |
|------------|----------|--------|
| IT         | Max      | 90000  |
| IT         | Joe      | 85000  |
| IT         | Randy    | 85000  |
| IT         | Will     | 70000  |
| Sales      | Henry    | 80000  |
| Sales      | Sam      | 60000  |
```

提示：
- 返回结果按部门名称升序排列，部门内按薪资降序排列
- 如果并列排名，不会导致人数超过三人

