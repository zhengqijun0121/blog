# 力扣155. Min Stack（最小栈）


## 力扣155. Min Stack（最小栈）

设计一个支持 push、pop、top 操作，并能在常数时间内检索到最小元素的栈。

实现 MinStack 类：
- MinStack() 初始化堆栈对象。
- void push(int val) 将元素 val 推入堆栈。
- void pop() 删除堆栈顶部的元素。
- int top() 获取堆栈顶部的元素。
- int getMin() 获取堆栈中的最小元素。

示例 1：

![](../posts/01_学习/87_LeetCode/0155_最小栈/img/0155-1-description.png)

```
输入：
["MinStack","push","push","push","getMin","pop","top","getMin"]
[[],[-2],[0],[-3],[],[],[],[]]

输出：
[null,null,null,null,-3,null,0,-2]

解释：
MinStack minStack = new MinStack();
minStack.push(-2);
minStack.push(0);
minStack.push(-3);
minStack.getMin();   --> 返回 -3.
minStack.pop();
minStack.top();      --> 返回 0.
minStack.getMin();   --> 返回 -2.
```

提示：
- -2^31 <= val <= 2^31 - 1
- pop、top 和 getMin 操作总是在非空栈上调用
- 最多调用 3 * 10^4 次 push、pop、top 和 getMin

