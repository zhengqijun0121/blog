# 力扣207. Course Schedule（课程表）


## 力扣207. Course Schedule（课程表）

你这个学期必须选修 numCourses 门课程。先修关系用 prerequisites 表示，判断是否可能完成所有课程的学习。

示例 1：

![](../posts/01_学习/87_LeetCode/0207_课程表/img/0207-1-description.png)

```
输入：numCourses = 2, prerequisites = [[1,0]]
输出：true
解释：总共有 2 门课程。学习课程 1 之前需要先完成课程 0。这是可能的。
```

提示：
- 1 <= numCourses <= 2000
- 0 <= prerequisites.length <= 5000
- prerequisites[i].length == 2
- 0 <= ai, bi < numCourses
- prerequisites[i] 中的所有课程对互不相同

