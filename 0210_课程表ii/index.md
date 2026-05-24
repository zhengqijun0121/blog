# 力扣210. 课程表 II


## 力扣210. Course Schedule II（课程表 II）

给定 numCourses 和 prerequisites，返回学完所有课程所安排的学习顺序。

示例 1：

![](../posts/01_学习/87_LeetCode/0210_课程表II/img/0210-1-description.png)

```
输入：numCourses = 2, prerequisites = [[1,0]]
输出：[0,1]
解释：总共有 2 门课程。要学习课程 1，你需要先完成课程 0。因此，正确的课程顺序为 [0,1]。
```

提示：
- 1 <= numCourses <= 2000
- 0 <= prerequisites.length <= numCourses * (numCourses - 1) / 2
- prerequisites[i].length == 2
- 0 <= ai, bi < numCourses
- ai != bi
- 所有 [ai, bi] 互不相同

