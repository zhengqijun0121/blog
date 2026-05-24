# 力扣34. 在排序数组中查找元素的第一个和最后一个位置


## 力扣34. Find First and Last Position of Element in Sorted Array（在排序数组中查找元素的第一个和最后一个位置）

给你一个按照非递减顺序排列的整数数组 nums，和一个目标值 target。请你找出给定目标值在数组中的开始位置和结束位置。如果数组中不存在目标值 target，返回 [-1, -1]。你必须设计并实现时间复杂度为 O(log n) 的算法解决此问题。

示例 1：

![](../posts/01_学习/87_LeetCode/0034_在排序数组中查找元素的第一个和最后一个位置/img/0034-1-description.png)

```
输入：nums = [5,7,7,8,8,10], target = 8
输出：[3,4]
```

示例 2：

![](../posts/01_学习/87_LeetCode/0034_在排序数组中查找元素的第一个和最后一个位置/img/0034-2-description.png)

```
输入：nums = [5,7,7,8,8,10], target = 6
输出：[-1,-1]
```

示例 3：

![](../posts/01_学习/87_LeetCode/0034_在排序数组中查找元素的第一个和最后一个位置/img/0034-3-description.png)

```
输入：nums = [], target = 0
输出：[-1,-1]
```

提示：
- 0 <= nums.length <= 10^5
- -10^9 <= nums[i] <= 10^9
- nums 是一个非递减数组
- -10^9 <= target <= 10^9

