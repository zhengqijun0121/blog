# 力扣175. 组合两个表


## 力扣175. Combine Two Tables（组合两个表）

编写 SQL 查询，报告所有 Person 的 `firstName`、`lastName`、`city`、`state`，无论 Person 是否有地址信息。

表结构：

**Person**
| Column Name | Type |
|-------------|------|
| personId    | int  |
| lastName    | varchar |
| firstName   | varchar |

personId 是主键。

**Address**
| Column Name | Type |
|-------------|------|
| addressId   | int  |
| personId    | int  |
| city        | varchar |
| state       | varchar |

addressId 是主键。

示例 1：

![](../posts/01_学习/87_LeetCode/0175_组合两个表/img/0175-1-description.png)

```
输入：
Person 表：
| personId | lastName | firstName |
|----------|----------|-----------|
| 1        | Wang     | Allen     |

Address 表：
| addressId | personId | city          | state    |
|-----------|----------|---------------|----------|
| 1         | 2        | New York City | New York |

输出：
| lastName | firstName | city          | state    |
|----------|-----------|---------------|----------|
| Allen    | Wang      | New York City | New York |
```

