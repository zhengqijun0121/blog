# 力扣197. Rising Temperature（上升的温度）


## 力扣197. Rising Temperature（上升的温度）

编写 SQL 查询，找出与之前（昨天）日期相比温度更高的所有日期的 id。表：Weather（id, recordDate, temperature）。

示例 1：

![](../posts/01_学习/87_LeetCode/0197_上升的温度/img/0197-1-description.png)

```
输入：
Weather table:
| id | recordDate | temperature |
|----|------------|-------------|
| 1  | 2015-01-01 | 10          |
| 2  | 2015-01-02 | 25          |
| 3  | 2015-01-03 | 20          |
| 4  | 2015-01-04 | 30          |
输出：
| id |
|----|
| 2  |
| 4  |
```

提示：
- recordDate 是日期类型且每个日期唯一
- 需要比较当天与前一天的温度

