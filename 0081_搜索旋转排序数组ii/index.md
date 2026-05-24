# 力扣81. 搜索旋转排序数组 II


## 力扣81. Search in Rotated Sorted Array II（搜索旋转排序数组 II）

已知存在一个按非降序排列的整数数组 nums，在传递给函数之前，nums 在预先未知的某个下标上进行了旋转。编写一个函数来判断给定的目标值是否存在于数组中。若存在返回 true，否则返回 false。本题中的 nums 可能包含重复元素。

示例 1：

![](../posts/01_学习/87_LeetCode/0081_搜索旋转排序数组II/img/0081-1-description.png)

```
输入：nums = [2,5,6,0,0,1,2], target = 0
输出：true
```

示例 2：

![](../posts/01_学习/87_LeetCode/0081_搜索旋转排序数组II/img/0081-2-description.png)

```
输入：nums = [2,5,6,0,0,1,2], target = 3
输出：false
```

提示：
- 1 <= nums.length <= 5000
- -10^4 <= nums[i] <= 10^4

