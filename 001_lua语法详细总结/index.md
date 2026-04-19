# Lua 语法详细总结


# Lua语法详细总结
Lua是一款**轻量级、高效、可嵌入的脚本语言**，基于标准C实现，跨平台性强，核心设计目标是为应用程序提供灵活的扩展能力，语法简洁优雅，原生支持面向对象、函数式编程、协程等特性，主流稳定版本为Lua 5.4。

---

## 一、基础语法规范
### 1. 注释
```lua
-- 单行注释

--[[
多行注释
支持换行
]]

--[=[
可嵌套的多行注释
避免内部的]]提前闭合
]=]
```

### 2. 标识符与关键字
- **标识符**：由字母、数字、下划线组成，不能以数字开头，**区分大小写**，不能与关键字重名，推荐下划线+小写的命名风格。
- **关键字**（Lua保留字，共22个）：
  ```
  and  break  do  else  elseif  end  false  for  function  goto  if
  in  local  nil  not  or  repeat  return  then  true  until  while
  ```

### 3. 语句与分隔符
- 语句结束的分号`;`是**可选**的，换行不会被识别为语句结束。
- 多个语句可写在同一行，推荐用分号分隔，提升可读性。
  ```lua
  local a = 1 local b = 2 -- 合法
  local a = 1; local b = 2 -- 推荐
  ```

### 4. 变量
Lua变量分为**全局变量**和**局部变量**，变量本身无类型，仅存储值的引用。
- **全局变量**：默认所有未声明的变量都是全局变量，赋值即创建，赋值`nil`可删除全局变量。
  ```lua
  g_var = 10 -- 全局变量
  g_var = nil -- 删除该全局变量
  ```
- **局部变量**：用`local`关键字声明，作用域仅限当前代码块（函数、循环、do-end块），访问效率远高于全局变量，推荐优先使用。
  ```lua
  local l_var = 20 -- 局部变量
  do
      local inner_var = 30 -- 仅在do-end块内有效
  end
  ```
- **多变量赋值**：Lua原生支持多值赋值，右侧值数量多于左侧时自动忽略，少于左侧时剩余变量赋值`nil`。
  ```lua
  local a, b = 1, 2 -- a=1, b=2
  a, b = b, a -- 快速交换变量，无需中间值
  local x, y, z = 10, 20 -- x=10, y=20, z=nil
  ```

---

## 二、8种基本数据类型
Lua是动态类型语言，值有固定类型，变量无类型约束，用`type()`函数可获取值的类型名称（字符串）。

| 类型 | 说明 | 核心特性 |
| :--- | :--- | :--- |
| `nil` | 空类型 | 唯一值为`nil`，代表无效值；未赋值的变量默认值为`nil`；条件判断中为假 |
| `boolean` | 布尔类型 | 仅`true`和`false`两个值；**仅nil和false为假，0、空字符串、空表均为真**（与多数语言不同） |
| `number` | 数值类型 | Lua5.3+分为64位整数integer和双精度浮点数float；支持十进制、十六进制(0x开头)、科学计数法(3e5) |
| `string` | 字符串类型 | 不可变的字节序列，支持单引号、双引号、长括号定义；UTF-8编码友好，不可修改，操作均返回新字符串 |
| `function` | 函数类型 | 一等公民，可赋值给变量、作为参数/返回值，支持匿名函数、闭包、尾调用优化 |
| `table` | 表类型 | Lua唯一的内置数据结构，关联数组，可同时作为数组/哈希表使用，引用传递，是实现OOP、模块的核心 |
| `userdata` | 用户数据 | 用于存储C/C++中的结构体数据，Lua层面无法直接修改，仅能通过元方法操作，用于扩展能力 |
| `thread` | 线程类型 | 对应Lua协程(coroutine)，协作式多任务，非抢占式，用于实现异步逻辑、状态机等 |

### 重点类型详解
1. **string字符串**
   ```lua
   -- 三种定义方式
   local s1 = '单引号字符串'
   local s2 = "双引号字符串\n支持转义"
   local s3 = [[长括号字符串
   支持换行，不会转义字符
   可用于多行文本]]

   -- 核心操作
   local s = "hello" .. " world" -- 字符串拼接用..，禁止用+（+为数值加法）
   local len = #s -- 获取字符串字节长度，#"hello"=5，UTF-8中文单字占3字节
   ```

2. **table表**
   Lua表的索引可以是除`nil`外的任意类型，**数组默认索引从1开始**（而非0），是Lua最核心的数据结构。
   ```lua
   -- 空表
   local t1 = {}

   -- 数组型表（列表），连续整数索引，默认从1开始
   local arr = {10, 20, 30, 40}
   print(arr[1]) -- 10，而非arr[0]

   -- 字典型表（哈希表），键值对结构
   local person = {
       name = "Lua", -- 等价于["name"] = "Lua"
       version = 5.4,
       ["full name"] = "Lua Programming Language" -- 特殊键名必须用[]包裹
   }
   -- 访问方式
   print(person.name) -- 等价于person["name"]
   print(person["full name"]) -- 特殊键名仅能通过[]访问

   -- 核心特性：引用传递
   local t_a = {a=1, b=2}
   local t_b = t_a -- 仅传递引用，不复制表
   t_b.a = 10
   print(t_a.a) -- 10，原表被修改

   -- 赋值nil删除元素
   person.version = nil
   ```

---

## 三、运算符
### 1. 算术运算符
| 运算符 | 说明 | 注意事项 |
| :--- | :--- | :--- |
| `+` | 加法 | 仅用于数值运算，字符串会自动尝试转数值 |
| `-` | 减法/负号 | 二元为减法，一元为负号 |
| `*` | 乘法 | - |
| `/` | 浮点除法 | 5/2=2.5，永远返回浮点数 |
| `//` | 整数除法 | 向下取整，5//2=2，Lua5.3+支持 |
| `%` | 取模运算 | 取余数，支持浮点数 |
| `^` | 幂运算 | 2^3=8，优先级最高 |

> 注意：Lua**不支持++、--、+=、-=**等复合赋值运算符。

### 2. 关系运算符
| 运算符 | 说明 | 核心规则 |
| :--- | :--- | :--- |
| `==` | 等于 | 不同类型值比较永远为false；表/函数/userdata比较引用地址，{}=={}为false |
| `~=` | 不等于 | 等价于not ==，**禁止使用!=** |
| `>` `<` | 大于/小于 | 仅数值和字符串可比较 |
| `>=` `<=` | 大于等于/小于等于 | 同上 |

### 3. 逻辑运算符
核心特性：**短路求值**，仅在必要时计算第二个操作数，返回值为操作数本身，而非固定的boolean值。
| 运算符 | 说明 | 规则 |
| :--- | :--- | :--- |
| `and` | 逻辑与 | 第一个操作数为假，返回第一个值；否则返回第二个值 |
| `or` | 逻辑或 | 第一个操作数为真，返回第一个值；否则返回第二个值 |
| `not` | 逻辑非 | 永远返回true/false，取反操作 |

```lua
-- 经典用法：设置默认值
local x = nil
x = x or 10 -- x=10，若x有非假值则保留原值

-- 模拟三目运算符：a and b or c，注意b不能为假值
local max = a > b and a or b
```

### 4. 其他运算符
- `..`：字符串拼接运算符，数字会自动转字符串，`123.."abc"`结果为`"123abc"`，注意数字后必须加空格，避免被识别为小数点。
- `#`：长度运算符，用于字符串获取字节长度，用于数组型表获取连续整数索引的长度，遇到nil终止。

### 5. 运算符优先级（从高到低）
```
^
not  -（负号）
*  /  //  %
+  -
..
<  >  <=  >=  ~=  ==
and
or
```
> 可通过括号`()`改变优先级，复杂表达式推荐用括号提升可读性。

---

## 四、流程控制语句
### 1. 分支结构：if语句
Lua仅支持if-elseif-else分支，**原生不支持switch-case**（可通过表模拟），语法结构：
```lua
if 条件表达式1 then
    -- 条件1为真时执行
elseif 条件表达式2 then
    -- 条件2为真时执行
else
    -- 所有条件为假时执行
end
```
示例：
```lua
local score = 85
if score >= 90 then
    print("优秀")
elseif score >= 60 then
    print("及格")
else
    print("不及格")
end
```

### 2. 循环结构
Lua支持4种循环方式，核心循环控制语句为`break`（跳出当前循环），**原生不支持continue**（可通过goto/if模拟）。

#### （1）while循环
先判断条件，条件为真时执行循环体，语法：
```lua
while 条件表达式 do
    -- 循环体
end
```
示例：
```lua
local i = 1
while i <= 5 do
    print(i)
    i = i + 1
end
```

#### （2）repeat-until循环
先执行循环体，再判断条件，**条件为真时退出循环**（与do-while相反），循环体内的变量在until条件中可见，语法：
```lua
repeat
    -- 循环体
until 条件表达式
```
示例：
```lua
local i = 1
repeat
    print(i)
    i = i + 1
until i > 5
```

#### （3）数值for循环
固定次数的循环，初始值、终止值、步长仅在循环开始前计算一次，循环变量为局部变量，仅在循环体内有效，语法：
```lua
for 变量 = 初始值, 终止值, 步长 do
    -- 循环体
end
```
> 步长可选，默认值为1，可设置负数实现倒序循环。

示例：
```lua
-- 正序循环，步长1
for i = 1, 5 do
    print(i)
end

-- 倒序循环，步长-2
for i = 10, 1, -2 do
    print(i)
end
```

#### （4）泛型for循环
通过迭代器遍历数据，最常用的是`ipairs`和`pairs`迭代器，语法：
```lua
for 变量列表 in 迭代器 do
    -- 循环体
end
```
核心迭代器区别：
- `ipairs(t)`：仅遍历数组型表的**连续整数索引（从1开始）**，遇到nil立即终止，按索引顺序遍历。
- `pairs(t)`：遍历表的**所有键值对**，包括非整数索引，不受nil影响，遍历顺序不固定。

示例：
```lua
-- ipairs遍历数组
local arr = {10, 20, 30, 40}
for i, v in ipairs(arr) do
    print(i, v) -- 输出索引和对应值
end

-- pairs遍历通用表
local tab = {name="Lua", version=5.4, 10, 20}
for k, v in pairs(tab) do
    print(k, v) -- 输出所有键值对
end
```

### 3. goto语句
Lua5.2+支持goto语句，用于跳转到指定标签，可用于模拟continue、跳出多层循环等场景，语法：
```lua
::标签名:: -- 定义标签
goto 标签名 -- 跳转到标签
```
示例（模拟continue）：
```lua
for i = 1, 10 do
    if i % 2 == 0 then
        goto continue -- 偶数跳过，模拟continue
    end
    print(i)
    ::continue:: -- 标签位置
end
```
> 限制：不能跳转到函数外部、不能跳转到局部变量的作用域内。

---

## 五、函数
Lua中函数是**一等公民**，支持匿名函数、多返回值、可变参数、闭包、尾调用优化等特性。

### 1. 函数定义与调用
```lua
-- 命名函数（全局）
function 函数名(参数列表)
    -- 函数体
    return 返回值列表
end

-- 局部函数（推荐）
local function 函数名(参数列表)
    -- 函数体
end

-- 等价于：匿名函数赋值给局部变量
local 函数名 = function(参数列表)
    -- 函数体
end
```
示例：
```lua
-- 定义加法函数
local function add(a, b)
    return a + b
end

-- 调用函数
local res = add(1, 2)
```

### 2. 核心特性
#### （1）多返回值
Lua函数可返回多个值，用逗号分隔，接收时可按需接收，多余返回值自动忽略，不足则补nil。
```lua
-- 多返回值函数
local function swap(a, b)
    return b, a
end

local x, y = swap(1, 2) -- x=2, y=1

-- 多返回值规则：非表达式末尾仅返回第一个值
local function foo() return 1, 2, 3 end
local a, b = foo(), 10 -- a=1, b=10
local c, d, e = foo() -- c=1, d=2, e=3
local t = {foo()} -- t={1,2,3}，表构造器保留所有返回值
local t2 = {(foo())} -- t2={1}，括号强制仅返回第一个值
```

#### （2）可变参数
用`...`表示可变参数，可接收任意数量的实参，常用于参数数量不固定的场景。
```lua
-- 可变参数求和
local function sum(...)
    local total = 0
    -- 可变参数转为表遍历
    for _, v in ipairs({...}) do
        total = total + v
    end
    return total
end

print(sum(1,2,3,4)) -- 10

-- 辅助操作
local function test(...)
    local arg_num = select("#", ...) -- 获取可变参数个数
    local first_arg = select(1, ...) -- 获取第n个及之后的参数
    print(arg_num, first_arg)
end
```

#### （3）闭包
闭包是指一个内部函数可以访问其外部函数的局部变量（upvalue/上值），即使外部函数已执行结束，upvalue依然会被保留，每个闭包拥有独立的upvalue实例。
```lua
-- 计数器闭包
local function new_counter()
    local count = 0 -- upvalue
    -- 返回匿名函数（闭包）
    return function()
        count = count + 1
        return count
    end
end

local c1 = new_counter()
print(c1()) -- 1
print(c1()) -- 2

local c2 = new_counter()
print(c2()) -- 1，独立的upvalue，互不影响
```

#### （4）尾调用优化
尾调用是指函数的最后一个动作是调用另一个函数，Lua会对尾调用进行优化，不会占用额外的栈空间，不会出现栈溢出，常用于递归优化。
```lua
-- 尾递归优化的阶乘函数
local function fact(n, acc)
    acc = acc or 1
    if n == 0 then return acc end
    return fact(n-1, n * acc) -- 纯尾调用，无额外运算
end

print(fact(5)) -- 120
```
> 注意：仅`return 函数(...)`为纯尾调用，`return 1 + f()`、`return f() + 1`等均不属于尾调用，无法优化。

### 3. 点号与冒号的区别
Lua中函数定义和调用支持`.`和`:`两种方式，核心区别是**冒号会自动将调用者作为第一个参数self传入**，是Lua实现面向对象的核心语法。
```lua
local obj = {name = "Lua"}

-- 点号定义，需手动接收self
function obj.say_hello(self)
    print("Hello, " .. self.name)
end
obj.say_hello(obj) -- 手动传入调用者

-- 冒号定义，自动接收self参数
function obj:say_hi()
    print("Hi, " .. self.name)
end
obj:say_hi() -- 自动将obj作为self传入，等价于obj.say_hi(obj)
```

---

## 六、元表与元方法
元表（metatable）是Lua的高级特性，用于**修改表/用户数据的默认行为**，实现运算符重载、索引拦截、面向对象、只读表等能力。元表是普通的表，其中的键值对称为元方法（metamethod）。

### 1. 核心操作函数
- `setmetatable(t, mt)`：为表`t`设置元表`mt`，返回表`t`。
- `getmetatable(t)`：获取表`t`的元表，无元表返回`nil`。

```lua
local t = {}
local mt = {} -- 元表
setmetatable(t, mt) -- 为t设置元表mt
```

### 2. 常用元方法
#### （1）算术/关系运算符元方法
用于重载运算符，实现自定义运算逻辑，常用如下：
| 元方法 | 对应运算符 | 说明 |
| :--- | :--- | :--- |
| `__add` | `+` | 加法重载 |
| `__sub` | `-` | 减法重载 |
| `__mul` | `*` | 乘法重载 |
| `__div` | `/` | 除法重载 |
| `__mod` | `%` | 取模重载 |
| `__pow` | `^` | 幂运算重载 |
| `__concat` | `..` | 字符串拼接重载 |
| `__eq` | `==` | 等于重载 |
| `__lt` | `<` | 小于重载 |
| `__le` | `<=` | 小于等于重载 |

示例：实现两个表的加法
```lua
local mt = {
    __add = function(a, b)
        local res = {}
        for i = 1, #a do
            res[i] = a[i] + b[i]
        end
        return res
    end
}

local t1 = {1, 2, 3}
local t2 = {4, 5, 6}
setmetatable(t1, mt)

local t3 = t1 + t2 -- t3 = {5,7,9}
```

#### （2）索引访问元方法
最核心的元方法，用于拦截表的索引访问和赋值，是实现面向对象继承的核心。
- `__index`：当访问表中**不存在的键**时触发，可设置为函数或表。
  - 若为函数：参数为`(表, 键)`，返回值为访问结果。
  - 若为表：自动在该表中查找对应的键，实现原型链继承。
  ```lua
  -- 函数形式：设置默认值
  local mt = {
      __index = function(t, k)
          return "默认值"
      end
  }
  local t = {a=1}
  setmetatable(t, mt)
  print(t.a) -- 1，键存在，不触发__index
  print(t.b) -- 默认值，键不存在，触发__index

  -- 表形式：实现继承
  local parent = {x=10, y=20}
  local child = {}
  setmetatable(child, {__index = parent})
  print(child.x) -- 10，child无x，去parent中查找
  ```

- `__newindex`：当给表中**不存在的键赋值**时触发，可设置为函数或表，用于拦截赋值、实现只读表等。
  ```lua
  -- 实现只读表
  local function read_only(t)
      local mt = {
          __newindex = function()
              error("只读表，禁止修改")
          end,
          __index = t
      }
      return setmetatable({}, mt)
  end

  local t = read_only({a=1, b=2})
  print(t.a) -- 1
  t.a = 10 -- 触发__newindex，抛出错误
  ```

#### （3）其他常用元方法
| 元方法 | 触发场景 | 说明 |
| :--- | :--- | :--- |
| `__tostring` | 表被转为字符串时（如print(t)） | 自定义表的字符串输出格式 |
| `__len` | 使用#运算符取表长度时 | 自定义长度计算逻辑 |
| `__call` | 把表当作函数调用时（如t()） | 让表具备函数执行能力 |
| `__metatable` | 获取/设置元表时 | 保护元表，设置后getmetatable返回该值，setmetatable报错 |

示例：`__tostring`自定义表输出
```lua
local mt = {
    __tostring = function(t)
        return "表内容：" .. table.concat(t, ", ")
    end
}
local t = {1, 2, 3}
setmetatable(t, mt)
print(t) -- 输出：表内容：1, 2, 3
```

---

## 七、面向对象编程（OOP）
Lua原生没有类的概念，通过**表+元表+__index元方法**实现原型链面向对象，核心是类与实例、继承、方法重写。

### 1. 类与实例的实现
```lua
-- 定义Person类（原型表）
Person = {
    name = "",
    age = 0
}

-- 构造函数
function Person:new(name, age)
    -- 创建实例对象
    local obj = {}
    -- 设置实例的元表为类本身
    setmetatable(obj, self)
    -- 找不到的属性/方法，去类中查找
    self.__index = self
    -- 初始化实例属性
    obj.name = name
    obj.age = age
    return obj
end

-- 定义成员方法（冒号自动传递self）
function Person:say_hello()
    print("你好，我是" .. self.name .. "，今年" .. self.age .. "岁")
end

-- 创建实例
local p1 = Person:new("张三", 20)
local p2 = Person:new("李四", 25)

p1:say_hello() -- 你好，我是张三，今年20岁
p2:say_hello() -- 你好，我是李四，今年25岁
```

### 2. 继承的实现
通过设置子类的元表__index指向父类，实现属性和方法的继承，支持方法重写和新增。
```lua
-- 定义子类Student，继承Person
Student = Person:new() -- 以父类实例为子类原型

-- 子类构造函数
function Student:new(name, age, grade)
    -- 调用父类构造函数
    local obj = Person:new(name, age)
    -- 设置子类元表
    setmetatable(obj, self)
    self.__index = self
    -- 子类独有属性
    obj.grade = grade
    return obj
end

-- 重写父类方法
function Student:say_hello()
    print("你好，我是" .. self.name .. "，今年" .. self.age .. "岁，读" .. self.grade .. "年级")
end

-- 子类新增方法
function Student:study()
    print(self.name .. "正在学习")
end

-- 创建子类实例
local s1 = Student:new("小明", 10, 3)
s1:say_hello() -- 重写后的方法
s1:study() -- 子类独有方法
```

---

## 八、模块与包
Lua的模块本质是一个包含函数、变量的table，通过`require()`函数加载，实现代码的复用和封装。

### 1. 模块的定义
创建模块文件`mymodule.lua`，推荐用`return table`的方式定义模块（兼容所有版本，废弃的module()方式不推荐）：
```lua
-- mymodule.lua
-- 定义模块表
local mymodule = {}

-- 模块常量
mymodule.version = "1.0.0"

-- 模块函数
function mymodule.add(a, b)
    return a + b
end

function mymodule.hello()
    print("Hello from Lua Module")
end

-- 私有函数（local声明，外部无法访问）
local function private_func()
    print("私有函数")
end

-- 返回模块表
return mymodule
```

### 2. 模块的加载与使用
通过`require()`函数加载模块，核心特性：
1. 模块**仅会被加载一次**，多次调用require返回第一次加载的结果，不会重复执行。
2. 加载的模块会缓存到`package.loaded`表中，赋值`nil`可卸载模块。
3. 按`package.path`中的路径搜索模块文件。

```lua
-- 加载模块
local mod = require("mymodule")

-- 使用模块内容
mod.hello()
print(mod.add(1, 2))
print(mod.version)

-- 卸载模块
package.loaded["mymodule"] = nil
```

### 3. 包的定义
包是多个模块组成的目录，目录中需包含`init.lua`作为包的入口文件，`require("包名")`会自动加载该目录下的`init.lua`。

---

## 九、协程（coroutine）
Lua协程是**协作式多任务**，而非抢占式，一个协程运行时，仅能主动通过`yield()`让出控制权，其他协程才能执行，不会被系统强制打断，无线程安全问题，常用于异步编程、迭代器、状态机等场景。

### 1. 协程的状态
- **suspended（挂起）**：协程创建后的初始状态，可通过resume启动。
- **running（运行）**：协程正在执行。
- **normal（正常）**：协程A唤醒协程B，A处于normal状态，直到B执行结束。
- **dead（死亡）**：协程执行完毕或报错，无法再次唤醒。

### 2. 核心API
所有协程API均在`coroutine`表中：
| API | 说明 |
| :--- | :--- |
| `coroutine.create(f)` | 创建协程，参数为协程主函数，返回协程对象，初始状态suspended |
| `coroutine.resume(co, ...)` | 启动/继续执行协程co，参数传递给协程函数，返回值：是否成功 + 结果/yield参数 |
| `coroutine.yield(...)` | 挂起当前协程，参数作为resume的返回值，下次resume的参数会作为yield的返回值 |
| `coroutine.status(co)` | 返回协程的当前状态 |
| `coroutine.running()` | 返回当前正在运行的协程对象 |

### 3. 基础示例
```lua
-- 创建协程
local co = coroutine.create(function(a, b)
    print("协程启动，参数：", a, b)
    -- 挂起协程，返回两个值给resume
    local c, d = coroutine.yield(a + b, a - b)
    print("协程继续执行，参数：", c, d)
    return c * d
end)

-- 第一次启动协程
print(coroutine.resume(co, 10, 20))
-- 输出：
-- 协程启动，参数：10 20
-- true 30 -10

-- 查看协程状态
print(coroutine.status(co)) -- suspended

-- 第二次继续协程
print(coroutine.resume(co, 5, 6))
-- 输出：
-- 协程继续执行，参数：5 6
-- true 30

-- 协程执行完毕
print(coroutine.status(co)) -- dead
```

---

## 十、常用标准库
Lua标准库精简高效，核心库无需require可直接使用，部分库需手动require。

### 1. 基础全局库
无需require，直接使用，核心函数：
- `print(...)`：打印内容，自动换行。
- `type(v)`：返回值的类型名称（字符串）。
- `pairs(t)`/`ipairs(t)`：表迭代器。
- `tonumber(s)`/`tostring(v)`：类型转换。
- `setmetatable()`/`getmetatable()`：元表操作。
- `error(msg)`：抛出错误，终止执行。
- `pcall(f, ...)`：保护模式调用函数，捕获错误，返回是否成功+结果。
- `xpcall(f, err_handler, ...)`：带错误处理函数的保护调用。
- `require(mod)`：加载模块。
- `next(t, k)`：表遍历底层实现，返回下一个键值对。

### 2. table库
用于表的操作，前缀`table.`：
- `table.insert(t, [pos,] val)`：在表的pos位置插入值，pos默认末尾。
- `table.remove(t, [pos])`：删除pos位置的元素，返回被删除值，pos默认末尾。
- `table.concat(t, [sep,] [i,] [j])`：将表元素用分隔符拼接为字符串。
- `table.sort(t, [comp])`：对表进行排序，comp为比较函数，默认升序。
- `table.unpack(t, [i,] [j])`：将表解包为多个返回值。
- `table.move(a1, f, e, t, a2)`：表元素移动/复制。

### 3. string库
用于字符串操作，前缀`string.`，所有操作均不修改原字符串，返回新字符串：
- `string.len(s)`：返回字符串长度，等价于#s。
- `string.sub(s, i, [j])`：截取子串，i/j支持负数，-1为最后一个字符。
- `string.upper(s)`/`string.lower(s)`：大小写转换。
- `string.find(s, pattern, [init,] [plain])`：查找模式，返回起始/结束位置。
- `string.match(s, pattern, [init])`：匹配模式，返回匹配内容。
- `string.gmatch(s, pattern)`：返回迭代器，遍历所有匹配结果。
- `string.gsub(s, pattern, repl, [n])`：字符串替换，返回新字符串和替换次数。
- `string.format(fmt, ...)`：格式化字符串，支持%d、%s、%f等格式符。

> Lua支持模式匹配（非正则），核心元字符：`.`任意字符、`%d`数字、`%a`字母、`%s`空白、`%w`字母数字、`*`贪婪匹配、`+`一次或多次、`-`非贪婪匹配、`?`0次或1次、`[]`字符集、`%`转义。

### 4. math库
数学运算库，前缀`math.`：
- `math.abs(x)`：绝对值。
- `math.floor(x)`/`math.ceil(x)`：向下/向上取整。
- `math.modf(x)`：拆分整数和小数部分。
- `math.sqrt(x)`：平方根。
- `math.random([m,] [n])`：随机数生成。
- `math.randomseed(x)`：设置随机数种子，推荐用`os.time()`初始化。
- `math.sin/cos/tan/asin/acos/atan`：三角函数（弧度制）。
- `math.max(...)`/`math.min(...)`：最大值/最小值。
- `math.pi`：圆周率常量，`math.huge`：无穷大常量。

### 5. io库
文件输入输出库，前缀`io.`，支持简单模式和完全模式：
- 简单模式：操作默认输入输出流，`io.read()`、`io.write()`、`io.input()`、`io.output()`。
- 完全模式：打开文件获取句柄，通过句柄操作，核心函数：
  ```lua
  -- 打开文件，模式：r只读、w只写、a追加、r+读写、b二进制模式
  local f = io.open("test.txt", "r")
  if f then
      local content = f:read("*a") -- *a读取全部、*l读取一行、*n读取数字
      print(content)
      f:close() -- 关闭文件
  end

  -- 写入文件
  local f = io.open("test.txt", "w")
  if f then
      f:write("Hello Lua")
      f:close()
  end
  ```

### 6. os库
操作系统相关库，前缀`os.`：
- `os.time([tab])`：返回当前时间戳（秒），可通过时间表指定时间。
- `os.date([fmt,] [timestamp])`：时间戳格式化，`%Y-%m-%d %H:%M:%S`为常用格式，`*t`返回时间表。
- `os.clock()`：返回程序运行的CPU时间，用于性能计时。
- `os.execute(cmd)`：执行系统命令。
- `os.getenv(var)`：获取环境变量。
- `os.remove(filename)`：删除文件。
- `os.rename(old, new)`：重命名文件/目录。
- `os.exit(code)`：退出程序。

---

## 十一、常见语法坑与注意事项
1. **数组索引从1开始**，而非0，是Lua最经典的新手坑。
2. **条件判断规则**：仅`nil`和`false`为假，0、空字符串、空表均为真，与多数语言不同。
3. **不等于运算符是`~=`**，禁止使用`!=`，Lua不识别。
4. **字符串拼接用`..`**，`+`仅用于数值加法，`"1"+"2"`会转为数字3，而非字符串"12"。
5. **全局变量污染**：未用`local`声明的变量均为全局变量，推荐优先使用局部变量，效率更高且无污染。
6. **#运算符限制**：仅对连续无nil的数组型表有效，含nil的数组或哈希表，#的结果不可预测。
7. **表的引用传递**：表赋值仅传递引用，修改副本会影响原表，需手动实现深拷贝/浅拷贝。
8. **无复合赋值运算符**：不支持`++`、`--`、`+=`、`-=`等，需手动写完整赋值语句。
9. **无switch-case**：原生不支持，推荐用表模拟实现。
10. **__index触发规则**：仅当键不存在时触发，键存在时直接返回值，不会走元方法。
11. **协程是协作式**：必须主动`yield`让出控制权，否则会一直占用执行权，无法切换。


