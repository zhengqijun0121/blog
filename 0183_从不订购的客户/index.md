# 力扣183. Customers Who Never Order（从不订购的客户）


## 力扣183. Customers Who Never Order（从不订购的客户）

编写 SQL 查询，找出所有从不订购任何东西的客户。表：Customers（id, name）和 Orders（id, customerId）。

示例 1：

![](../posts/01_学习/87_LeetCode/0183_从不订购的客户/img/0183-1-description.png)

```
输入：
Customers table:
| id | name  |
|----|-------|
| 1  | Joe   |
| 2  | Henry |
| 3  | Sam   |
| 4  | Max   |
Orders table:
| id | customerId |
|----|------------|
| 1  | 3          |
| 2  | 1          |
输出：
| Customers |
|-----------|
| Henry     |
| Max       |
```

提示：
- Customers 表包含所有客户信息
- Orders 表包含所有订单及其客户 id

