# Markdown 语法详解


## 简单介绍

> Markdown 是一种轻量级标记语言，创始人为约翰·格鲁伯。它允许人们使用易读易写的纯文本格式编写文档，然后转换成有效的 XHTML 文档。 -- [维基百科](https://zh.wikipedia.org/zh-hans/Markdown)

----

## 基本语法

### 标题

```markdown
# 一级标题
## 二级标题
### 三级标题
#### 四级标题
##### 五级标题
###### 六级标题
```

### 段落

使用空行进行段落分割

### 字体设置

```markdown
*斜体*
_斜体_
**粗体**
__粗体__
***粗斜体***
___粗斜体___
~~删除线~~
==高亮==
<u>下划线</u>
```

### 上标下标

```markdown
上^标^
下~标~
```

### 注释

```markdown
<!--注释-->
```

### 脚注

```markdown
[^1]
[^1]: Markdown
```

### 链接

- 地址链接

```markdown
[描述](路径)
<链接路径>
```

- 图片链接

```markdown
![描述](路径)
```

### 分割线

```markdown
****
----
____
```

### 代码块

{{< highlight markdown >}}
```language
code
```
{{< / highlight >}}

### 行内代码

```markdown
`code`
```

### 引用

```markdown
> 引用的文字
>> 嵌套的引用
```

### 列表

- 无序列表

```markdown
- list1
+ list2
* list3
```

- 有序列表

```markdown
1. list1
2. list2
3. list3
```

- 混合列表

```markdown
- 1. list1
+ 2. list2
* 3. list3
```

- 嵌套列表

```markdown
1. list1
	- list11
	- list12
```

- 任务列表

```markdown
- [ ] 未完成
- [x] 已完成
```

### 表格

```markdown
| head | head |
| ---- | ---- |
| cell | cell |
| cell | cell |

:-  设置内容和标题栏居左对齐
:-: 设置内容和标题栏居中对齐
-:  设置内容和标题栏居右对齐
```

----

## 博客

因为 `Markdown` 使用十分简单，因此决定用 `Markdown` 整理笔记。将 `Markdown` 基本语法整理出来，仅作参考！


