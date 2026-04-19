# Go 语法详细总结


# Go 语法详细总结
Go（Golang）是Google推出的**静态强类型、编译型、并发原生**的编程语言，核心设计理念是简洁、高效、易维护，天生支持高并发场景。本文基于Go 1.18+稳定版本，从基础到进阶全面梳理Go核心语法与特性。

## 一、程序基础结构与包管理
Go程序以**包（package）** 为组织单位，所有代码都必须归属某个包，程序入口是`main`包下的`main()`函数。

### 1. 包声明与导入
```go
// 1. 包声明：必须在文件第一行，同一个包下的所有文件包名一致
package main // main包表示可执行程序，非main包是库包

// 2. 包导入
import "fmt" // 单行导入

// 批量导入（推荐）
import (
    "errors"        // 标准库包
    "sync"
    "github.com/xxx/xxx" // 第三方包
    m "math"         // 别名导入，用m替代math
    _ "net/http/pprof" // 匿名导入，仅执行包的init函数，不使用包内标识符
    . "fmt"          // 点导入，可直接使用包内的Println等函数，不推荐滥用
)
```

### 2. 程序入口与init函数
- **入口函数**：`func main()`，无参数、无返回值，仅能在`main`包中定义，程序启动后自动执行。
- **init函数**：无参数、无返回值，包初始化时自动执行，**先于main函数**，每个文件可定义多个init函数，执行顺序：
  1. 按依赖顺序初始化导入的包
  2. 初始化包内的常量、变量
  3. 执行包内的init函数
  4. 执行main包的main函数

### 3. 标识符与可见性规则
- **命名规则**：标识符由字母、数字、下划线组成，首字符不能是数字；区分大小写。
- **关键字**：Go仅有25个关键字，极简设计，避免语法冗余
  ```
  break    default      func    interface  select
  case     defer        go      map        struct
  chan     else         goto    package    switch
  const    fallthrough  if      range      type
  continue for          import  return     var
  ```
- **核心可见性规则**：
  - 标识符**首字母大写**：导出标识符，包外可访问（相当于public）
  - 标识符**首字母小写**：包内私有，仅当前包可访问（相当于private）

### 4. 注释
- 单行注释：`// 注释内容`（最常用）
- 多行注释：`/* 注释内容 */`，不可嵌套
- 文档注释：以`//`开头，写在包、导出函数、类型、字段上方，用于`godoc`生成API文档，禁止空行分隔。

## 二、变量与常量
### 1. 变量声明
Go变量必须先声明后使用，声明后有默认零值，不允许声明未使用的变量（函数内）。

| 声明方式 | 语法示例 | 适用场景 | 注意事项 |
|----------|----------|----------|----------|
| 完整声明 | `var age int = 18` | 全局/局部变量，显式指定类型 | 类型与值必须匹配 |
| 类型推导 | `var age = 18` | 全局/局部变量，编译器自动推导类型 | 不可同时省略var和类型 |
| 短变量声明 | `age := 18` | **仅函数内可用** | 至少有一个新变量，允许同作用域已有变量赋值，会发生变量遮蔽 |
| 批量声明 | `var (name string = "go"; age int = 18)` | 多个变量集中声明 | 支持同类型/不同类型变量 |

- **零值规则**：声明未赋值的变量，编译器自动赋予对应类型的零值
  - 数值类型：`0`
  - 布尔类型：`false`
  - 字符串：`""`（空串）
  - 指针、切片、map、chan、接口：`nil`
- **变量作用域**：全局变量（包内）、函数内变量、块级变量（if/for内）；内层作用域变量会遮蔽外层同名变量。
- **平行赋值**：`a, b = b, a`，无需中间变量即可交换两个变量的值。

### 2. 常量
常量是编译期确定的值，声明后不可修改，仅支持布尔、数值、字符串类型。
```go
// 1. 基础声明
const Pi float64 = 3.1415926
const Name = "Go" // 类型推导

// 2. 批量声明
const (
    StatusOk = 200
    NotFound = 404
    ServerError = 500
)

// 3. iota枚举：iota从0开始，每行自动+1，是Go的枚举实现方案
const (
    Zero = iota // 0
    One         // 1
    Two         // 2
    _           // 跳过值3
    Four        // 4
)

// 位运算枚举示例
const (
    Read = 1 << iota // 1<<0 = 1
    Write             // 1<<1 = 2
    Execute           // 1<<2 = 4
)
```
- 无类型常量：未显式指定类型的常量，可隐式转换为匹配的类型，避免类型溢出问题。
- 注意：iota仅在const块内生效，每个新的const块iota重置为0。

## 三、基础数据类型
Go数据类型分为**值类型**和**引用类型**，核心分类如下：

### 1. 基础值类型
#### 布尔类型 `bool`
- 取值：仅`true`/`false`，零值`false`
- 关键特性：**不支持与数字的隐式/显式转换**，不支持0/1代表布尔值，与C/C++完全不同。
- 操作符：`!`（非）、`&&`（与）、`||`（或），支持短路求值。

#### 数值类型
| 类型分类 | 具体类型 | 说明 |
|----------|----------|------|
| 有符号整数 | `int8/int16/int32/int64` | 8/16/32/64位有符号整数，范围固定 |
| 无符号整数 | `uint8/uint16/uint32/uint64` | 8/16/32/64位无符号整数 |
| 平台相关整数 | `int/uint` | 32位系统为32位，64位系统为64位，业务开发优先使用int |
| 指针整数 | `uintptr` | 存储指针地址的无符号整数，用于底层编程，runtime不保证其生命周期 |
| 浮点型 | `float32/float64` | 单精度/双精度浮点数，零值0.0，优先使用float64 |
| 复数 | `complex64/complex128` | 实部+虚部，complex64对应float32精度，complex128对应float64 |
| 字符类型 | `byte/rune` | byte=uint8（存储ASCII字符）；rune=int32（存储Unicode码点，处理中文等UTF-8字符） |

- 注意：不同数值类型之间必须**显式强制转换**，无隐式类型转换，例如`int32`和`int`也不能直接运算。

#### 字符串类型 `string`
- 核心特性：**不可变**，一旦创建，无法修改字符串内的单个字符；UTF-8编码，原生支持中文等多语言；零值是空串`""`。
- 常用操作：
  ```go
  // 1. 声明
  s1 := "hello go"
  s2 := `换行
  原生字符串` // 反引号声明，保留换行和特殊字符，无转义

  // 2. 拼接
  s3 := s1 + " 你好"

  // 3. 切片：s[起始索引:结束索引]，左闭右开，截取字节（非字符）
  sub := s1[0:5] // "hello"

  // 4. 长度：len()获取字节长度，非字符个数
  len("中文") // 6，每个中文占3个字节

  // 5. 遍历：for range处理Unicode字符，for循环处理字节
  for i, c := range "中文go" {
      fmt.Printf("索引:%d, 字符:%c\n", i, c)
  }

  // 6. 类型转换：与[]byte/[]rune互转
  b := []byte(s1)
  s4 := string(b)
  ```

### 2. 复合类型
#### 数组 `[N]T`
- 定义：固定长度的同类型元素集合，长度是类型的一部分，`[3]int`和`[5]int`是完全不同的类型。
- 特性：值类型，赋值、传参都会拷贝整个数组，零值是每个元素为对应类型的零值。
- 基础用法：
  ```go
  // 1. 声明
  var arr1 [3]int // 长度3，元素零值0
  arr2 := [5]int{1,2,3,4,5} // 字面量初始化
  arr3 := [...]int{1,2,3} // 编译器自动推导长度
  arr4 := [3]int{1: 10, 2: 20} // 指定索引初始化

  // 2. 访问与赋值
  arr1[0] = 100
  fmt.Println(arr1[0])

  // 3. 遍历
  for i := 0; i < len(arr1); i++ {
      fmt.Println(arr1[i])
  }
  for i, v := range arr1 { // for range遍历，i索引，v元素副本
      fmt.Println(i, v)
  }
  ```

#### 切片 `[]T`
- 定义：动态长度的同类型元素序列，Go中最常用的集合类型，底层基于数组实现。
- 底层结构：包含3个字段，指向底层数组的指针`ptr`、切片长度`len`、切片容量`cap`（底层数组从ptr开始的最大长度）。
- 零值：`nil`，`len(nil切片) = 0`，`cap(nil切片) = 0`。
- 核心用法：
  ```go
  // 1. 声明与创建
  var s1 []int // nil切片
  s2 := []int{1,2,3} // 字面量创建，len=3, cap=3
  s3 := make([]int, 2, 5) // make创建，len=2(初始元素零值), cap=5，推荐方式
  s4 := arr[0:3] // 从数组/切片截取，左闭右开，cap=底层数组剩余长度

  // 2. 核心操作
  len(s3) // 获取长度
  cap(s3) // 获取容量
  s3 = append(s3, 4, 5) // 追加元素，容量不足时自动扩容
  copy(s4, s2) // 拷贝元素，拷贝长度为两个切片的len最小值

  // 3. 扩容规则
  // - len < 1024时，每次扩容为原cap的2倍
  // - len >= 1024时，每次扩容为原cap的1.25倍
  // - 追加元素后cap足够时，复用底层数组；不足时，分配新数组，拷贝元素
  ```
- 注意：切片截取会共享底层数组，修改切片元素会影响原数组/其他共享切片；append扩容后会脱离原底层数组。

#### 映射 `map[K]V`
- 定义：无序的键值对集合，哈希表实现，键`K`必须是可比较类型（不能是切片、map、函数）。
- 零值：`nil`，nil的map不能直接赋值，必须先make初始化。
- 核心用法：
  ```go
  // 1. 声明与创建
  var m1 map[string]int // nil map
  m2 := make(map[string]int, 10) // make创建，指定初始容量，推荐提前预估容量避免扩容
  m3 := map[string]int{"a": 1, "b": 2} // 字面量初始化

  // 2. 增删改查
  m2["a"] = 100 // 新增/修改
  v := m2["a"] // 查找，键不存在返回值类型的零值
  v, ok := m2["a"] // 安全查找，ok=true表示键存在，false表示不存在
  delete(m2, "a") // 删除键值对，键不存在也不会panic

  // 3. 遍历
  for k, v := range m3 {
      fmt.Println(k, v)
  }
  // 仅遍历键
  for k := range m3 {
      fmt.Println(k)
  }
  ```
- 注意：map是引用类型，传参时不会拷贝整个map；map并发读写不安全，并发场景需加锁或使用`sync.Map`。

#### 指针 `*T`
- 定义：指向变量内存地址的类型，`T`是指针指向的变量类型。
- 核心特性：**不支持指针运算**，仅支持取地址`&`和解引用`*`，是安全指针。
- 零值：`nil`。
- 基础用法：
  ```go
  a := 10
  p := &a // 取a的地址，赋值给指针p，p的类型是*int
  fmt.Println(*p) // 解引用，获取p指向的变量值，输出10
  *p = 20 // 修改p指向的变量值，a变为20

  // 结构体指针自动解引用
  type User struct { Name string }
  u := &User{Name: "go"}
  fmt.Println(u.Name) // 无需(*u).Name，编译器自动解引用
  ```
- 常用场景：函数传参时避免大对象拷贝，修改函数外的变量值。

#### 结构体 `struct`
- 定义：自定义的复合类型，由多个字段组合而成，是Go实现面向对象的核心载体，Go无类的概念，用结构体+方法替代。
- 特性：值类型，赋值、传参会拷贝；支持匿名字段、嵌套结构体、标签（tag）。
- 核心用法：
  ```go
  // 1. 定义结构体
  type User struct {
      ID   int    // 导出字段，包外可访问
      Name string // 导出字段
      age  int    // 私有字段，仅包内可访问
      Address      // 匿名字段，类型名作为字段名
  }

  type Address struct {
      Province string
      City     string
  }

  // 2. 结构体标签tag：用于序列化、ORM等场景，反引号包裹
  type Student struct {
      Name string `json:"name" db:"user_name"`
      Age  int    `json:"age" db:"user_age"`
  }

  // 3. 初始化
  u1 := User{ID: 1, Name: "go", age: 18, Address: Address{Province: "北京", City: "北京"}}
  u2 := User{2, "java", 20, Address{"上海", "上海"}} // 按字段顺序赋值，不推荐，字段顺序变化会出问题
  u3 := new(User) // 指针类型，零值结构体，所有字段为对应零值
  u3.Name = "python"

  // 4. 字段访问
  fmt.Println(u1.Name)
  fmt.Println(u1.City) // 匿名字段的字段提升，直接访问嵌套结构体的字段
  ```
- 面向对象特性：
  - 封装：通过大小写可见性实现
  - 继承：通过结构体嵌套实现组合，Go推荐**组合优于继承**
  - 多态：通过接口实现

## 四、流程控制
Go流程控制极简，仅支持`if`、`for`、`switch`、`select`，无`while`、`do-while`循环，无三元运算符。

### 1. if 条件语句
```go
// 1. 基础用法
age := 18
if age >= 18 {
    fmt.Println("成年")
} else if age >= 6 {
    fmt.Println("未成年")
} else {
    fmt.Println("幼儿")
}

// 2. 初始化语句：变量作用域仅限if-else块内
if score := 90; score >= 90 {
    fmt.Println("优秀")
} else if score >= 60 {
    fmt.Println("及格")
} else {
    fmt.Println("不及格")
}
```
- 强制规则：条件表达式无需括号；大括号`{}`必须存在，即使单行代码；不支持三元运算符。

### 2. for 循环语句
Go唯一的循环结构，可替代`while`、`do-while`。
```go
// 1. 经典三段式循环
for i := 0; i < 10; i++ {
    fmt.Println(i)
}

// 2. while式循环：替代while
i := 0
for i < 10 {
    fmt.Println(i)
    i++
}

// 3. 无限循环：替代while(true)
for {
    fmt.Println("无限循环")
    break // 跳出循环
}

// 4. for range遍历：支持数组、切片、字符串、map、chan
arr := [3]int{1,2,3}
for i, v := range arr {
    fmt.Println(i, v)
}
// 忽略索引
for _, v := range arr {
    fmt.Println(v)
}
```
- 跳转控制：
  - `break`：跳出当前循环
  - `continue`：跳过本次循环，进入下一次
  - 标签：支持跳出多层循环
    ```go
    outer:
    for i := 0; i < 3; i++ {
        for j := 0; j < 3; j++ {
            if j == 2 {
                break outer // 跳出外层循环
            }
        }
    }
    ```

### 3. switch 分支语句
Go的switch极其灵活，无需break，支持多值匹配、表达式分支、类型判断。
```go
// 1. 表达式switch：默认不fallthrough，无需break
score := 85
switch score {
case 90, 95, 100: // 多值匹配
    fmt.Println("满分/优秀")
case 80:
    fmt.Println("良好")
    fallthrough // 显式穿透，执行下一个case，不推荐滥用
case 60, 70:
    fmt.Println("及格")
default:
    fmt.Println("不及格")
}

// 2. 初始化语句
switch s := 90; s {
case s >= 90:
    fmt.Println("优秀")
case s >= 60:
    fmt.Println("及格")
}

// 3. 空switch：替代多if-else
age := 18
switch {
case age >= 18:
    fmt.Println("成年")
case age >= 6:
    fmt.Println("未成年")
default:
    fmt.Println("幼儿")
}

// 4. 类型switch：判断接口的动态类型
var x interface{} = 10
switch v := x.(type) {
case int:
    fmt.Println("int类型", v)
case string:
    fmt.Println("string类型", v)
default:
    fmt.Println("未知类型", v)
}
```

### 4. goto 跳转语句
仅支持函数内跳转，不能跳过变量声明，不推荐滥用，仅适用于错误处理、跳出多层嵌套等特殊场景。
```go
func test() {
    i := 0
loop:
    if i < 10 {
        fmt.Println(i)
        i++
        goto loop // 跳转到标签
    }
}
```

## 五、函数与方法
Go中函数是**一等公民**，支持多返回值、匿名函数、闭包、可变参数；方法是绑定了接收者的函数，是Go实现面向对象的核心。

### 1. 函数基础定义
```go
// 语法：func 函数名(参数列表) (返回值列表) { 函数体 }

// 1. 无参数无返回值
func Hello() {
    fmt.Println("hello go")
}

// 2. 有参数有返回值
func Add(a int, b int) int {
    return a + b
}
// 同类型参数简写
func Add2(a, b int) int {
    return a + b
}

// 3. 多返回值
func Divide(a, b int) (int, error) {
    if b == 0 {
        return 0, errors.New("除数不能为0")
    }
    return a / b, nil
}

// 4. 命名返回值：提前声明返回值变量，函数内可直接赋值，支持裸返回
func Sum(a, b int) (sum int) {
    sum = a + b
    return // 裸返回，等价于return sum，复杂函数不推荐，可读性差
}

// 5. 可变参数：...T，必须放在参数列表最后，函数内当作切片处理
func SumAll(nums ...int) int {
    sum := 0
    for _, v := range nums {
        sum += v
    }
    return sum
}
// 调用：SumAll(1,2,3)  SumAll(arr...) 切片解包
```

### 2. 函数核心特性
#### 一等公民特性
函数可赋值给变量、作为参数传递、作为返回值，支持函数类型定义。
```go
// 定义函数类型
type CalcFunc func(int, int) int

// 函数作为参数
func Calc(a, b int, f CalcFunc) int {
    return f(a, b)
}

// 函数作为返回值
func AddFunc() CalcFunc {
    return func(a, b int) int {
        return a + b
    }
}

// 调用
add := AddFunc()
fmt.Println(Calc(1,2, add))
```

#### 匿名函数与闭包
```go
// 1. 匿名函数赋值给变量
add := func(a, b int) int {
    return a + b
}
fmt.Println(add(1,2))

// 2. 立即执行函数
func(a, b int) {
    fmt.Println(a + b)
}(1, 2)

// 3. 闭包：匿名函数捕获外部作用域的变量，延长变量生命周期
func Counter() func() int {
    i := 0
    return func() int {
        i++
        return i
    }
}
// 调用：c1和c2各自维护独立的i变量
c1 := Counter()
fmt.Println(c1()) // 1
fmt.Println(c1()) // 2
c2 := Counter()
fmt.Println(c2()) // 1
```
- 常见坑：循环内闭包捕获循环变量，会导致所有闭包共享同一个变量，需在循环内重新赋值解决。

#### defer 延迟语句
- 核心特性：defer后的语句会延迟到**函数返回前**执行，多个defer按**逆序**执行（先进后出）。
- 常用场景：资源释放、锁释放、文件关闭、panic捕获。
```go
func ReadFile() {
    file, err := os.Open("test.txt")
    if err != nil {
        return
    }
    defer file.Close() // 函数返回前自动关闭文件，避免资源泄漏

    // 多个defer逆序执行
    defer fmt.Println("第一个defer")
    defer fmt.Println("第二个defer")
    // 执行顺序：第二个defer → 第一个defer
}
```
- 关键细节：defer的函数参数会**预计算**，在defer声明时就确定值，而非执行时。

#### panic与recover 异常机制
- `panic`：抛出致命错误，终止程序正常执行，触发当前函数内所有defer逆序执行。
- `recover`：捕获panic抛出的错误，恢复程序正常执行，**仅能在defer的匿名函数内调用**，无法跨goroutine捕获panic。
```go
func TestPanic() {
    defer func() {
        if err := recover(); err != nil {
            fmt.Println("捕获panic:", err)
        }
    }()

    fmt.Println("开始执行")
    panic("发生致命错误") // 抛出panic
    fmt.Println("不会执行")
}
```
- 最佳实践：panic仅用于不可恢复的致命错误，普通业务错误使用error接口处理，不要用panic-recover替代try-catch。

### 3. 方法
方法是绑定了**接收者（Receiver）** 的函数，接收者可以是值类型或指针类型，仅能给包内的自定义类型绑定方法，不能给原生类型（如int）直接绑定（需type定义别名）。
```go
// 定义自定义类型
type User struct {
    Name string
    Age  int
}

// 1. 值接收者方法：拷贝结构体，无法修改原对象的字段
func (u User) GetName() string {
    return u.Name
}

// 2. 指针接收者方法：传递结构体地址，可修改原对象的字段
func (u *User) SetName(name string) {
    u.Name = name
}

// 调用
u := User{Name: "go", Age: 18}
u.SetName("golang") // 编译器自动转为(&u).SetName("golang")
fmt.Println(u.GetName())
```
- 核心规则：
  1. 值接收者：方法内操作的是结构体的副本，修改不会影响原对象；适用于小结构体、只读场景。
  2. 指针接收者：方法内操作的是原结构体的地址，修改会影响原对象；适用于大结构体、需要修改原对象的场景。
  3. 一致性原则：同一个类型的方法，接收者类型要保持一致，不要混用值和指针接收者。
- 方法集规则：
  - 值类型的方法集：仅包含值接收者的方法。
  - 指针类型的方法集：包含值接收者 + 指针接收者的所有方法。
  - 接口实现的核心：指针接收者实现的接口，仅指针类型实现了该接口；值接收者实现的接口，值和指针类型都实现了该接口。

## 六、接口
Go的接口是**非侵入式**的鸭子类型设计，是实现多态、解耦的核心，无需显式声明`implements`，只要类型实现了接口的所有方法，就自动实现了该接口。

### 1. 接口定义与实现
```go
// 1. 定义接口：包含方法签名列表
type Animal interface {
    Speak() string
    Move() string
}

// 2. 定义类型，实现接口的所有方法，自动实现接口
type Dog struct{}

func (d Dog) Speak() string {
    return "汪汪汪"
}

func (d Dog) Move() string {
    return "狗跑"
}

type Cat struct{}

func (c Cat) Speak() string {
    return "喵喵喵"
}

func (c Cat) Move() string {
    return "猫走"
}

// 3. 多态使用：接口类型变量可接收所有实现了该接口的类型实例
func TestAnimal(a Animal) {
    fmt.Println(a.Speak())
    fmt.Println(a.Move())
}

// 调用
TestAnimal(Dog{})
TestAnimal(Cat{})
```

### 2. 空接口 `interface{}`
- 空接口没有任何方法签名，**所有类型都自动实现了空接口**，Go1.18+提供别名`any`，等价于`interface{}`。
- 常用场景：接收任意类型的参数、存储任意类型的值。
```go
// 别名any，推荐使用
func PrintAny(v any) {
    fmt.Println(v)
}

// 调用
PrintAny(10)
PrintAny("hello")
PrintAny([]int{1,2,3})

// map存储任意类型
m := map[string]any{
    "name": "go",
    "age": 18,
}
```

### 3. 类型断言与类型判断
接口类型变量需要获取底层的动态类型和值，需使用类型断言。
```go
var a Animal = Dog{}

// 1. 直接断言：失败会panic
d := a.(Dog)
fmt.Println(d)

// 2. 安全断言：ok=true断言成功，ok=false断言失败，不会panic
d, ok := a.(Dog)
if ok {
    fmt.Println("是Dog类型", d)
}

// 3. 类型switch：批量判断类型
switch v := a.(type) {
case Dog:
    fmt.Println("Dog类型", v)
case Cat:
    fmt.Println("Cat类型", v)
default:
    fmt.Println("未知类型", v)
}
```
- 常见坑：**nil接口 ≠ 接口值为nil**。接口变量由`类型type`和`值data`两部分组成，只有type和data都为nil时，接口变量才等于nil。
  ```go
  var i interface{} // type=nil, data=nil → i == nil
  var p *int = nil
  i = p // type=*int, data=nil → i != nil
  ```

### 4. 接口嵌套
接口可以嵌入其他接口，组合方法集，嵌入的接口方法会被合并到当前接口。
```go
type Reader interface {
    Read() []byte
}

type Writer interface {
    Write([]byte) int
}

// 嵌套接口，合并Reader和Writer的所有方法
type ReadWriter interface {
    Reader
    Writer
    Close() error
}
```
- 最佳实践：**接口最小化原则**，接口越小，灵活性越高，Go标准库中大量使用单方法接口（如`io.Reader`、`io.Writer`）。

## 七、泛型（Go1.18+）
泛型是Go1.18引入的核心特性，解决了代码复用、类型安全的问题，支持泛型函数、泛型类型，不支持泛型方法。

### 1. 泛型基础
泛型的核心是**类型参数**，通过方括号`[]`声明，指定类型约束。
```go
// 1. 泛型函数：支持任意数值类型的加法
func Add[T int | float64](sslocal://flow/file_open?url=a%2C+b+T&flow_extra=eyJsaW5rX3R5cGUiOiJjb2RlX2ludGVycHJldGVyIn0=) T {
    return a + b
}

// 调用
fmt.Println(Add(1, 2)) // int类型
fmt.Println(Add(1.1, 2.2)) // float64类型

// 2. 泛型类型：通用的切片类型
type Slice[T any] []T

// 给泛型类型绑定方法
func (s Slice[T]) Len() int {
    return len(s)
}

// 调用
s := Slice[int]{1,2,3}
fmt.Println(s.Len())
```

### 2. 类型约束
类型约束用于限制类型参数的范围，通过接口定义。
```go
// 1. 内置约束
// - any：所有类型，等价于interface{}
// - comparable：所有可比较类型（支持==/!=），map的键必须是comparable

// 2. 自定义类型集约束：限制类型范围
type Number interface {
    ~int | ~int64 | ~float32 | ~float64 // ~表示底层类型是该类型的所有类型
}

func Sum[T Number](sslocal://flow/file_open?url=nums+...T&flow_extra=eyJsaW5rX3R5cGUiOiJjb2RlX2ludGVycHJldGVyIn0=) T {
    var sum T
    for _, v := range nums {
        sum += v
    }
    return sum
}

// 3. 方法集约束：限制类型必须实现指定方法
type Stringer interface {
    String() string
}

func PrintString[T Stringer](sslocal://flow/file_open?url=v+T&flow_extra=eyJsaW5rX3R5cGUiOiJjb2RlX2ludGVycHJldGVyIn0=) {
    fmt.Println(v.String())
}
```

### 3. 泛型应用场景
- 通用工具函数：排序、过滤、映射、最大值/最小值等。
- 通用数据结构：栈、队列、链表、树等。
- 通用业务组件：ORM、序列化、缓存组件等。
- 限制：不能给普通类型的方法添加类型参数，仅支持泛型类型的方法。

## 八、并发编程（goroutine与channel）
Go的核心优势，基于**CSP（通信顺序进程）** 模型，核心设计理念：**不要通过共享内存来通信，而要通过通信来共享内存**。

### 1. goroutine 轻量级协程
goroutine是Go runtime调度的用户态轻量级线程，相比内核线程，资源占用极低，初始栈仅2KB，可动态扩容缩容，单机可轻松启动百万级goroutine。
```go
// 启动goroutine：go关键字 + 函数调用
func Hello() {
    fmt.Println("hello goroutine")
}

func main() {
    go Hello() // 启动一个goroutine
    go func() { // 匿名函数启动goroutine
        fmt.Println("匿名goroutine")
    }()

    time.Sleep(time.Second) // 等待goroutine执行完成，主goroutine退出，所有子goroutine都会退出
    fmt.Println("main函数结束")
}
```
- GMP调度模型：Go runtime的调度核心，分为三个组件：
  - G：goroutine，即用户启动的协程。
  - M：内核线程，真正执行代码的载体。
  - P：逻辑处理器，管理G队列，持有运行资源，M必须绑定P才能执行G。

### 2. channel 通道
channel是goroutine之间通信的载体，用于在goroutine之间传递数据，实现同步与通信。
```go
// 1. 声明与创建
var ch chan int // 声明通道，零值nil
ch = make(chan int) // 无缓冲通道
ch = make(chan int, 5) // 有缓冲通道，缓冲大小5

// 2. 核心操作
ch <- 10 // 向通道写入数据
v := <-ch // 从通道读取数据
<-ch // 读取数据并丢弃
close(ch) // 关闭通道，关闭后可读不可写，重复关闭、向已关闭的通道写入会panic

// 3. 无缓冲通道：读写必须配对，否则阻塞，用于goroutine同步
func NoBufferChan() {
    ch := make(chan int)
    go func() {
        ch <- 10 // 阻塞，直到主goroutine读取
        fmt.Println("写入完成")
    }()
    v := <-ch // 阻塞，直到goroutine写入
    fmt.Println("读取到", v)
}

// 4. 有缓冲通道：缓冲未满，写入不阻塞；缓冲未空，读取不阻塞
func BufferChan() {
    ch := make(chan int, 3)
    ch <- 1
    ch <- 2
    ch <- 3
    // ch <-4 缓冲满，阻塞
    fmt.Println(<-ch)
    fmt.Println(<-ch)
}

// 5. 通道遍历：for range，通道必须关闭才会退出循环
func RangeChan() {
    ch := make(chan int, 3)
    ch <- 1
    ch <- 2
    ch <- 3
    close(ch) // 关闭通道

    for v := range ch {
        fmt.Println(v)
    }
}

// 6. 单向通道：限制通道的读写权限，用于函数参数，避免滥用
// chan<- T：只写通道
// <-chan T：只读通道
func WriteChan(ch chan<- int) {
    ch <- 10
}
func ReadChan(ch <-chan int) int {
    return <-ch
}
```

### 3. select 多路通道监听
select专门用于多路通道的读写操作监听，随机选择一个就绪的case执行，无就绪case时阻塞，支持default实现非阻塞操作。
```go
func SelectDemo() {
    ch1 := make(chan int)
    ch2 := make(chan int)

    go func() {
        time.Sleep(time.Second)
        ch1 <- 10
    }()
    go func() {
        time.Sleep(time.Second)
        ch2 <- 20
    }()

    // 多路监听，随机选择就绪的case
    select {
    case v := <-ch1:
        fmt.Println("从ch1读取到", v)
    case v := <-ch2:
        fmt.Println("从ch2读取到", v)
    case <-time.After(2 * time.Second): // 超时控制
        fmt.Println("超时")
    // default: // 非阻塞，无就绪case直接执行default
    //     fmt.Println("无就绪通道")
    }
}
```
- 常用场景：超时控制、多路通道复用、goroutine退出信号监听、非阻塞读写。

### 4. 同步原语
Go`sync`包提供了基础的同步原语，用于共享内存场景的并发控制。
| 同步原语 | 作用 | 核心方法 |
|----------|------|----------|
| `sync.WaitGroup` | 等待一组goroutine执行完成 | `Add()`：添加计数；`Done()`：计数-1；`Wait()`：阻塞直到计数为0 |
| `sync.Mutex` | 互斥锁，保证同一时间只有一个goroutine访问临界区 | `Lock()`：加锁；`Unlock()`：解锁 |
| `sync.RWMutex` | 读写锁，读共享、写独占，读多写少场景性能优于互斥锁 | `RLock()`/`RUnlock()`：读加锁/解锁；`Lock()`/`Unlock()`：写加锁/解锁 |
| `sync.Once` | 保证代码只执行一次，常用于单例、初始化场景 | `Do(func())`：传入的函数仅执行一次 |
| `sync.Cond` | 条件变量，用于goroutine之间的通知与等待 | `Wait()`：等待；`Signal()`：单个通知；`Broadcast()`：广播通知 |
| `sync.Pool` | 对象池，复用临时对象，减少GC压力，零值可用 | `Get()`：获取对象；`Put()`：归还对象 |

- `context`上下文：用于goroutine的取消、超时、截止时间、元数据传递，是Go并发编程的核心工具。
  ```go
  func ContextDemo() {
      // 根上下文
      ctx, cancel := context.WithCancel(context.Background())
      // 超时上下文：ctx, cancel := context.WithTimeout(context.Background(), 2*time.Second)
      defer cancel() // 函数退出时取消，避免goroutine泄漏

      go func(ctx context.Context) {
          for {
              select {
              case <-ctx.Done(): // 收到取消信号
                  fmt.Println("goroutine退出")
                  return
              default:
                  fmt.Println("运行中")
                  time.Sleep(500 * time.Millisecond)
              }
          }
      }(ctx)

      time.Sleep(2 * time.Second)
      cancel() // 发送取消信号
      time.Sleep(time.Second)
  }
  ```

## 九、错误处理
Go的错误处理哲学是**显式处理**，不支持try-catch异常机制，强制开发者直面错误，代码更健壮。

### 1. error 接口
Go通过`error`接口表示错误，是Go错误处理的核心。
```go
// error接口定义
type error interface {
    Error() string
}

// 1. 创建简单错误
err1 := errors.New("用户名不存在")

// 2. 格式化错误，%w包装错误（Go1.13+）
err2 := fmt.Errorf("查询用户失败: %w", err1)

// 3. 自定义错误类型，携带更多上下文
type BizError struct {
    Code int
    Msg  string
}

// 实现error接口
func (e *BizError) Error() string {
    return fmt.Sprintf("code:%d, msg:%s", e.Code, e.Msg)
}

// 使用自定义错误
func QueryUser(id int) (*User, error) {
    if id <= 0 {
        return nil, &BizError{Code: 400, Msg: "用户ID非法"}
    }
    return &User{ID: id}, nil
}
```

### 2. 错误处理规范
```go
// 标准错误处理模式：函数多返回值，最后一个返回error
func Divide(a, b int) (int, error) {
    if b == 0 {
        return 0, errors.New("除数不能为0")
    }
    return a / b, nil
}

// 调用：显式判断error
res, err := Divide(10, 0)
if err != nil {
    // 错误处理，优先处理错误，减少嵌套
    fmt.Println("计算失败:", err)
    return
}
fmt.Println("计算结果:", res)
```

### 3. 错误包装与解包（Go1.13+）
- `errors.Is(err, target)`：判断err链中是否包含target错误，替代直接`==`判断。
- `errors.As(err, &target)`：将err链中的错误转换为目标自定义错误类型，替代类型断言。
- `errors.Unwrap(err)`：解包被%w包装的错误。
```go
err1 := errors.New("底层错误")
err2 := fmt.Errorf("上层错误: %w", err1)

// 判断是否包含底层错误
fmt.Println(errors.Is(err2, err1)) // true

// 转换为自定义错误
var bizErr *BizError
if errors.As(err, &bizErr) {
    fmt.Println(bizErr.Code, bizErr.Msg)
}
```

### 4. 最佳实践
1. 不要忽略错误，即使要忽略，也要显式用`_`标注。
2. 错误信息要包含完整上下文，不要大写开头，不要换行。
3. 优先使用错误包装，保留原始错误信息。
4. 自定义错误仅用于需要携带额外信息的业务场景，简单错误用`errors.New`。
5. panic仅用于不可恢复的致命错误，不要用于普通业务错误处理。

## 十、Go模块与包管理（go mod）
Go1.11+正式引入`go mod`，是Go官方的标准包管理工具，替代了旧的GOPATH模式，解决了依赖版本管理问题。

### 1. 核心命令
| 命令 | 作用 |
|------|------|
| `go mod init 模块名` | 初始化项目，生成go.mod文件，模块名通常是仓库地址 |
| `go mod tidy` | 整理依赖，自动添加缺失的依赖，删除未使用的依赖，更新go.sum |
| `go get 依赖包@版本` | 添加/更新依赖，@latest表示最新版本 |
| `go mod vendor` | 生成vendor目录，将所有依赖拷贝到项目中，离线编译 |
| `go mod download` | 下载go.mod中声明的所有依赖到本地缓存 |
| `go list -m all` | 列出当前项目所有依赖 |

### 2. go.mod 文件结构
```go
module github.com/xxx/xxx // 模块名，全局唯一

go 1.22 // Go版本

require ( // 依赖声明
    github.com/gin-gonic/gin v1.9.1
    github.com/go-redis/redis/v8 v8.11.5
)

replace github.com/xxx/xxx => ./local/xxx // 替换依赖，本地调试常用
exclude github.com/xxx/xxx v1.0.0 // 排除指定版本的依赖
```

### 3. 核心规则
- 一个项目一个go.mod文件，放在项目根目录。
- 包的可见性由标识符首字母大小写决定。
- 禁止循环导入包，编译器会直接报错。
- 依赖包默认下载到GOPATH/pkg/mod目录，全局缓存。

## 十一、常见语法坑与最佳实践
### 1. 高频语法坑
1. **for range遍历副本问题**：遍历的value是元素的副本，修改value不会影响原集合的元素，需通过索引修改。
2. **循环内闭包捕获循环变量**：所有闭包共享同一个循环变量，循环结束后变量值固定，需在循环内重新赋值解决。
3. **nil接口与接口值为nil的区别**：接口变量的type不为nil时，即使data为nil，接口变量也不等于nil。
4. **defer参数预计算**：defer的函数参数在声明时就计算完成，而非执行时。
5. **map零值赋值**：nil的map不能直接赋值，必须先make初始化。
6. **goroutine泄漏**：通道未关闭、select无退出分支、阻塞的goroutine会导致泄漏，需用context控制生命周期。
7. **切片共享底层数组**：切片截取后共享底层数组，修改元素会影响原数组，append扩容后会脱离原数组。

### 2. 核心最佳实践
1. 代码格式化强制使用`gofmt`/`goimports`，保持代码风格统一。
2. 错误显式处理，不忽略、不滥用panic-recover。
3. 接口最小化，单方法接口优先，提升代码灵活性。
4. 优先使用值接收者，需要修改原对象、大结构体使用指针接收者，保持接收者类型一致。
5. 并发编程优先使用channel通信，共享内存场景用锁保护，避免数据竞争。
6. 避免全局变量，使用依赖注入，降低代码耦合。
7. 提前预估map、切片的容量，避免运行时频繁扩容，提升性能。
8. 单元测试文件以`_test.go`结尾，使用`testing`包，覆盖核心逻辑。


