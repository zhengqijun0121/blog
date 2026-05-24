# 力扣550. Game Play Analysis IV（游戏玩法分析 IV）


## 力扣550. Game Play Analysis IV（游戏玩法分析 IV）

SQL：报告在首次登录后第二天再次登录的玩家比例，四舍五入到小数点后两位。

表：Activity（player_id, device_id, event_date, games_played）

示例：

```
Activity:
player_id | device_id | event_date | games_played
1         | 2         | 2016-03-01 | 5
1         | 2         | 2016-03-02 | 6
2         | 3         | 2017-06-25 | 1
3         | 1         | 2016-03-02 | 0
3         | 4         | 2018-07-03 | 5

输出：| fraction |
        0.33
```

