# 力扣33. Search in Rotated Sorted Array（搜索旋转排序数组）


## 力扣33. Search in Rotated Sorted Array（搜索旋转排序数组）

整数数组 nums 按升序排列，数组中的值互不相同。在传递给函数之前，nums 在预先未知的某个下标 k（0 <= k < nums.length）上进行了旋转。给你旋转后的数组 nums 和一个整数 target，如果 nums 中存在这个目标值 target，则返回它的下标，否则返回 -1。你必须设计一个时间复杂度为 O(log n) 的算法解决此问题。

示例 1：

![](../posts/01_学习/87_LeetCode/0033_搜索旋转排序数组/img/0033-1-description.png)

```
输入：nums = [4,5,6,7,0,1,2], target = 0
输出：4
```

示例 2：

![](../posts/01_学习/87_LeetCode/0033_搜索旋转排序数组/img/0033-2-description.png)

```
输入：nums = [4,5,6,7,0,1,2], target = 3
输出：-1
```

示例 3：

![](../posts/01_学习/87_LeetCode/0033_搜索旋转排序数组/img/0033-3-description.png)

```
输入：nums = [1], target = 0
输出：-1
```

提示：
- 1 <= nums.length <= 5000
- -10^4 <= nums[i] <= 10^4
- nums 中的每个值都独一无二
- nums 在某个未知的下标上进行了旋转
- -10^4 <= target <= 10^4

