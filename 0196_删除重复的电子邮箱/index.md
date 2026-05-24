# 力扣196. 删除重复的电子邮箱


## 力扣196. Delete Duplicate Emails（删除重复的电子邮箱）

编写 SQL 查询，删除 Person 表中所有重复的电子邮箱，重复的邮箱里只保留 id 最小的那个。表：Person（id, email）。

示例 1：

![](../posts/01_学习/87_LeetCode/0196_删除重复的电子邮箱/img/0196-1-description.png)

```
输入：
Person table:
| id | email            |
|----|------------------|
| 1  | john@example.com |
| 2  | bob@example.com  |
| 3  | john@example.com |
输出（删除后）：
| id | email            |
|----|------------------|
| 1  | john@example.com |
| 2  | bob@example.com  |
```

提示：
- 删除操作后，查询 Person 表返回剩余记录
- 注意是删除操作，而不是 select

