# Python 语法详细总结


# Python 语法详细总结（基于Python3，覆盖核心语法全体系）
本文档系统梳理Python3核心语法，从基础规范到高级特性，兼顾入门理解与实战使用，标注关键版本特性与使用注意事项。

## 一、基础语法规范
### 1.1 注释
Python注释分为单行、多行与文档字符串，解释器会忽略注释内容，仅用于代码说明。
```python
# 1. 单行注释：以#开头，#后建议加空格
print("hello world")  # 行尾注释

# 2. 多行注释：三对单引号/双引号包裹（本质是未赋值的字符串常量）
'''
这是多行注释
第二行注释
'''
"""
这也是多行注释
常用于函数/类的文档说明
"""

# 3. 文档字符串docstring：函数/类/模块的首行三引号注释，可通过__doc__属性访问
def add(a, b):
    """两个数字相加，返回求和结果"""
    return a + b
print(add.__doc__)  # 输出文档字符串
```

### 1.2 标识符与关键字
- **标识符**：用于变量、函数、类、模块等的命名，规则如下：
  1. 只能由字母、数字、下划线`_`组成，不能以数字开头
  2. 严格区分大小写（`age`和`Age`是两个不同标识符）
  3. 不能使用Python关键字
  4. 约定规范：蛇形命名（`user_name`）用于变量/函数，大驼峰（`UserInfo`）用于类，全大写（`MAX_SIZE`）用于常量
- **关键字**：Python内置的具有特殊含义的标识符，不可用作命名
  ```python
  import keyword
  print(keyword.kwlist)  # 查看所有关键字
  # 常用关键字：if、elif、else、for、while、def、class、return、import、from、try、except、finally、with、as、lambda、global、nonlocal等
  ```

### 1.3 缩进与代码块
Python**不用大括号`{}`划分代码块**，完全依靠缩进控制代码层级，是Python最核心的语法特点之一。
1. 缩进规范：PEP8建议使用**4个空格**作为1个缩进层级，禁止Tab和空格混用
2. 同一代码块的缩进必须完全一致，缩进错误会直接触发`IndentationError`
3. 冒号`:`是代码块的开启标志（分支、循环、函数、类等语句后必须加冒号）
```python
# 正确缩进示例
if 10 > 5:
    print("条件成立")  # 4个空格缩进
    for i in range(3):
        print(i)  # 再嵌套4个空格，共8个
else:
    print("条件不成立")
```

### 1.4 语句与换行
1. 一行默认一个语句，语句结尾无需分号
2. 长语句换行：使用反斜杠`\`，或在括号`()/[]/{}`内自动换行（推荐）
3. 一行多个语句：用分号`;`分隔（不推荐，降低可读性）
```python
# 1. 长语句换行
total = 1 + 2 + 3 + 4 + \
        5 + 6 + 7 + 8

total = (1 + 2 + 3 + 4 +
         5 + 6 + 7 + 8)  # 推荐，无需反斜杠

# 2. 一行多个语句
a = 1; b = 2; c = 3  # 不推荐
```

## 二、核心数据类型
Python是**动态强类型语言**：变量无需提前声明类型，赋值时自动绑定类型；不同类型之间不允许隐式转换（如字符串和数字不能直接相加）。

数据类型分为**不可变类型**（值不可修改，修改即创建新对象）和**可变类型**（值可原地修改，内存地址不变）两大类。

| 类型分类 | 具体类型 | 核心特点 |
|----------|----------|----------|
| 不可变类型 | 数字(int/float/complex/bool)、字符串(str)、元组(tuple)、None | 值不可修改，可哈希，可作为字典的key |
| 可变类型 | 列表(list)、字典(dict)、集合(set) | 值可原地修改，不可哈希，不能作为字典的key |

### 2.1 数字类型(Number)
Python支持4种核心数字类型，无大小限制，无需担心溢出。
```python
# 1. 整数int：正负整数，无大小限制
a = 10
b = -5
c = 0x10  # 十六进制
d = 0o10  # 八进制
e = 0b10  # 二进制

# 2. 浮点数float：双精度小数，支持科学计数法
f = 3.14
g = -2.5
h = 1e3  # 1000.0

# 3. 布尔值bool：int的子类，True=1，False=0
i = True
j = False
print(True + 1)  # 输出2

# 4. 复数complex：实部+虚部，虚部用j表示
k = 3 + 4j
print(k.real)  # 实部3.0
print(k.imag)  # 虚部4.0
```

**核心数字运算**
```python
# 算术运算
print(10 + 3)   # 加 13
print(10 - 3)   # 减 7
print(10 * 3)   # 乘 30
print(10 / 3)   # 除 3.333...（永远返回浮点数）
print(10 // 3)  # 整除 3（向下取整）
print(10 % 3)   # 取模 1（余数）
print(10 ** 3)  # 幂运算 1000

# 赋值运算
a = 10
a += 5  # 等价于a = a + 5，同理-=、*=、/=、//=、%=、**=

# 比较运算：返回True/False
print(10 > 3)
print(10 < 3)
print(10 >= 3)
print(10 <= 3)
print(10 == 3)  # 等于
print(10 != 3)  # 不等于

# 逻辑运算
print(True and False)  # 与：全True才返回True
print(True or False)   # 或：有一个True就返回True
print(not True)        # 非：取反

# 位运算（针对二进制）
print(10 & 3)  # 按位与
print(10 | 3)  # 按位或
print(10 ^ 3)  # 按位异或
print(~10)     # 按位取反
print(10 << 2) # 左移2位
print(10 >> 2) # 右移2位
```

### 2.2 字符串类型(String)
字符串是**有序、不可变**的字符序列，用单引号、双引号、三引号包裹，支持Unicode编码。
```python
# 字符串定义
s1 = '单引号字符串'
s2 = "双引号字符串"
s3 = '''三引号多行字符串
第二行
第三行'''
s4 = r"原始字符串\不转义"  # r开头，转义字符\失效，常用于路径、正则
```

**核心操作**
1. 索引与切片：字符串是有序序列，支持通过索引访问单个字符，切片访问子串
   ```python
   s = "python"
   # 索引：正索引从0开始，负索引从-1开始（末尾）
   print(s[0])   # 第一个字符p
   print(s[-1])  # 最后一个字符n

   # 切片语法：[起始索引:结束索引:步长]，左闭右开，省略则取边界
   print(s[1:4])   # yth（索引1-3，不包含4）
   print(s[:3])    # pyt（从开头到索引2）
   print(s[2:])    # thon（从索引2到末尾）
   print(s[::2])   # pto（步长2，隔一个取一个）
   print(s[::-1])  # nohtyp（步长-1，字符串反转）
   ```
2. 基础运算
   ```python
   s1 = "hello"
   s2 = "world"
   print(s1 + s2)  # 拼接：helloworld
   print(s1 * 3)   # 重复：hellohellohello
   print("h" in s1)  # 成员判断：True
   print("z" not in s1)  # True
   ```
3. 字符串格式化（3种主流方式）
   ```python
   name = "张三"
   age = 20
   # 1. f-string（Python3.6+推荐，简洁高效）
   print(f"姓名：{name}，年龄：{age}")
   print(f"计算结果：{10+20}")  # 支持表达式

   # 2. str.format()
   print("姓名：{}，年龄：{}".format(name, age))
   print("姓名：{0}，年龄：{1}，姓名重复：{0}".format(name, age))  # 索引指定

   # 3. %占位符（传统方式）
   print("姓名：%s，年龄：%d" % (name, age))  # %s字符串，%d整数，%f浮点数
   ```
4. 常用内置方法
   | 方法 | 作用 |
   |------|------|
   | `strip()` | 去除首尾空白字符（空格、换行、制表符），可指定字符 |
   | `split(sep)` | 按指定分隔符分割字符串，返回列表 |
   | `join(iterable)` | 用字符串拼接可迭代对象的元素，如`"-".join(["a","b"])` |
   | `replace(old, new)` | 替换字符串中的指定子串 |
   | `upper()/lower()` | 全部转为大写/小写 |
   | `startswith()/endswith()` | 判断是否以指定子串开头/结尾 |
   | `find(sub)` | 查找子串首次出现的索引，不存在返回-1 |
   | `count(sub)` | 统计子串出现的次数 |

### 2.3 列表类型(List)
列表是**有序、可变、可重复**的序列，用`[]`包裹，元素类型无限制，是Python最常用的数据结构。
```python
# 列表定义
list1 = [1, 2, 3, 4, 5]  # 同类型元素
list2 = [1, "python", True, 3.14, [1,2]]  # 不同类型元素
list3 = []  # 空列表
```

**核心操作**
1. 索引与切片：和字符串一致，区别是列表切片赋值可原地修改原列表
   ```python
   li = [1,2,3,4,5]
   print(li[0])  # 1
   print(li[-1]) # 5
   print(li[1:4]) # [2,3,4]

   # 切片赋值（列表特有）
   li[1:3] = [20,30]
   print(li)  # [1,20,30,4,5]
   ```
2. 增删改查
   ```python
   li = [1,2,3]
   # 增
   li.append(4)  # 末尾添加单个元素：[1,2,3,4]
   li.extend([5,6])  # 末尾拼接可迭代对象：[1,2,3,4,5,6]
   li.insert(1, 10)  # 指定索引插入元素：[1,10,2,3,4,5,6]

   # 改
   li[0] = 100  # 索引直接赋值：[100,10,2,3,4,5,6]

   # 查
   print(li.index(2))  # 查找元素首次出现的索引
   print(li.count(3))  # 统计元素出现次数

   # 删
   li.pop()  # 删除末尾元素，返回被删除的值
   li.pop(1) # 删除指定索引元素
   li.remove(2) # 删除首次出现的指定元素
   del li[0] # 删除指定索引/切片元素
   li.clear() # 清空列表，返回空列表
   ```
3. 其他常用方法
   ```python
   li = [3,1,4,2]
   li.sort()  # 原地升序排序：[1,2,3,4]
   li.sort(reverse=True)  # 原地降序排序
   li.reverse() # 原地反转列表
   li2 = li.copy() # 浅拷贝列表
   ```

### 2.4 元组类型(Tuple)
元组是**有序、不可变、可重复**的序列，用`()`包裹，元素类型无限制，一旦创建无法修改，安全性高。
```python
# 元组定义
t1 = (1,2,3,4,5)
t2 = (1, "python", True, 3.14)
t3 = ()  # 空元组
t4 = (10,)  # 单元素元组必须加逗号，否则会被识别为int类型
```

**核心操作**
1. 索引、切片、拼接、重复、成员判断，和字符串完全一致，不可修改，无增删改方法
2. 仅有的两个内置方法：`count()`统计元素出现次数，`index()`查找元素索引
3. 核心用途：保护数据不被修改、作为字典的key、函数返回多个值（本质是返回元组）
   ```python
   def get_user():
       return "张三", 20  # 等价于return ("张三", 20)
   name, age = get_user()  # 元组解包
   ```

### 2.5 字典类型(Dict)
字典是**键值对(key-value)结构**，Python3.7+保留插入顺序，key必须是不可变类型（可哈希）且唯一，value无类型限制，用`{}`包裹。
```python
# 字典定义
dict1 = {"name":"张三", "age":20, "gender":"男"}
dict2 = {}  # 空字典
dict3 = dict(name="李四", age=22)  # 关键字创建
```

**核心操作**
```python
d = {"name":"张三", "age":20}

# 查
print(d["name"])  # 按键取值，key不存在会报错KeyError
print(d.get("age"))  # 推荐，key不存在返回None，不会报错
print(d.get("height", 180))  # key不存在返回默认值180
print(d.keys())  # 获取所有key
print(d.values())  # 获取所有value
print(d.items())  # 获取所有(key, value)键值对，常用于for循环

# 增/改
d["age"] = 21  # key存在则修改，不存在则新增
d["height"] = 180  # 新增键值对
d.update({"gender":"男", "weight":65})  # 批量新增/修改

# 删
d.pop("age")  # 删除指定key的键值对，返回value
d.popitem()  # Python3.7+删除最后插入的键值对
del d["name"]  # 删除指定key的键值对
d.clear()  # 清空字典
```

### 2.6 集合类型(Set)
集合是**无序、元素唯一、可变**的容器，用`{}`包裹，元素必须是不可变类型，核心用途是**去重**和**集合运算**。
```python
# 集合定义
s1 = {1,2,3,4,5}
s2 = set()  # 空集合必须用set()创建，{}是空字典
s3 = set([1,2,2,3,3,3])  # 列表转集合，自动去重：{1,2,3}
```

**核心操作**
1. 集合运算
   ```python
   a = {1,2,3,4}
   b = {3,4,5,6}
   print(a & b)  # 交集：{3,4}
   print(a | b)  # 并集：{1,2,3,4,5,6}
   print(a - b)  # 差集：{1,2}（a有b没有的元素）
   print(a ^ b)  # 对称差集：{1,2,5,6}（两个集合独有的元素）
   ```
2. 增删操作
   ```python
   s = {1,2,3}
   s.add(4)  # 添加单个元素
   s.update([5,6])  # 批量添加可迭代对象
   s.remove(3)  # 删除指定元素，元素不存在报错
   s.discard(10)  # 删除指定元素，不存在不报错（推荐）
   s.pop()  # 随机删除一个元素
   s.clear()  # 清空集合
   ```

### 2.7 空值None
None是Python中唯一的`NoneType`类型，代表空、无值，不是0、不是空字符串、不是False，常用于变量初始化、函数默认返回值（函数无return语句时，默认返回None）。

## 三、流程控制语句
Python流程控制分为**顺序结构**、**分支结构**、**循环结构**三大类，控制程序的执行逻辑。

### 3.1 分支结构（if-elif-else）
根据条件判断执行不同的代码块，支持多分支嵌套。
```python
# 基础语法
age = 20
if age < 18:
    print("未成年")
elif age < 60:
    print("成年")
else:
    print("老年")

# 三元表达式（简化单if-else，一行完成）
# 语法：结果1 if 条件 else 结果2
is_adult = "成年" if age >= 18 else "未成年"
print(is_adult)

# 多条件判断
score = 85
if score >= 90 and score <= 100:
    print("优秀")
elif score >= 80:
    print("良好")
elif score >= 60:
    print("及格")
else:
    print("不及格")
```

**match-case模式匹配（Python3.10+）**
类似其他语言的switch-case，支持更复杂的模式匹配，语法更简洁。
```python
status = 200
match status:
    case 200:
        print("请求成功")
    case 404:
        print("页面不存在")
    case 500 | 502:  # 多值匹配，用|分隔
        print("服务器错误")
    case _:  # 通配符，匹配所有情况，相当于default
        print("未知状态码")
```

### 3.2 循环结构
Python提供`for`和`while`两种循环，`for`更常用，用于遍历可迭代对象；`while`用于满足条件时循环执行。

#### 3.2.1 while循环
```python
# 基础语法
i = 0
while i < 5:
    print(i)
    i += 1

# while-else结构：else代码块在循环正常结束（非break跳出）时执行
i = 0
while i < 5:
    print(i)
    i += 1
else:
    print("循环正常结束")
```

#### 3.2.2 for循环
用于遍历可迭代对象（列表、元组、字符串、字典、range、生成器等），语法简洁。
```python
# 基础遍历
li = [1,2,3,4,5]
for num in li:
    print(num)

# 遍历字符串
for char in "python":
    print(char)

# 遍历字典
d = {"name":"张三", "age":20}
for key in d:  # 遍历key
    print(key, d[key])
for k, v in d.items():  # 遍历键值对（推荐）
    print(k, v)

# range()函数：生成整数序列，语法range(start, end, step)，左闭右开
for i in range(5):  # 0-4
    print(i)
for i in range(1, 10, 2):  # 1,3,5,7,9
    print(i)

# for-else结构：和while-else一致，循环正常结束执行else
for i in range(5):
    print(i)
else:
    print("循环正常结束")
```

#### 3.2.3 循环控制语句
| 语句 | 作用 |
|------|------|
| `break` | 立即跳出**整个循环**，不再执行后续循环，else代码块也不执行 |
| `continue` | 跳过**本次循环**剩余代码，直接进入下一次循环 |
| `pass` | 空语句，仅作占位符，不执行任何操作，用于语法要求必须有代码块的场景 |

```python
# break示例
for i in range(5):
    if i == 3:
        break
    print(i)  # 输出0,1,2

# continue示例
for i in range(5):
    if i == 3:
        continue
    print(i)  # 输出0,1,2,4

# pass示例
for i in range(5):
    if i == 3:
        pass  # 暂不实现逻辑，先占位，避免语法报错
    print(i)
```

## 四、函数
Python中函数是**一等公民**，可赋值给变量、作为参数传递、作为返回值，支持丰富的参数类型和高级特性。

### 4.1 函数定义与调用
用`def`关键字定义函数，`return`返回值，无return默认返回None。
```python
# 函数定义
def add(a, b):
    """两个数字相加，返回求和结果"""
    result = a + b
    return result

# 函数调用
sum_result = add(10, 20)
print(sum_result)  # 30

# 无返回值函数
def say_hello():
    print("hello world")

say_hello()  # 调用
```

### 4.2 函数参数（核心重点）
Python函数参数类型丰富，按优先级从高到低依次为：位置参数 > 关键字参数 > 默认参数 > 可变位置参数 > 强制关键字参数 > 可变关键字参数。

| 参数类型 | 说明 |
|----------|------|
| 位置参数 | 按顺序传递，数量必须和定义一致，是最基础的参数 |
| 关键字参数 | 按参数名传递，顺序可任意，必须放在位置参数之后 |
| 默认参数 | 定义时给参数设置默认值，调用时可省略，必须放在位置参数之后 |
| 可变位置参数`*args` | 接收任意多个位置参数，打包成元组 |
| 强制关键字参数 | `*`之后的参数，必须用关键字传递 |
| 可变关键字参数`**kwargs` | 接收任意多个关键字参数，打包成字典 |

```python
# 1. 位置参数
def func1(a, b, c):
    print(a, b, c)
func1(1, 2, 3)  # 按顺序传递

# 2. 关键字参数
func1(a=1, c=3, b=2)  # 顺序可乱
func1(1, c=3, b=2)  # 位置参数在前，关键字参数在后

# 3. 默认参数
def func2(a, b=10, c=20):
    print(a, b, c)
func2(5)  # 输出5 10 20，b和c用默认值
func2(5, 15)  # 输出5 15 20，c用默认值
func2(5, c=25)  # 输出5 10 25，b用默认值

# 注意：默认参数不能用可变类型（列表/字典），否则会有累积问题
# 错误示例
def func_bad(a, li=[]):
    li.append(a)
    print(li)
func_bad(1)  # [1]
func_bad(2)  # [1,2]（默认列表被复用，出现累积）
# 正确示例
def func_good(a, li=None):
    if li is None:
        li = []
    li.append(a)
    print(li)

# 4. 可变位置参数*args
def func3(a, *args):
    print("a:", a)
    print("args:", args)  # 元组类型
func3(1, 2, 3, 4, 5)  # a=1，args=(2,3,4,5)

# 5. 强制关键字参数
def func4(a, *, b, c):
    print(a, b, c)
func4(1, b=2, c=3)  # 正确，b和c必须用关键字传递
# func4(1, 2, 3)  # 报错，位置参数无法传递*之后的参数

# 6. 可变关键字参数**kwargs
def func5(a, **kwargs):
    print("a:", a)
    print("kwargs:", kwargs)  # 字典类型
func5(1, name="张三", age=20, gender="男")

# 完整参数组合示例
def func_all(a, b=10, *args, c, d=20, **kwargs):
    print(a, b, args, c, d, kwargs)
```

### 4.3 匿名函数lambda
lambda是匿名函数，只能有一个表达式，返回值就是表达式的结果，无需写return，常用于简单逻辑、高阶函数的参数。
```python
# 语法：lambda 参数1, 参数2... : 表达式
add = lambda a, b: a + b
print(add(10, 20))  # 30

# 常用场景：sorted排序
li = [(1, 3), (4, 1), (2, 2)]
li.sort(key=lambda x: x[1])  # 按元组第二个元素排序
print(li)  # [(4, 1), (2, 2), (1, 3)]
```

### 4.4 变量作用域与LEGB规则
Python变量作用域按优先级从高到低分为4层，即LEGB规则：
1. **L(Local)**：局部作用域，函数内部定义的变量
2. **E(Enclosing)**：闭包外层函数的作用域
3. **G(Global)**：全局作用域，模块顶层定义的变量
4. **B(Built-in)**：内置作用域，Python内置的函数/变量

```python
# 全局变量
g_num = 100

def func():
    # 局部变量
    l_num = 200
    print(g_num)  # 可访问全局变量
    print(l_num)

func()
# print(l_num)  # 报错，外部无法访问局部变量

# global关键字：函数内修改全局变量必须声明global
def func2():
    global g_num
    g_num = 200  # 修改全局变量，而非创建局部变量

func2()
print(g_num)  # 200

# nonlocal关键字：闭包内修改外层函数的变量，必须声明nonlocal
def outer():
    num = 10
    def inner():
        nonlocal num
        num = 20
        print(num)
    inner()
    print(num)

outer()  # 输出20 20
```

### 4.5 函数高级特性
#### 4.5.1 闭包
内层函数引用外层函数的变量，外层函数返回内层函数，即为闭包。闭包可保留外层函数的变量状态，不会随外层函数执行结束而销毁。
```python
def outer(num1):
    # 内层函数
    def inner(num2):
        # 引用外层函数的num1
        return num1 + num2
    # 返回内层函数
    return inner

# 调用外层函数，得到内层函数的引用，保留num1=10的状态
add_10 = outer(10)
print(add_10(5))   # 15
print(add_10(20))  # 30

add_20 = outer(20)
print(add_20(5))   # 25
```

#### 4.5.2 装饰器
基于闭包实现，**在不修改原函数代码和调用方式的前提下，给原函数增加额外功能**，是Python的语法糖，常用场景：日志记录、性能统计、权限校验、缓存等。
```python
# 基础装饰器
def decorator(func):
    def wrapper(*args, **kwargs):
        # 额外功能：执行前
        print("函数开始执行")
        # 执行原函数
        result = func(*args, **kwargs)
        # 额外功能：执行后
        print("函数执行结束")
        return result
    return wrapper

# 用@语法糖应用装饰器
@decorator
def add(a, b):
    print(f"执行add函数，参数{a}, {b}")
    return a + b

# 调用原函数，自动触发装饰器功能
res = add(10, 20)
print(res)
```

#### 4.5.3 生成器函数
用`yield`关键字的函数，返回生成器对象，**惰性迭代**，调用时不会立即执行代码，只有迭代时才会执行，每次执行到yield暂停，返回值，下次迭代从暂停位置继续，极大节省内存，适合处理大数据量。
```python
# 生成器函数：生成0-n的数字
def gen_num(n):
    for i in range(n):
        yield i

# 得到生成器对象
gen = gen_num(5)
print(gen)  # <generator object gen_num at 0x...>

# 迭代生成器
print(next(gen))  # 0
print(next(gen))  # 1
for num in gen:
    print(num)  # 2,3,4
```

## 五、面向对象编程(OOP)
Python是完全面向对象的语言，所有数据都是对象，支持封装、继承、多态三大面向对象特性。

### 5.1 类与对象的定义
类是对象的模板，对象是类的实例，用`class`关键字定义类。
```python
# 定义类
class Person:
    # 类属性：所有实例共享，定义在方法外
    species = "人类"

    # 构造方法：实例化时自动调用，用于初始化实例属性
    def __init__(self, name, age):
        # 实例属性：每个实例独有，self代表当前实例
        self.name = name
        self.age = age

    # 实例方法：第一个参数必须是self
    def say_hello(self):
        print(f"大家好，我是{self.name}，今年{self.age}岁")

# 实例化对象
p1 = Person("张三", 20)
p2 = Person("李四", 22)

# 访问属性和方法
print(p1.name)  # 张三
print(p1.species)  # 人类
p1.say_hello()  # 调用方法

# 修改属性
p1.age = 21
p1.say_hello()
```

### 5.2 类的方法类型
Python类中有3种核心方法类型，区别如下：

| 方法类型 | 装饰器 | 必选参数 | 调用方式 | 可访问内容 |
|----------|--------|----------|----------|------------|
| 实例方法 | 无 | self | 只能实例调用 | 实例属性、类属性 |
| 类方法 | `@classmethod` | cls | 类/实例均可调用 | 仅类属性 |
| 静态方法 | `@staticmethod` | 无 | 类/实例均可调用 | 不能直接访问类/实例属性 |

```python
class Person:
    species = "人类"

    def __init__(self, name):
        self.name = name

    # 实例方法
    def say_name(self):
        print(f"姓名：{self.name}")

    # 类方法
    @classmethod
    def get_species(cls):
        print(f"物种：{cls.species}")

    # 静态方法
    @staticmethod
    def breathe():
        print("人类需要呼吸")

# 调用
Person.get_species()  # 类调用类方法
Person.breathe()      # 类调用静态方法

p = Person("张三")
p.say_name()    # 实例调用实例方法
p.get_species() # 实例调用类方法
p.breathe()     # 实例调用静态方法
```

### 5.3 三大面向对象特性
#### 5.3.1 封装
将属性和方法封装在类内部，通过访问权限控制，对外暴露必要的接口，隐藏内部实现细节。
Python的访问权限通过命名约定实现：
1. **公有成员**：默认命名，如`name`，类内外均可访问
2. **受保护成员**：单下划线开头，如`_age`，约定俗成，子类可访问，外部不建议直接访问
3. **私有成员**：双下划线开头，如`__gender`，Python会进行名称改写，外部无法直接访问，只能通过类内部方法访问
```python
class Person:
    def __init__(self, name, age, gender):
        self.name = name  # 公有
        self._age = age    # 受保护
        self.__gender = gender  # 私有

    # 公有方法，访问私有属性
    def get_gender(self):
        return self.__gender

p = Person("张三", 20, "男")
print(p.name)  # 正常访问
print(p._age)  # 可访问，但不建议
# print(p.__gender)  # 报错，外部无法直接访问私有属性
print(p.get_gender())  # 通过内部方法访问，正常
```

#### 5.3.2 继承
子类继承父类的所有属性和方法，可重写父类方法，也可扩展自己的属性和方法，支持单继承和多继承，减少代码冗余。
```python
# 父类（基类）
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def say_hello(self):
        print(f"我是{self.name}，今年{self.age}岁")

# 子类（派生类），继承Person
class Student(Person):
    def __init__(self, name, age, student_id):
        # 调用父类的构造方法
        super().__init__(name, age)
        # 子类独有属性
        self.student_id = student_id

    # 重写父类方法
    def say_hello(self):
        print(f"我是{self.name}，学号{self.student_id}，今年{self.age}岁")

    # 子类独有方法
    def study(self):
        print(f"{self.name}正在学习")

# 实例化子类
s = Student("张三", 20, "2024001")
s.say_hello()  # 调用重写后的方法
s.study()      # 调用子类独有方法
```

**多继承**：一个子类可继承多个父类，语法`class 子类(父类1, 父类2, ...):`，Python通过C3算法确定MRO（方法解析顺序），可通过`类名.__mro__`查看。

#### 5.3.3 多态
不同类的对象，调用同一个方法，产生不同的执行结果。Python是**鸭子类型**，不要求严格的继承关系，只要对象有对应的方法，就可以调用，无需关注对象类型。
```python
class Cat:
    def speak(self):
        print("喵喵喵")

class Dog:
    def speak(self):
        print("汪汪汪")

class Duck:
    def speak(self):
        print("嘎嘎嘎")

# 多态：统一的调用接口，不同的执行结果
def animal_speak(animal):
    animal.speak()

# 传入不同对象，执行不同方法
animal_speak(Cat())
animal_speak(Dog())
animal_speak(Duck())
```

### 5.4 魔法方法（双下划线方法）
Python内置的特殊方法，以`__`开头和结尾，无需手动调用，在特定场景下自动触发，用于自定义类的行为。
| 常用魔法方法 | 触发场景 |
|--------------|----------|
| `__init__` | 实例化对象时自动调用，初始化实例属性 |
| `__str__` | 打印对象/str()转换对象时调用，返回字符串，用于友好展示 |
| `__repr__` | repr()转换对象时调用，返回对象的官方表示 |
| `__len__` | len()函数调用对象时触发 |
| `__call__` | 对象像函数一样被调用时触发 |
| `__getitem__` | 通过索引/key取值时触发，如obj[key] |
| `__setitem__` | 通过索引/key赋值时触发，如obj[key]=value |

```python
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age

    # 打印对象时触发
    def __str__(self):
        return f"Person(name={self.name}, age={self.age})"

    # 对象可调用时触发
    def __call__(self):
        print(f"{self.name}被调用了")

p = Person("张三", 20)
print(p)  # 触发__str__，输出Person(name=张三, age=20)
p()       # 触发__call__，输出张三被调用了
```

## 六、模块与包
### 6.1 模块
一个`.py`文件就是一个Python模块，模块内可定义函数、类、变量，也可编写可执行代码，用于拆分大型项目，实现代码复用。

**模块导入方式**
```python
# 1. 导入整个模块
import math
print(math.pi)  # 调用模块内的属性/函数，需加模块名前缀

# 2. 导入模块并起别名
import numpy as np
print(np.array([1,2,3]))

# 3. 从模块中导入指定的函数/类/变量
from math import pi, sqrt
print(pi)
print(sqrt(4))  # 无需加模块名前缀

# 4. 从模块中导入所有内容（不推荐，易造成命名冲突）
from math import *
```

**`__name__`属性**
- 当模块被**直接运行**时，`__name__`的值为`__main__`
- 当模块被**导入**时，`__name__`的值为模块名
- 常用场景：编写模块的测试代码，只有直接运行模块时才执行
```python
# test.py模块
def add(a, b):
    return a + b

# 测试代码，只有直接运行test.py时才会执行
if __name__ == '__main__':
    print(add(10, 20))
```

### 6.2 包
包是包含`__init__.py`文件的文件夹，用于组织多个模块，解决模块命名冲突问题。
- `__init__.py`：包的标识文件，可为空，也可定义`__all__`变量，控制`from 包 import *`的导入内容
- 包的导入方式
  ```python
  # 导入包内的模块
  from package import module1
  # 导入包内模块的指定函数/类
  from package.module1 import func1
  ```

### 6.3 常用标准库
Python自带丰富的标准库，无需额外安装，常用如下：
| 标准库 | 用途 |
|--------|------|
| `sys` | 与Python解释器交互，如命令行参数、退出程序、模块路径 |
| `os` | 操作系统交互，如目录操作、文件路径、环境变量 |
| `time/datetime` | 时间与日期处理 |
| `json` | JSON数据序列化与反序列化 |
| `re` | 正则表达式 |
| `math/random` | 数学运算与随机数生成 |
| `collections` | 扩展数据类型，如namedtuple、defaultdict、deque |
| `itertools` | 迭代器工具，高效生成可迭代对象 |
| `functools` | 函数式编程工具，如reduce、partial、lru_cache |

## 七、异常处理
程序运行时出现的错误会触发异常，若不处理会导致程序中断，异常处理可捕获并处理错误，保证程序的健壮性。

### 7.1 基础异常处理语法
完整语法：`try-except-else-finally`
```python
try:
    # 可能出现异常的代码
    num1 = int(input("请输入第一个数字："))
    num2 = int(input("请输入第二个数字："))
    result = num1 / num2
except ValueError:
    # 捕获指定异常，处理逻辑
    print("输入的不是有效数字！")
except ZeroDivisionError:
    # 捕获多个异常分支
    print("除数不能为0！")
except Exception as e:
    # 捕获所有异常，e为异常对象，获取异常信息
    print(f"发生未知错误：{e}")
else:
    # try代码块无异常时执行
    print(f"计算结果：{result}")
finally:
    # 无论是否发生异常，都会执行，常用于资源释放
    print("程序执行结束")
```

### 7.2 主动抛出异常
用`raise`关键字主动抛出指定异常，常用于参数校验、业务逻辑错误处理。
```python
def divide(a, b):
    if b == 0:
        # 主动抛出异常，附带提示信息
        raise ZeroDivisionError("除数不能为0")
    return a / b

try:
    divide(10, 0)
except ZeroDivisionError as e:
    print(e)
```

### 7.3 自定义异常
继承Python内置的`Exception`类，可自定义异常类型，适配业务场景。
```python
# 自定义异常
class AgeError(Exception):
    """年龄输入错误异常"""
    pass

def set_age(age):
    if age < 0 or age > 150:
        raise AgeError(f"年龄{age}不合法，必须在0-150之间")
    print(f"年龄设置为{age}")

try:
    set_age(200)
except AgeError as e:
    print(e)
```

### 7.4 常用内置异常
| 异常类型 | 触发场景 |
|----------|----------|
| `SyntaxError` | 语法错误，代码不符合Python语法规范 |
| `NameError` | 访问未定义的变量 |
| `TypeError` | 操作/函数传入了不匹配的类型 |
| `ValueError` | 传入的类型正确，但值不合法 |
| `IndexError` | 序列索引越界 |
| `KeyError` | 访问字典中不存在的key |
| `AttributeError` | 访问对象不存在的属性/方法 |
| `ZeroDivisionError` | 除数为0 |
| `IOError` | 输入输出错误，如文件不存在、无法读取 |

## 八、文件操作
Python提供了完善的文件读写、目录操作功能，支持文本文件和二进制文件（图片、视频、音频等）处理。

### 8.1 文件打开与关闭
用`open()`函数打开文件，返回文件对象，操作完成后必须关闭文件，**推荐使用with语句（上下文管理器），自动关闭文件，无需手动调用close()**。
```python
# 语法：open(file, mode='r', encoding=None)
# file：文件路径，mode：打开模式，encoding：编码格式（文本文件推荐utf-8）

# 1. 传统方式：手动关闭
f = open("test.txt", "r", encoding="utf-8")
content = f.read()
f.close()  # 必须关闭，否则会占用资源

# 2. 推荐方式：with语句，自动关闭
with open("test.txt", "r", encoding="utf-8") as f:
    content = f.read()
# 跳出with代码块，文件自动关闭
```

### 8.2 文件打开模式mode
| 模式 | 说明 |
|------|------|
| `r` | 只读模式（默认），文件不存在报错，指针在文件开头 |
| `w` | 只写模式，文件不存在则创建，存在则清空原有内容 |
| `a` | 追加模式，文件不存在则创建，存在则在末尾追加内容，指针在文件结尾 |
| `r+` | 读写模式，文件不存在报错，可读写，指针在开头 |
| `w+` | 读写模式，文件不存在创建，存在则清空，可读写 |
| `a+` | 读写追加模式，文件不存在创建，指针在结尾，读从开头，写在末尾 |
| `b` | 二进制模式，配合上述模式使用，如`rb`/`wb`/`ab`，用于非文本文件 |

### 8.3 文件读写方法
```python
# 1. 文件读取
with open("test.txt", "r", encoding="utf-8") as f:
    content1 = f.read()  # 一次性读取全部内容，返回字符串
    content2 = f.readline()  # 读取一行内容
    content3 = f.readlines() # 读取所有行，返回列表，每个元素是一行

# 2. 文件写入
with open("test.txt", "w", encoding="utf-8") as f:
    f.write("hello world\n")  # 写入字符串，\n换行
    f.writelines(["第一行\n", "第二行\n", "第三行\n"])  # 写入列表，不会自动加换行
```

### 8.4 目录与路径操作
常用`os`模块和`pathlib`模块（Python3.4+，面向对象，更简洁）处理目录和路径。
```python
# os模块常用操作
import os

print(os.getcwd())  # 获取当前工作目录
os.mkdir("test_dir")  # 创建单级目录
os.makedirs("a/b/c")  # 创建多级目录
os.rmdir("test_dir")  # 删除空目录
os.removedirs("a/b/c") # 删除多级空目录
print(os.listdir())  # 列出当前目录下的所有文件和目录
os.rename("old.txt", "new.txt")  # 重命名文件/目录
print(os.path.exists("test.txt"))  # 判断文件/目录是否存在
print(os.path.isfile("test.txt"))  # 判断是否是文件
print(os.path.isdir("test_dir"))   # 判断是否是目录
print(os.path.join("/home/user", "test.txt"))  # 拼接路径

# pathlib模块（推荐）
from pathlib import Path

path = Path("/home/user/test.txt")
print(path.exists())  # 判断是否存在
print(path.is_file()) # 判断是否是文件
print(path.parent)    # 获取父目录
print(path.name)      # 获取文件名
print(path.suffix)    # 获取文件后缀
new_path = path.parent / "new.txt"  # 路径拼接，用/运算符
```

## 九、Python高级核心特性
### 9.1 推导式
Python特有的简洁语法，一行代码快速生成数据结构，比for循环更高效简洁，支持4种推导式。
```python
# 1. 列表推导式：[表达式 for 变量 in 可迭代对象 if 条件]
# 生成0-9的偶数的平方
li = [i**2 for i in range(10) if i % 2 == 0]
print(li)  # [0, 4, 16, 36, 64]

# 2. 字典推导式：{key表达式: value表达式 for 变量 in 可迭代对象 if 条件}
# 生成数字到平方的映射
d = {i: i**2 for i in range(5)}
print(d)  # {0:0, 1:1, 2:4, 3:9, 4:16}

# 3. 集合推导式：{表达式 for 变量 in 可迭代对象 if 条件}
# 去重并生成偶数集合
s = {i for i in [1,2,2,3,3,4,5] if i % 2 == 0}
print(s)  # {2,4}

# 4. 生成器表达式：(表达式 for 变量 in 可迭代对象 if 条件)
# 惰性计算，返回生成器，节省内存
gen = (i**2 for i in range(10) if i % 2 == 0)
print(gen)  # <generator object <genexpr> at 0x...>
for num in gen:
    print(num)
```

### 9.2 迭代器与可迭代对象
1. **可迭代对象**：实现了`__iter__`方法的对象，如列表、元组、字符串、字典、集合、生成器，可被for循环遍历，用`isinstance(obj, Iterable)`判断。
2. **迭代器**：实现了`__iter__`和`__next__`方法的对象，`__next__`返回下一个值，无元素时抛出`StopIteration`异常，惰性计算，只能向前遍历，不能后退。
```python
from collections.abc import Iterable, Iterator

li = [1,2,3]
print(isinstance(li, Iterable))  # True，列表是可迭代对象
print(isinstance(li, Iterator))  # False，列表不是迭代器

# iter()将可迭代对象转为迭代器
it = iter(li)
print(isinstance(it, Iterator))  # True
print(next(it))  # 1
print(next(it))  # 2
print(next(it))  # 3
# next(it)  # 无元素，抛出StopIteration
```

### 9.3 解包操作
Python支持对可迭代对象进行解包，简化赋值和参数传递，`*`解包可迭代对象，`**`解包字典。
```python
# 1. 变量解包
a, b, c = [1,2,3]
print(a, b, c)  # 1 2 3

# 2. 多变量解包，*接收剩余元素
a, *b, c = [1,2,3,4,5]
print(a)  # 1
print(b)  # [2,3,4]
print(c)  # 5

# 3. 函数参数解包
def add(a, b, c):
    return a + b + c

li = [1,2,3]
print(add(*li))  # 等价于add(1,2,3)

d = {"a":1, "b":2, "c":3}
print(add(**d))  # 等价于add(a=1, b=2, c=3)

# 4. 容器合并
li1 = [1,2,3]
li2 = [4,5,6]
li3 = [*li1, *li2]  # [1,2,3,4,5,6]

d1 = {"a":1, "b":2}
d2 = {"c":3, "d":4}
d3 = {**d1, **d2}  # {'a':1, 'b':2, 'c':3, 'd':4}
```

### 9.4 深拷贝与浅拷贝
针对可变对象，拷贝分为赋值、浅拷贝、深拷贝，区别在于是否拷贝嵌套的可变对象。
```python
import copy

# 1. 赋值：仅传递引用，指向同一个内存地址，修改一个另一个必变
li1 = [1,2, [3,4]]
li2 = li1
li2[0] = 100
li2[2][0] = 300
print(li1)  # [100, 2, [300, 4]]，原列表被修改

# 2. 浅拷贝copy.copy()：拷贝外层对象，内层嵌套的可变对象仍共享引用
li1 = [1,2, [3,4]]
li2 = copy.copy(li1)
li2[0] = 100  # 外层修改，原列表不变
li2[2][0] = 300  # 内层修改，原列表同步变化
print(li1)  # [1, 2, [300, 4]]

# 3. 深拷贝copy.deepcopy()：完全拷贝，内外层所有对象都独立，修改互不影响
li1 = [1,2, [3,4]]
li2 = copy.deepcopy(li1)
li2[0] = 100
li2[2][0] = 300
print(li1)  # [1, 2, [3, 4]]，原列表完全不受影响
```

### 9.5 类型注解
Python3.5+支持类型注解，给变量、函数参数、返回值添加类型提示，不会强制类型检查，仅用于代码提示、可读性提升，可配合`mypy`工具做静态类型检查。
```python
from typing import List, Dict, Tuple, Optional, Any

# 1. 变量类型注解
name: str = "张三"
age: int = 20
height: Optional[float] = None  # 可选类型，可为float或None

# 2. 函数类型注解
def add(a: int, b: int) -> int:
    """两个int相加，返回int"""
    return a + b

# 复杂类型注解
def get_user_info(user_id: int) -> Dict[str, Any]:
    """根据用户ID获取用户信息，返回字典"""
    return {"user_id": user_id, "name": "张三", "age": 20}

def get_numbers() -> Tuple[int, str, float]:
    """返回元组"""
    return 1, "hello", 3.14

def process_list(li: List[int]) -> List[int]:
    """处理int列表，返回int列表"""
    return [i*2 for i in li]
```

### 9.6 上下文管理器
上下文管理器用于自动管理资源的获取和释放，实现了`__enter__`和`__exit__`方法的对象即为上下文管理器，通过`with`语句使用，避免资源泄漏，常用场景：文件操作、数据库连接、锁操作等。
```python
# 自定义上下文管理器
class MyContext:
    def __enter__(self):
        # 进入with代码块时执行，返回值赋值给as后的变量
        print("资源获取")
        return "资源对象"

    def __exit__(self, exc_type, exc_val, exc_tb):
        # 退出with代码块时执行，无论是否异常都会执行
        # exc_type/exc_val/exc_tb为异常信息，无异常则为None
        print("资源释放")
        return True  # 返回True可抑制异常向外抛出

# 使用
with MyContext() as res:
    print(res)
    print("业务逻辑执行")
```

也可通过`@contextmanager`装饰器简化上下文管理器实现，无需定义类：
```python
from contextlib import contextmanager

@contextmanager
def my_context():
    # __enter__部分：yield之前的代码
    print("资源获取")
    res = "资源对象"
    try:
        yield res  # 暂停，执行with内的代码，返回值给as后的变量
    finally:
        # __exit__部分：yield之后的代码，无论是否异常都会执行
        print("资源释放")

# 使用
with my_context() as res:
    print(res)
    print("业务逻辑执行")
```


