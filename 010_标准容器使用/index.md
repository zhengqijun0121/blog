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

**API 定义**

```cpp
template <class T, size_t N>
class array {
public:
    // Iterators
    iterator begin() noexcept;
    const_iterator begin() const noexcept;
    iterator end() noexcept;
    const_iterator end() const noexcept;
    reverse_iterator rbegin() noexcept;
    const_reverse_iterator rbegin() const noexcept;
    reverse_iterator rend() noexcept;
    const_reverse_iterator rend() const noexcept;
    const_iterator cbegin() const noexcept;
    const_iterator cend() const noexcept;
    const_reverse_iterator crbegin() const noexcept;
    const_reverse_iterator crend() const noexcept;

    // Capacity
    constexpr size_type size() noexcept;
    constexpr size_type max_size() noexcept;
    constexpr bool empty() noexcept;

    // Element Access
    reference operator[](size_type n);
    const_reference operator[](size_type n) const;
    reference at(size_type n);
    const_reference at(size_type n) const;
    reference front();
    const_reference front() const;
    reference back();
    const_reference back() const;
    value_type* data() noexcept;
    const value_type* data() const noexcept;

    // Modifiers
    void fill(const value_type& val);
    void swap(array& x) noexcept(noexcept(swap(declval<value_type&>(), declval<value_type&>())));
};
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

**API 定义**

```cpp
class string {
public:
    string();
    string(const string& str);	
    string(const string& str, size_t pos, size_t len = npos);
    string(const char* s);
    string(const char* s, size_t n);
    string(size_t n, char c);
    template <class InputIterator> string(InputIterator first, InputIterator last);
    string(initializer_list<char> il);
    string(string&& str) noexcept;
    ~string();
    string& operator=(const string& str);
    string& operator=(const char* s);
    string& operator=(char c);
    string& operator=(initializer_list<char> il);
    string& operator=(string&& str) noexcept;

    // Iterators
    iterator begin() noexcept;
    const_iterator begin() const noexcept;
    iterator end() noexcept;
    const_iterator end() const noexcept;
    reverse_iterator rbegin() noexcept;
    const_reverse_iterator rbegin() const noexcept;
    reverse_iterator rend() noexcept;
    const_reverse_iterator rend() const noexcept;
    const_iterator cbegin() const noexcept;
    const_iterator cend() const noexcept;
    const_reverse_iterator crbegin() const noexcept;
    const_reverse_iterator crend() const noexcept;

    // Capacity
    size_t size() const noexcept;
    size_t length() const noexcept;
    size_t max_size() const noexcept;
    void resize(size_t n);
    void resize(size_t n, char c);
    size_t capacity() const noexcept;
    void reserve(size_t n = 0);
    void clear() noexcept;
    bool empty() const noexcept;
    void shrink_to_fit();

    // Element Access
    char& operator[](size_t pos);
    const char& operator[](size_t pos) const;
    char& at(size_t pos);
    const char& at(size_t pos) const;
    char& back();
    const char& back() const;
    char& front();
    const char& front() const;

    // Modifiers
    string& operator+=(const string& str);
    string& operator+=(const char* s);
    string& operator+=(char c);
    string& operator+=(initializer_list<char> il);
    string& append(const string& str);
    string& append(const string& str, size_t subpos, size_t sublen);
    string& append(const char* s);
    string& append(const char* s, size_t n);
    string& append(size_t n, char c);
    template <class InputIterator> string& append(InputIterator first, InputIterator last);
    string& append(initializer_list<char> il);
    void push_back(char c);
    string& assign(const string& str);
    string& assign(const string& str, size_t subpos, size_t sublen);
    string& assign(const char* s);
    string& assign(const char* s, size_t n);
    string& assign(size_t n, char c);
    template <class InputIterator> string& assign(InputIterator first, InputIterator last);
    string& assign(initializer_list<char> il);
    string& assign(string&& str) noexcept;
    string& insert(size_t pos, const string& str);
    string& insert(size_t pos, const string& str, size_t subpos, size_t sublen);
    string& insert(size_t pos, const char* s);
    string& insert(size_t pos, const char* s, size_t n);
    string& insert(size_t pos, size_t n, char c);
    iterator insert(const_iterator p, size_t n, char c);
    iterator insert(const_iterator p, char c);
    template <class InputIterator> iterator insert(iterator p, InputIterator first, InputIterator last);
    string& insert(const_iterator p, initializer_list<char> il);
    string& erase(size_t pos = 0, size_t len = npos);
    iterator erase(const_iterator p);
    iterator erase(const_iterator first, const_iterator last);
    string& replace(size_t pos, size_t len, const string& str);
    string& replace(const_iterator i1, const_iterator i2, const string& str);
    string& replace(size_t pos, size_t len, const string& str, size_t subpos, size_t sublen);
    string& replace(size_t pos, size_t len, const char* s);
    string& replace(const_iterator i1, const_iterator i2, const char* s);
    string& replace(size_t pos, size_t len, const char* s, size_t n);
    string& replace(const_iterator i1, const_iterator i2, const char* s, size_t n);
    string& replace(size_t pos, size_t len, size_t n, char c);
    string& replace(const_iterator i1, const_iterator i2, size_t n, char c);
    template <class InputIterator> string& replace(const_iterator i1, const_iterator i2, InputIterator first, InputIterator last);
    string& replace(const_iterator i1, const_iterator i2, initializer_list<char> il);
    void swap(string& str);
    void pop_back();

    // String Operations
    const char* c_str() const noexcept;
    const char* data() const noexcept;
    allocator_type get_allocator() const noexcept;
    size_t copy(char* s, size_t len, size_t pos = 0) const;
    size_t find(const string& str, size_t pos = 0) const noexcept;
    size_t find(const char* s, size_t pos = 0) const;
    size_t find(const char* s, size_t pos, size_type n) const;
    size_t find(char c, size_t pos = 0) const noexcept;
    size_t rfind(const string& str, size_t pos = npos) const noexcept;
    size_t rfind(const char* s, size_t pos = npos) const;
    size_t rfind(const char* s, size_t pos, size_t n) const;
    size_t rfind(char c, size_t pos = npos) const noexcept;
    size_t find_first_of(const string& str, size_t pos = 0) const noexcept;
    size_t find_first_of(const char* s, size_t pos = 0) const;
    size_t find_first_of(const char* s, size_t pos, size_t n) const;
    size_t find_first_of(char c, size_t pos = 0) const noexcept;
    size_t find_last_of(const string& str, size_t pos = npos) const noexcept;
    size_t find_last_of(const char* s, size_t pos = npos) const;
    size_t find_last_of(const char* s, size_t pos, size_t n) const;
    size_t find_last_of(char c, size_t pos = npos) const noexcept;
    size_t find_first_not_of(const string& str, size_t pos = 0) const noexcept;
    size_t find_first_not_of(const char* s, size_t pos = 0) const;
    size_t find_first_not_of(const char* s, size_t pos, size_t n) const;
    size_t find_first_not_of(char c, size_t pos = 0) const noexcept;
    size_t find_last_not_of(const string& str, size_t pos = npos) const noexcept;
    size_t find_last_not_of(const char* s, size_t pos = npos) const;
    size_t find_last_not_of(const char* s, size_t pos, size_t n) const;
    size_t find_last_not_of(char c, size_t pos = npos) const noexcept;
    string substr(size_t pos = 0, size_t len = npos) const;
    int compare(const string& str) const noexcept;
    int compare(size_t pos, size_t len, const string& str) const;
    int compare(size_t pos, size_t len, const string& str, size_t subpos, size_t sublen) const;
    int compare(const char* s) const;
    int compare(size_t pos, size_t len, const char* s) const;
    int compare(size_t pos, size_t len, const char* s, size_t n) const;

    // Member Constants
    static const size_t npos = -1;
};
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

向量是表示可以更改大小的数组的序列容器。和数组一样，其元素使用连续的存储位置，但不同的是向量大小是可以动态变化。

原型如下：

```cpp
template <class T, class Alloc = allocator<T>> class vector;
```

**API 定义**

```cpp
template <class T, class Alloc = allocator<T>>
class vector {
public:
    explicit vector(const allocator_type& alloc = allocator_type());
    explicit vector(size_type n);
    vector(size_type n, const value_type& val, const allocator_type& alloc = allocator_type());
    template <class InputIterator> vector(InputIterator first, InputIterator last, const allocator_type& alloc = allocator_type());
    vector(const vector& x);
    vector(const vector& x, const allocator_type& alloc);
    vector(vector&& x);
    vector(vector&& x, const allocator_type& alloc);
    vector(initializer_list<value_type> il, const allocator_type& alloc = allocator_type());
    ~vector();
    vector& operator=(const vector& x);
    vector& operator=(vector&& x);
    vector& oerator=(initializer_list<value_type> il);

    // Iterators
    iterator begin() noexcept;
    const_iterator begin() const noexcept;
    iterator end() noexcept;
    const_iterator end() const noexcept;
    reverse_iterator rbegin() noexcept;
    const_reverse_iterator rbegin() const noexcept;
    reverse_iterator rend() noexcept;
    const_reverse_iterator rend() const noexcept;
    const_iterator cbegin() const noexcept;
    const_iterator cend() const noexcept;
    const_reverse_iterator crbegin() const noexcept;
    const_reverse_iterator crend() const noexcept;

    // Capacity
    size_type size() const noexcept;
    size_type max_size() const noexcept;
    void resize(size_type n);
    void resize(size_type n, const value_type& val);
    size_type capacity() const noexcept;
    bool empty() const noexcept;
    void reserve(size_type n);
    void shrink_to_fit();

    // Element Access
    reference operator[](size_type n);
    const_reference operator[](size_type n) const;
    reference at(size_type n);
    const_reference at(size_type n) const;
    reference front();
    const_reference front() const;
    reference back();
    const_reference back() const;
    value_type* data() noexcept;
    const value_type* data() const noexcept;

    // Modifiers
    template <class InputIterator> void assign(InputIterator first, InputIterator last);
    void assign(size_type n, const value_type& val);
    void assign(initializer_list<value_type> il);
    void push_back(const value_type& val);
    void push_back(value_type&& val);
    void pop_back();
    iterator insert(const_iterator position, const value_type& val);
    iterator insert(const_iterator position, size_type n, const value_type& val);
    template <class InputIterator> iterator insert(const_iterator position, InputIterator first, InputIterator last);
    iterator insert(const_iterator position, value_type&& val);
    iterator insert(const_iterator position, initializer_list<value_type> il);
    iterator erase(const_iterator position);
    iterator erase(const_iterator first, const_iterator last);
    void swap(vector& x);
    void clear() noexcept;
    template <class... Args> iterator emplace(const_iterator position, Args&&... args);
    template <class... Args> void emplace_back(Args&&... args);

    // Allocator
    allocator_type get_allocator() const noexcept;
};
```

### 使用

1. 头文件

```cpp
#include <vector>
```

2. 新增操作

```cpp
std::vector<int> vec;
vec.push_back(1);
vec.insert(vec.begin(), 2);
vec.emplace(vec.begin(), 3);
vec.emplace_back(4);
```

3. 删除操作

```cpp
std::vector<int> vec;
vec.pop_back();
vec.erase(vec.begin());
vec.clear();
std::vector<int>().swap(vec);
```


4. 修改操作

```cpp
std::vector<int> vec;
vec[0] = 10;
int *p = vec.data();
*p = 11;
```

5. 查找操作

```cpp
auto it = find(vec.begin(), vec.end(), 11);
if (it != vec.end()) {
    std::cout << "found" << std::endl;
} else {
    std::cout << "not found" << std::endl;
}
```

6. 遍历操作

```cpp
for (size_t i = 0; i < vec.size(); ++i) {
    std::cout << vec[i] << std::endl;
}
// or
for (const auto& it : vec) {
    std::cout << *it << std::endl;
}
// or
for (auto it = vec.begin(); it != vec.end(); ++it) {
    std::cout << *it << std::endl;
}
```

----

## list

### 定义

列表是序列容器，允许在序列中的任何位置进行恒定时间插入和擦除操作，并允许双向迭代。列表容器实现为双链接列表。

原型如下:

```cpp
template <class T, class Alloc = allocator<T>> class list;
```

**API 定义**

```cpp
template <class T, class Alloc = allocator<T>>
class list {
public:
    explicit list(const allocator_type& alloc = allocator_type());
    explicit list(size_type n);
    list(size_type n, const value_type& val, const allocator_type& alloc = allocator_type());
    template <class InputIterator> list(InputIterator first, InputIterator last, const allocator_type& alloc = allocator_type());
    list(const list& x);
    list(const list& x, const allocator_type& alloc);
    list(list&& x);
    list(list&& x, const allocator_type& alloc);
    list(initializer_list<value_type> il, const allocator_type& alloc = allocator_type());
    ~list();
    list& operator=(const list& x);
    list& operator=(list&& x);
    list& operator=(initializer_list<value_type> il);

    // Iterators
    iterator begin() noexcept;
    const_iterator begin() const noexcept;
    iterator end() noexcept;
    const_iterator end() const noexcept;
    reverse_iterator rbegin() noexcept;
    const_reverse_iterator rbegin() const noexcept;
    reverse_iterator rend() noexcept;
    const_reverse_iterator rend() const noexcept;
    const_iterator cbegin() const noexcept;
    const_iterator cend() const noexcept;
    const_reverse_iterator crbegin() const noexcept;
    const_reverse_iterator crend() const noexcept;

    // Capacity
    size_type size() const noexcept;
    size_type max_size() const noexcept;
    bool empty() const noexcept;

    // Element Access
    reference front();
    const_reference front() const;
    reference back();
    const_reference back() const;

    // Modifiers
    template <class InputIterator> void assign(InputIterator first, InputIterator last);
    void assign(size_type n, const value_type& val);
    void assign(initializer_list<value_type> il);
    template <class... Args> void emplace_front(Args&&... args);
    void push_back(const value_type& val);
    void push_back(value_type&& val);
    void pop_back();
    template <class... Args> iterator emplace(const_iterator position, Args&&... args);
    iterator insert(const_iterator position, const value_type& val);
    iterator insert(const_iterator position, size_type n, const value_type& val);
    template <class InputIterator> iterator insert(const_iterator position, InputIterator first, InputIterator last);
    iterator insert(const_iterator position, value_type&& val);
    iterator insert(const_iterator position, initializer_list<value_type> il);
    iterator erase(const_iterator position);
    iterator erase(const_iterator first, const_iterator last);
    void swap(vector& x);
    void resize(size_type n);
    void resize(size_type n, const value_type& val);
    void clear() noexcept;

    // Operations
    void splice(const_iterator position, list& x);
    void splice(const_iterator position, list&& x);
    void splice(const_iterator position, list& x, const_iterator i);
    void splice(const_iterator position, list&& x, const_iterator i);
    void splice(const_iterator position, list& x, const_iterator first, const_iterator last);
    void splice(const_iterator position, list&& x, const_iterator first, const_iterator last);
    void remove(const value_type& val);
    template <class Predicate> void remove_if(Predicate pred);
    void unique();
    template <class BinaryPredicate> void unique(BinaryPredicate binary_pred);
    void merge(list& x);
    void merge(list&& x);
    template <class Compare> void merge(list& x, Compare comp);
    template <class Compare> void merge(list&& x, Compare comp);
    void sort();
    template <class Compare> void sort(Compare comp);
    void reverse() noexcept;

    // Allocator
    allocator_type get_allocator() const noexcept;
};
```

### 使用

1. 头文件

```cpp
#include <list>
```

2. 新增操作

```cpp
std::list<int> li;
li.emplace_front(1);
li.emplace_back(2);
li.push_front(3);
li.push_back(4);
li.emplace(li.begin(), 5);
li.insert(li.begin(), 6);
```

3. 删除操作

```cpp
std::list<int> li;
li.pop_front();
li.pop_back();
li.remove(7);
li.clear();
li.erase(li.begin());
```

4. 修改操作

列表只能用迭代器进行访问元素。

5. 查找操作

```cpp
auto it = find(li.begin(), li.end(), 11);
if (it != li.end()) {
    std::cout << "found" << std::endl;
} else {
    std::cout << "not found" << std::endl;
}
```

6. 遍历操作

```cpp
for (const auto& it : li) {
    std::cout << *it << std::endl;
}
// or
for (auto it = li.begin(); it != li.end(); ++it) {
    std::cout << *it << std::endl;
}
```

----

## forward_list

### 定义

前向列表是序列容器，允许恒定时间插入和擦除序列中的任何地方操作。前向列表实现为单链接列表。

原型如下:

```cpp
template <class T, class Alloc = allocator<T>> class forward_list;
```

**API 定义**

```cpp
template <class T, class Alloc = allocator<T>>
class forward_list {
public:
    explicit forward_list(const allocator_type& alloc = allocator_type());
    explicit forward_list(size_type n);
    forward_list(size_type n, const value_type& val, const allocator_type& alloc = allocator_type());
    template <class InputIterator> forward_list(InputIterator first, InputIterator last, const allocator_type& alloc = allocator_type());
    forward_list(const forward_list& fwdlst);
    forward_list(const forward_list& fwdlst, const allocator_type& alloc);
    forward_list(forward_list&& fwdlst);
    forward_list(forward_list&& fwdlst, const allocator_type& alloc);
    forward_list(initializer_list<value_type> il, const allocator_type& alloc = allocator_type());
    ~forward_list();
    forward_list& operator=(const forward_list& fwdlst);
    forward_list& operator=(forward_list&& fwdlst);
    forward_list& operator=(initializer_list<value_type> il);

    // Iterators
    iterator before_begin() noexcept;
    const_iterator before_begin() const noexcept;
    iterator begin() noexcept;
    const_iterator begin() const noexcept;
    iterator end() noexcept;
    const_iterator end() const noexcept;
    const_iterator cbefore_begin() const noexcept;
    const_iterator cbegin() const noexcept;
    const_iterator cend() const noexcept;

    // Capacity
    size_type max_size() const noexcept;
    bool empty() const noexcept;

    // Element Access
    reference front();
    const_reference front() const;

    // Modifiers
    template <class InputIterator> void assign(InputIterator first, InputIterator last);
    void assign(size_type n, const value_type& val);
    void assign(initializer_list<value_type> il);
    template <class... Args> void emplace_front(Args&&... args);
    void push_front(const value_type& val);
    void push_front(value_type&& val);
    void pop_front();
    template <class... Args> iterator emplace_after(const_iterator position, Args&&... args);
    iterator insert_after(const_iterator position, const value_type& val);
    iterator insert_after(const_iterator position, value_type&& val);
    iterator insert_after(const_iterator position, size_type n, const value_type& val);
    template <class InputIterator> iterator insert_after(const_iterator position, InputIterator first, InputIterator last);
    iterator insert_after(const_iterator position, initializer_list<value_type> il);
    iterator erase_after(const_iterator position);
    iterator erase_after(const_iterator position, const_iterator last);
    void swap(forward_list& fwdlst);
    void resize(size_type n);
    void resize(size_type n, const value_type& val);
    void clear() noexcept;

    // Operations
    void remove(const value_type& val);
    template <class Predicate> void remove_if(Predicate pred);
    void unique();
    template <class BinaryPredicate> void unique(BinaryPredicate binary_pred);
    void merge(forward_list& fwdlst);
    void merge(forward_list&& fwdlst);
    template <class Compare> void merge(forward_list& fwdlst, Compare comp);
    template <class Compare> void merge(forward_list&& fwdlst, Compare comp);
    void sort();
    template <class Compare> void sort(Compare comp);
    void reverse() noexcept;

    // Allocator
    allocator_type get_allocator() const noexcept;
};
```

### 使用

1. 头文件

```cpp
#include <forward_list>
```

2. 新增操作

```cpp
std::forward_list<int> fl;
fl.emplace_front(10);
fl.push_front(11);
fl.emplace_back(fl.begin(), 12);
fl.insert_after(fl.begin(), 13);
```

3. 删除操作

```cpp
std::forward_list<int> fl;
fl.pop_front();
fl.erase_after(fl.begin());
fl.remove(14);
fl.remove_if([](const int& value) {
    return value < 15;
});
```

4. 修改操作

前向列表只能用迭代器进行访问元素。

5. 查找操作

```cpp
auto it = find(fl.begin(), fl.end(), 11);
if (it != fl.end()) {
    std::cout << "found" << std::endl;
} else {
    std::cout << "not found" << std::endl;
}
```

6. 遍历操作

```cpp
for (const auto& it : fl) {
    std::cout << *it << std::endl;
}
// or
for (auto it = fl.begin(); it != fl.end(); ++it) {
    std::cout << *it << std::endl;
}
```

----

## deque

### 定义

原型如下:

```cpp
template <class T, class Alloc = allocator<T>> class deque;
```

**API 定义**

```cpp
template <class T, class Alloc = allocator<T>>
class deque {
public:
    explicit deque(const allocator_type& alloc = allocator_type());
    explicit deque(size_type n);
    deque(size_type n, const value_type& val, const allocator_type& alloc = allocator_type());
    template <class InputIterator> deque(InputIterator first, InputIterator last, const allocator_type& alloc = allocator_type());
    deque(const deque& x);
    deque(const deque& x, const allocator_type& alloc);
    deque(deque&& x);
    deque(deque&& x, const allocator_type& alloc);
    deque(initializer_list<value_type> il, const allocator_type& alloc = allocator_type());
    ~deque();
    deque& operator=(const deque& x);
    deque& operator=(deque&& x);
    deque& operator=(initializerlist<value_type> il);

    // Iterators
    iterator begin() noexcept;
    const_iterator begin() const noexcept;
    iterator end() noexcept;
    const_iterator end() const noexcept;
    reverse_iterator rbegin() noexcept;
    const_reverse_iterator rbegin() const noexcept;
    reverse_iterator rend() noexcept;
    const_reverse_iterator rend() const noexcept;
    const_iterator cbegin() const noexcept;
    const_iterator cend() const noexcept;
    const_reverse_iterator crbegin() const noexcept;
    const_reverse_iterator crend() const noexcept;

    // Capacity
    size_type size() const noexcept;
    size_type max_size() const noexcept
    void resize(size_type n);
    void resize(size_type n, const value_type& val);
    bool empty() const noexcept;
    void shrink_to_fit();

    // Element Access
    reference operator[](size_type n);
    const_reference operator[](size_type n) const;
    reference at(size_type n);
    const_reference at(size_type n) const;
    reference front();
    const_reference front() const;
    reference back();
    const_reference back() const;

    // Modifiers
    template <class InputIterator> void assign(InputIterator first, InputIterator last);
    void assign(size_type n, const value_type& val);
    void assign(initializer_list<value_type> il);
    template <class... Args> void emplace_front(Args&&... args);
    void push_front(const value_type& val);
    void push_front(value_type&& val);
    void push_back(const value_type& val);
    void push_back(value_type&& val);
    void pop_front();
    void pop_back();
    iterator insert(const_iterator position, const value_type& val);
    iterator insert(const_iterator position, size_type n, const value_type& val);
    template <class InputIterator> iterator insert(const_iterator position, InputIterator first, InputIterator last);
    iterator insert(const_iterator position, value_type&& val);
    iterator insert(const_iterator position, initializer_list<value_type> il);
    iterator erase(const_iterator position);
    iterator erase(const_iterator first, const_iterator last);
    void swap(deque& x);
    void clear() noexcept;
    template <class... Args> iterator emplace(const_iterator position, Args&&... args);
    template <class... Args> void emplace_front(Args&&... args);
    template <class... Args> void emplace_back(Args&&... args);

    // Allocator
    allocator_type get_allocator() const noexcept;
};
```

### 使用

1. 头文件

```cpp
#include <deque>
```

2. 新增操作

```cpp
std::deque<int> q;
q.push_front(1);
q.push_back(2);
q.insert(q.begin(), 3);
q.emplace(q.begin(), 4);
q.emplace_front(5);
q.emplace_back(6);
```

3. 删除操作

```cpp
std::deque<int> q;
q.pop_front();
q.pop_back();
q.erase(q.begin());
q.clear();
```

4. 修改操作

```cpp
std::deque<int> q;
q[0] = 7;
q.at(1) = 8;
```

5. 查找操作

```cpp
auto it = find(q.begin(), q.end(), 11);
if (it != q.end()) {
    std::cout << "found" << std::endl;
} else {
    std::cout << "not found" << std::endl;
}
```

6. 遍历操作

```cpp
for (size_t i = 0; i < q.size(); ++i) {
    std::cout << q[i] << std::endl;
}
// or
for (const auto& it : q) {
    std::cout << *it << std::endl;
}
// or
for (auto it = q.begin(); it != q.end(); ++it) {
    std::cout << *it << std::endl;
}
```

----

## map

### 定义

`map` 是有序关联容器，其元素是由键值对的方式存储，底层是用红黑树实现的，可以快速的查找对象。

原型如下:

```cpp
template <class Key,                                    // map::key_type
          class T,                                      // map::mapped_type
          class Compare = less<Key>,                    // map::key_compare
          class Alloc = allocator<pair<const Key, T>>   // map::allocator_type
          > class map;
```

**API 定义**

```cpp
template <class Key,                                    // map::key_type
          class T,                                      // map::mapped_type
          class Compare = less<Key>,                    // map::key_compare
          class Alloc = allocator<pair<const Key, T>>>  // map::allocator_type
class map {
public:
    explicit map(const key_compare& comp = key_compare(), const allocator_type& alloc = allocator_type());
    explicit map(const allocator_type& alloc);
    template <class InputIterator> map(InputIterator first, InputIterator last, const key_compare& comp = key_compare(), const allocator_type& = allocator_type());
    map(const map& x);
    map(const map& x, const allocator_type& alloc);
    map(map&& x);
    map(map&& x, const allcator_type& alloc);
    map(initializer_list<value_type> il, const key_compare& comp = key_compare(), const allocator_type& alloc = allocator_type());
    ~map();
    map& operator=(const map& x);
    map& operator=(map&& x);
    map& operator=(initializer_list<value_type> il);

    // Iterators
    iterator begin() noexcept;
    const_iterator begin() const noexcept;
    iterator end() noexcept;
    const_iterator end() const noexcept;
    reverse_iterator rbegin() noexcept;
    const_reverse_iterator rbegin() const noexcept;
    reverse_iterator rend() noexcept;
    const_reverse_iterator rend() const noexcept;
    const_iterator cbegin() const noexcept;
    const_iterator cend() const noexcept;
    const_reverse_iterator crbegin() const noexcept;
    const_reverse_iterator crend() const noexcept;

    // Capacity
    size_type size() const noexcept;
    size_type max_size() const noexcept
    bool empty() const noexcept;

    // Element Access
    mapped_type& operator[](const key_type& k);
    mapped_type& operator[](key_type&& k);
    mapped_type& at(const key_type& k);
    const mapped_type& at(const key_type& k) const;

    // Modifiers
    pair<iterator, bool> insert(const value_type& val);
    template <class P> pair<iterator, bool> insert(P&& val);
    iterator insert(const_iterator position, const value_type& val);
    template <class P> iterator insert(const_iterator position, P&& val);
    template <class InputIterator> void insert(InputIterator first, InputIterator last);
    void insert(initializer_list<value_type> il);
    iterator  erase(const_iterator position);
    size_type erase(const key_type& k);
    iterator  erase(const_iterator first, const_iterator last);
    void swap(map& x);
    void clear() noexcept;
    template <class... Args> pair<iterator, bool> emplace(Args&&... args);
    template <class... Args> iterator emplace_hint(const_iterator position, Args&&... args);

    // Observers
    key_compare key_comp() const;
    value_compare value_comp() const;

    // Operations
    iterator find(const key_type& k);
    const_iterator find(const key_type& k) const;
    size_type count(const key_type& k) const;
    iterator lower_bound(const key_type& k);
    const_iterator lower_bound(const key_type& k) const;
    iterator upper_bound(const key_type& k);
    const_iterator upper_bound(const key_type& k) const;
    pair<const_iterator, const_iterator> equal_range(const key_type& k) const;
    pair<iterator, iterator> equal_range(const key_type& k);

    // Allocator
    allocator_type get_allocator() const noexcept;
};
```

### 使用

1. 头文件

```cpp
#include <map>
```

2. 新增操作

```cpp
std::map<int, std::string> m;
m[1] = "Hello";
a.at(2) = " ";
m.insert(std::pair<int, std::string>(3, "World"));
m.emplace(4, "!");
m.emplace_hint(5, "hahhh");
```

3. 删除操作

```cpp
std::map<int, std::string> m;
m.erase(m.begin());
m.erase(2);
m.clear();
```

4. 修改操作

```cpp
std::map<int, std::string> m;
m[2] = "OK";
```

5. 查找操作

```cpp
auto& it = m.find(2);
if (it != m.end()) {
    m.erase(it);
}
```

6. 遍历操作

```cpp
for (auto& it : m) {
    std::cout << "m[" << it.first << "] = " << it.second << std::endl;
}
// or
for (auto& it = m.begin(); it != m.end(); ++it) {
    std::cout << "m[" << it->first << "] = " << it->second << std::endl;
}
```

----

## unordered_map

### 定义

`unordered_map` 是无序关联容器，其元素是由键值对的方式存储，底层是用哈希表实现的，可以根据单个元素的密钥快速检索。

原型如下:

```cpp
template <class Key,                                    // unordered_map::key_type
          class T,                                      // unordered_map::mapped_type
          class Hash = hash<Key>,                       // unordered_map::hasher
          class Pred = equal_to<Key>,                   // unordered_map::key_equal
          class Alloc = allocator<pair<const Key, T>>   // unordered_map::allocator_type
          > class unordered_map;
```

**API 定义**

```cpp
template <class Key,                                    // unordered_map::key_type
          class T,                                      // unordered_map::mapped_type
          class Hash = hash<Key>,                       // unordered_map::hasher
          class Pred = equal_to<Key>,                   // unordered_map::key_equal
          class Alloc = allocator<pair<const Key, T>>>  // unordered_map::allocator_type
class unordered_map {
public:
    explicit unordered_map(size_type n = /* see below */, 
                           const hasher& hf = hasher(),
                           const key_equal& eql = key_equal(),
                           const allocator_type& alloc = allocator_type());
    explicit unordered_map(const allocator_type& alloc);
    template <class InputIterator>
    unordered_map(InputIterator first, InputIterator last,
                  size_type n = /* see below */,
                  const hasher& hf = hasher(),
                  const key_equal& eql = key_equal(),
                  const allocator_type& alloc = allocator_type());
    unordered_map(const unordered_map& ump);
    unordered_map(const unordered_map& ump, const allocator_type& alloc);
    unordered_map(unordered_map&& ump);
    unordered_map(unordered_map&& ump, const allocator_type& alloc);
    unordered_map(initializer_list<value_type> il,
                  size_type n = /* see below */,
                  const hasher& hf = hasher(),
                  const key_equal& eql = key_equal(),
                  const allocator_type& alloc = allocator_type());
    ~unordered_map();
    unordered_map& operator=(const unordered_map& ump);
    unordered_map& operator=(unordered_map&& ump);
    unordered_map& operator=(initializer_list<value_type> il);

    // Iterators
    iterator begin() noexcept;
    const_iterator begin() const noexcept;
    local_iterator begin(size_type n);
    const_local_iterator begin(size_type n) const;
    iterator end() noexcept;
    const_iterator end() const noexcept;
    local_iterator end(size_type n);
    const_local_iterator end(size_type n) const;
    const_iterator cbegin() const noexcept;
    const_local_iterator cbegin(size_type n) const;
    const_iterator cend() const noexcept;
    const_local_iterator cend(size_type n)const;

    // Capacity
    size_type size() const noexcept;
    size_type max_size() const noexcept
    bool empty() const noexcept;

    // Element Access
    mapped_type& operator[](const key_type& k);
    mapped_type& operator[](key_type&& k);
    mapped_type& at(const key_type& k);
    const mapped_type& at(const key_type& k) const;

    // Element Lookup
    iterator find(const key_type& k);
    const_iterator find(const key_type& k) const;
    size_type count(const key_type& k) const;
    pair<iterator, iterator> equal_range(const key_type& k);
    pair<const_iterator, const_iterator> equal_range(const key_type& k) const;

    // Modifiers
    template <class... Args> pair<iterator, bool> emplace(Args&&... args);
    template <class... Args> iterator emplace_hint(const_iterator position, Args&&... args);
    pair<iterator, bool> insert(const value_type& val);
    template <class P> pair<iterator, bool> insert(P&& val);
    iterator insert(const_iterator hint, const value_type& val);
    template <class P> iterator insert(const_iterator hint, P&& val);
    template <class InputIterator> void insert(InputIterator first, InputIterator last);
    void insert(initializer_list<value_type> il);
    iterator erase(const_iterator position);
    size_type erase(const key_type& k);
    iterator erase(const_iterator first, const_iterator last);
    void clear() noexcept;
    void swap(unordered_map& ump);

    // Buckets
    size_type bucket_count() const noexcept;
    size_type max_bucket_count() const noexcept;
    size_type bucket_size(size_type n) const;
    size_type bucket(const key_type& k) const;

    // Hash Policy
    float load_factor() const noexcept;
    float max_load_factor() const noexcept;
    void max_load_factor(float z);
    void rehash( size_type n);
    void reserve(size_type n);

    // Observers
    hasher hash_function() const;
    key_equal key_eq() const;
    allocator_type get_allocator() const noexcept;
};
```

### 使用

1. 头文件

```cpp
#include <unordered_map>
```

2. 新增操作

```cpp
std::unordered_map<int, std::string> umap;
umap[1] = "Hello";
umap.at(2) = "World";
umap.emplace(3, "Face");
umap.emplace(umap.begin(), "To");
umap.insert({4, "Smile"});
umap.insert(std::make_pair<int, std::string>(5, "OK"));
```

3. 删除操作

```cpp
std::unordered_map<int, std::string> umap;
umap.erase(umap.begin());
umap.clear();
```

4. 修改操作

```cpp
std::unordered_map<int, std::string> umap;
umap[1] = "Map";
```

5. 查找操作

```cpp
auto& it = umap.find(2);
if (it != umap.end()) {
    umap.erase(it);
}
```

6. 遍历操作

```cpp
for (auto& it : umap) {
    std::cout << "umap[" << it.first << "] = " << it.second << std::endl;
}
// or
for (auto& it = umap.begin(); it != umap.end(); ++it) {
    std::cout << "umap[" << it->first << "] = " << it->second << std::endl;
}
```

----

## set

### 定义

集合是按照特定顺序存储唯一元素的容器。底层是用红黑树实现的，可以快速的查找对象。

原型如下:

```cpp
template <class T,                      // set::key_type/value_type
          class Compare = less<T>,      // set::key_compare/value_compare
          class Alloc = allocator<T>    // set::allocator_type
          > class set;
```

**API 定义**

```cpp
template <class T,                      // set::key_type/value_type
          class Compare = less<T>,      // set::key_compare/value_compare
          class Alloc = allocator<T>>   // set::allocator_type
class set {
public:
    explicit set(const key_compare& comp = key_compare(),
                 const allocator_type& alloc = allocator_type());
    explicit set(const allocator_type& alloc);
    template <class InputIterator>
    set(InputIterator first, InputIterator last,
        const key_compare& comp = key_compare(),
        const allocator_type& = allocator_type());
    set(const set& x);
    set(const set& x, const allocator_type& alloc);
    set(set&& x);
    set(set&& x, const allocator_type& alloc);
    set(initializer_list<value_type> il,
       const key_compare& comp = key_compare(),
        const allocator_type& alloc = allocator_type());
    ~set();
    set& operator=(const set& x);
    set& operator=(set&& x);
    set& operator=(initializer_list<value_type> il);

    // Iterators
    iterator begin() noexcept;
    const_iterator begin() const noexcept;
    iterator end() noexcept;
    const_iterator end() const noexcept;
    reverse_iterator rbegin() noexcept;
    const_reverse_iterator rbegin() const noexcept;
    reverse_iterator rend() noexcept;
    const_reverse_iterator rend() const noexcept;
    const_iterator cbegin() const noexcept;
    const_iterator cend() const noexcept;
    const_reverse_iterator crbegin() const noexcept;
    const_reverse_iterator crend() const noexcept;

    // Capacity
    size_type size() const noexcept;
    size_type max_size() const noexcept
    bool empty() const noexcept;

    // Modifiers
    pair<iterator, bool> insert(const value_type& val);
    pair<iterator, bool> insert(value_type&& val);
    iterator insert(const_iterator position, const value_type& val);
    iterator insert(const_iterator position, value_type&& val);
    template <class InputIterator> void insert(InputIterator first, InputIterator last);
    void insert(initializer_list<value_type> il);
    iterator  erase(const_iterator position);
    size_type erase(const value_type& val);
    iterator  erase(const_iterator first, const_iterator last);
    void clear() noexcept;
    void swap(set& ump);
    template <class... Args> pair<iterator, bool> emplace(Args&&... args);
    template <class... Args> iterator emplace_hint(const_iterator position, Args&&... args);

    // Observers
    key_compare key_comp() const;
    value_compare value_comp() const;

    // Operations
    const_iterator find(const value_type& val) const;
    iterator find(const value_type& val);
    size_type count(const value_type& val) const;
    iterator lower_bound(const value_type& val);
    const_iterator lower_bound(const value_type& val) const;
    iterator upper_bound(const value_type& val);
    const_iterator upper_bound(const value_type& val) const;
    pair<const_iterator, const_iterator> equal_range(const value_type& val) const;
    pair<iterator, iterator> equal_range(const value_type& val);

    // Allocator
    allocator_type get_allocator() const noexcept;
};
```

### 使用

1. 头文件

```cpp
#include <set>
```

2. 新增操作

```cpp
std::set<int> s;
s.insert(10);
s.emplace(11);
s.emplace_hint(s.begin(), 12);
```

3. 删除操作

```cpp
std::set<int> s;
s.erase(s.begin());
s.erase(13);
s.clear();
```

4. 修改操作

集合中元素的值是常量，不能在容器中进行修改，但它们可以从容器中插入或删除。

5. 查找操作

```cpp
auto& it = s.find(2);
if (it != s.end()) {
    s.erase(it);
}
```

6. 遍历操作

```cpp
for (auto& it : s) {
    std::cout << "s[" << it.first << "] = " << it.second << std::endl;
}
// or
for (auto& it = s.begin(); it != s.end(); ++it) {
    std::cout << "s[" << it->first << "] = " << it->second << std::endl;
}
```

----

## unordered_set

### 定义

`unordered_set` 是无序关联容器，底层是用哈希表实现的，可以根据元素的值快速检索。

原型如下:

```cpp
template <class Key,                    // unordered_set::key_type/value_type
          class Hash = hash<Key>,       // unordered_set::hasher
          class Pred = equal_to<Key>,   // unordered_set::key_equal
          class Alloc = allocator<Key>  // unordered_set::allocator_type
          > class unordered_set;
```

**API 定义**

```cpp
template <class Key,                        // unordered_set::key_type/value_type
          class Hash = hash<Key>,           // unordered_set::hasher
          class Pred = equal_to<Key>,       // unordered_set::key_equal
          class Alloc = allocator<Key>>     // unordered_set::allocator_type
class unordered_set {
public:
    explicit unordered_set(size_type n = /* see below */,
                           const hasher& hf = hasher(),
                           const key_equal& eql = key_equal(),
                           const allocator_type& alloc = allocator_type());
    explicit unordered_set(const allocator_type& alloc);
    template <class InputIterator>
    unordered_set(InputIterator first, InputIterator last,
                  size_type n = /* see below */,
                  const hasher& hf = hasher(),
                  const key_equal& eql = key_equal(),
                  const allocator_type& alloc = allocator_type());
    unordered_set(const unordered_set& ust);
    unordered_set(const unordered_set& ust, const allocator_type& alloc);
    unordered_set(unordered_set&& ust);
    unordered_set(unordered_set&& ust, const allocator_type& alloc);
    unordered_set(initializer_list<value_type> il,
                  size_type n = /* see below */,
                  const hasher& hf = hasher(),
                  const key_equal& eql = key_equal(),
                  const allocator_type& alloc = allocator_type());
    ~unordered_set();
    unordered_set& operator=(const unordered_set& ust);
    unordered_set& operator=(unordered_set&& ust);
    unordered_set& operator=(initializer_list<value_type> il);

    // Iterators
    iterator begin() noexcept;
    const_iterator begin() const noexcept;
    local_iterator begin(size_type n);
    const_local_iterator begin(size_type n) const;
    iterator end() noexcept;
    const_iterator end() const noexcept;
    local_iterator end(size_type n);
    const_local_iterator end(size_type n) const;
    const_iterator cbegin() const noexcept;
    const_local_iterator cbegin(size_type n) const;
    const_iterator cend() const noexcept;
    const_local_iterator cend(size_type n) const;

    // Element Lookup
    iterator find(const key_type& k);
    const_iterator find(const key_type& k) const;
    size_type count(const key_type& k) const;
    pair<iterator, iterator> equal_range(const key_type& k);
    pair<const_iterator, const_iterator> equal_range(const key_type& k) const;

    // Modifiers
    template <class... Args> pair<iterator, bool> emplace(Args&&... args);
    template <class... Args> iterator emplace_hint(const_iterator position, Args&&... args);
    pair<iterator, bool> insert(const value_type& val);
    pair<iterator, bool> insert(value_type&& val);
    iterator insert(const_iterator hint, const value_type& val);
    iterator insert(const_iterator hint, value_type&& val);
    template <class InputIterator> void insert(InputIterator first, InputIterator last);
    void insert(initializer_list<value_type> il);
    iterator erase(const_iterator position);
    size_type erase(const key_type& k);
    iterator erase(const_iterator first, const_iterator last);
    void clear() noexcept;
    void swap(unordered_set& ump);

    // Buckets
    size_type bucket_count() const noexcept;
    size_type max_bucket_count() const noexcept;
    size_type bucket_size(size_type n) const;
    size_type bucket(const key_type& k) const;

    // Hash Policy
    float load_factor() const noexcept;
    float max_load_factor() const noexcept;
    void max_load_factor(float z);
    void rehash(size_type n);
    void reserve(size_type n);

    // Observers
    hasher hash_function() const;
    key_equal key_eq() const;
    allocator_type get_allocator() const noexcept;
};
```

### 使用

1. 头文件

```cpp
#include <unordered_set>
```

2. 新增操作

```cpp
std::unordered_set<int> uset;
uset.insert(10);
uset.emplace(11);
uset.emplace_hint(uset.begin(), 13);
```

3. 删除操作

```cpp
std::unordered_set<int> uset;
uset.erase(uset.begin());
uset.erase(12);
uset.clear();
```

4. 修改操作

`unordered_set` 元素的值同时是其密钥，可以唯一地标识它。键是不可变的，因此 `unordered_set` 中的元素不能修改。不过，它们可以插入和删除。

5. 查找操作

```cpp
auto& it = uset.find(2);
if (it != uset.end()) {
    uset.erase(it);
}
```

6. 遍历操作

```cpp
for (auto& it : uset) {
    std::cout << "uset[" << it.first << "] = " << it.second << std::endl;
}
// or
for (auto& it = uset.begin(); it != uset.end(); ++it) {
    std::cout << "uset[" << it->first << "] = " << it->second << std::endl;
}
```

----


