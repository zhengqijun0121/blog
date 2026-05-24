# 力扣167. 两数之和 II - 输入有序数组


## 力扣167. Two Sum II - Input Array Is Sorted（两数之和 II - 输入有序数组）

给你一个下标从 1 开始的整数数组 numbers，该数组已按非递减顺序排列，请你从数组中找出满足相加之和等于目标数 target 的两个数。

返回这两个数的下标（从 1 开始）。你可以假设每个输入只对应唯一的答案，而且你不可以重复使用相同的元素。

示例 1：

![](../posts/01_学习/87_LeetCode/0167_两数之和II/img/0167-1-description.png)

```
输入：numbers = [2,7,11,15], target = 9
输出：[1,2]
解释：2 与 7 之和等于目标数 9，因此 index1 = 1, index2 = 2。
```

示例 2：

![](../posts/01_学习/87_LeetCode/0167_两数之和II/img/0167-2-description.png)

```
输入：numbers = [2,3,4], target = 6
输出：[1,3]
```

示例 3：

![](../posts/01_学习/87_LeetCode/0167_两数之和II/img/0167-3-description.png)

```
输入：numbers = [-1,0], target = -1
输出：[1,2]
```

提示：

- 2 <= numbers.length <= 3 * 10^4
- -1000 <= numbers[i] <= 1000
- numbers 按非递减顺序排列
- -1000 <= target <= 1000
- 仅存在一个有效答案

