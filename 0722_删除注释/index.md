# 力扣722. Remove Comments（删除注释）


## 力扣722. Remove Comments（删除注释）

删除 C++ 代码中的注释。

示例 1：

![](../posts/01_学习/87_LeetCode/0722_删除注释/img/0722-1-description.png)

```
输入：source = ["/*Test program */", "int main()", "{ ", "  // variable declaration ", "int a, b, c;", "/* This is a test", "   multiline  ", "   comment for ", "   testing */", "a = b + c;", "}"]
输出：["int main()","{ ","  ","int a, b, c;","a = b + c;","}"]
解释：...
```

提示：
- 1 <= source.length <= 100
- 0 <= source[i].length <= 80

