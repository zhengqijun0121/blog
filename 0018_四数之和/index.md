# 力扣18. 4Sum（四数之和）


## 力扣18. 4Sum（四数之和）

给你一个由 n 个整数组成的数组 nums，和一个目标值 target。请你找出并返回满足下述全部条件且不重复的四元组 [nums[a], nums[b], nums[c], nums[d]]。a、b、c、d 互不相同，且 nums[a] + nums[b] + nums[c] + nums[d] == target。答案中不可以包含重复的四元组。

示例 1：

![](../posts/01_学习/87_LeetCode/0018_四数之和/img/0018-1-description.png)

```
输入：nums = [1,0,-1,0,-2,2], target = 0
输出：[[-2,-1,1,2],[-2,0,0,2],[-1,0,0,1]]
```

示例 2：

![](../posts/01_学习/87_LeetCode/0018_四数之和/img/0018-2-description.png)

```
输入：nums = [2,2,2,2,2], target = 8
输出：[[2,2,2,2]]
```

提示：
- 1 <= nums.length <= 200
- -10^9 <= nums[i] <= 10^9
- -10^9 <= target <= 10^9

