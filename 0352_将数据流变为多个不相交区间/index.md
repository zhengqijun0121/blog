# 力扣352. 将数据流变为多个不相交区间


## 力扣352. 将数据流变为多个不相交区间

给你一个由非负整数组成的数据流输入 `a1, a2, ..., an`，请你将目前为止看到的数字汇总为一组不相交的区间列表。

实现 `SummaryRanges` 类：



- `SummaryRanges()` 初始化一个空的数据流对象。

- `void addNum(int value)` 将整数 `value` 添加到数据流中。

- `int[][] getIntervals()` 返回当前数据流中的整数汇总为一组不相交的区间列表 `[starti, endi]`。答案应按 `starti` 升序排序。

**示例 1：**


```
输入
["SummaryRanges", "addNum", "getIntervals", "addNum", "getIntervals", "addNum", "getIntervals", "addNum", "getIntervals", "addNum", "getIntervals"]
[[], [1], [], [3], [], [7], [], [2], [], [6], []]
输出
[null, null, [[1, 1]], null, [[1, 1], [3, 3]], null, [[1, 1], [3, 3], [7, 7]], null, [[1, 3], [7, 7]], null, [[1, 3], [6, 7]]]

解释
SummaryRanges summaryRanges = new SummaryRanges();
summaryRanges.addNum(1);      // arr = [1]
summaryRanges.getIntervals(); // 返回 [[1, 1]]
summaryRanges.addNum(3);      // arr = [1, 3]
summaryRanges.getIntervals(); // 返回 [[1, 1], [3, 3]]
summaryRanges.addNum(7);      // arr = [1, 3, 7]
summaryRanges.getIntervals(); // 返回 [[1, 1], [3, 3], [7, 7]]
summaryRanges.addNum(2);      // arr = [1, 2, 3, 7]
summaryRanges.getIntervals(); // 返回 [[1, 3], [7, 7]]
summaryRanges.addNum(6);      // arr = [1, 2, 3, 6, 7]
summaryRanges.getIntervals(); // 返回 [[1, 3], [6, 7]]
```


**提示：**



- `0 <= value <= 10^{4}`

- 最多会调用 `addNum` 和 `getIntervals` 方法 `3 * 10^{4}` 次。

- 最多会调用 `getIntervals` 方法 `10^{2}` 次。

**进阶：**如果存在大量合并，并且与数据流的大小相比，不相交区间的数量很小，该怎么办?

