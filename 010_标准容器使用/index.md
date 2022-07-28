# C++11 标准容器使用


## 容器的定义

在数据存储上，有一种对象类型，它可以持有其它对象或指向其它对像的指针，这种对象类型就叫做容器。

----

## 容器的分类

1. 顺序容器

顺序容器是一种各元素之间有顺序关系的线性表，是一种线性结构的有序集合。这种顺序不依赖于元素的值，而是与元素加入容器的顺序相对应。顺序容器提供控制元素存储和访问顺序的能力。

顺序容器包括`vector`、`dequeue`、`list`、`forward_list`、`array`、`string`。

2. 关联容器

关联式容器是非线性的树结构，各元素之间没有严格的物理上的顺序关系，也就是说元素在容器中并没有保存元素置入容器时的逻辑顺序。但是关联式容器提供了另一种根据元素特点排序的功能。

有序关联容器包括：`set`、`multiset`、`map`、`multimap`。

无序关联容器包括：`unordered_map`、`unordered_multimap`、`unordered_set`、`unordered_multiset`。

3. 容器适配器

适配器是使一种不同的行为类似于另一事物的行为的一种机制。容器适配器让一种已存在的容器类型采用另一种不同的抽象类型的工作方式实现。适配器是容器的接口，它本身不能直接保存元素，它保存元素的机制是调用另一种顺序容器去实现。

容器适配器包括`stack`、`queue`、`priority_queue`。

----

## array

### 定义

数组是固定大小的序列容器，不通过分配器管理其元素的分配。因此不能动态的扩容和收缩。元素存储在连续的内存位置，允许对元素进行恒定时间随机访问。

原型如下：

```cpp
template <class T, size_t N> class array;
```

### 使用

1. 头文件

```cpp
#include <array>
```

2. 新增操作

`array` 在初始化的时候已经定义好数组大小，不能新增元素！

3. 删除操作

`array` 在初始化的时候已经定义好数组大小，不能删除元素！

4. 修改操作

```cpp
std::array<int, 10> arr;

arr[0] = 1;
arr.at(1) = 2;
std::get<2>(arr) = 3;
```

5. 查找操作

```cpp
if (arr[3] == 4) {
    ...
}
```

6. 遍历操作

- 按元素大小遍历

```cpp
for (size_t i = 0; i < arr.size(); ++i) {
    std::cout << arr[i] << std::endl;
}
```

- 迭代器遍历

```cpp
for (const auto& it : arr) {
    std::cout << *it << std::endl;
}
// or
for (auto it = arr.begin(); it != arr.end(); ++it) {
    std::cout << *it << std::endl;
}
```

----

## string

### 定义

`string` 表示可变长的字符序列。

原型如下：

```cpp
typedef basic_string<char> string;
```

### 使用

1. 头文件

```cpp
#include <string>
```

2. 新增操作

```cpp
std::string str("");

str += "Hello";
str.append("World");
str.push_back('\0');  // 新增'\0'字符
```

3. 删除操作

```cpp
str.erase(5, 3);  // 删除第5个字符往后3个
str.erase(str.begin() + 3);  // 删除第4个字符
str.erase(str.begin(), str.end() - 3);  // 删除开始到最后3个字符
str.clear();  // 清空字符
str.pop_back();  // 删除最后1个字符
```


4. 修改操作

```cpp
str.replace(3, 4, "just");
str.replace(4, 1, 3, '!');
str.replace(str.begin(), str.end() - 3, str2);
str = "Hello";
str[0] = 'W';
str.at(1) = 'S';
```

5. 查找操作

```cpp
std::size_t found = str.find(str2);
if (found != std::string::npos) {
    std::cout << "first 'needle' found at: " << found << std::endl;
}
```

6. 遍历操作

```cpp
std::cout << str.c_str() << std::endl;
std::cout << str.data() << std::endl;
```

7. 字符串转换

- string 转 char*

```cpp
str.c_str() or str.data();
printf("%s", str.c_str());
```

- char* 转 string

```cpp
std::string str = "Hello";
```

- string 转 int

```cpp
std::string str = "123";
int num = atoi(str.c_str());  // 123
```

- int 转 string

```cpp
std::cout << std::to_string(123) << std::endl;  // "123"
```

- stringstream 转 string

```cpp
std::stringstream ss;
ss << 123;
std::cout << ss.str() << std::endl;  // 123
```

8. 其他操作

```cpp
std::cout << str.size() << std::endl;  // 长度
std::cout << str.length() << std::endl;  // 长度

if (str1.compare(str2) != 0) {  // 字符串比较
    std::cout << str1 << " is not " << str2 << std::endl;
}

std::cout << str.substr() << std::endl;  // 子串
```

----

## vector

### 定义




### 使用







----


