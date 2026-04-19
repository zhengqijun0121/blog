# Rust 语法详细总结


# Rust 语法详细总结
Rust 是一门注重**内存安全、高性能、并发安全**的系统编程语言，无GC、无运行时开销，核心设计围绕**所有权系统**展开。本文从基础语法到核心特性、进阶用法，全面梳理Rust标准语法（基于Rust稳定版标准规范）。

## 一、基础语法入门
### 1.1 变量与可变性
Rust 变量**默认不可变**，这是核心安全设计之一，通过关键字控制可变性与生命周期。
| 语法 | 说明 | 示例 |
| :--- | :--- | :--- |
| `let` | 声明不可变变量，类型可自动推导，也可手动标注 | `let a = 10; let b: i32 = 20;` |
| `let mut` | 声明可变变量，允许修改值，类型固定 | `let mut count = 0; count += 1;` |
| `const` | 声明编译期常量，必须标注类型，全局有效，不可变 | `const MAX_NUM: u32 = 10000;` |
| `static` | 声明静态变量，有固定内存地址，生命周期为整个程序，可声明可变（需unsafe访问） | `static GLOBAL: &str = "hello"; static mut COUNT: i32 = 0;` |

- **变量遮蔽（Shadowing）**：同名`let`声明可覆盖之前的变量，支持修改类型，与`mut`不同
  ```rust
  let x = 5;
  let x = x + 1; // 遮蔽，不可变x被新的不可变x覆盖
  let x = "hello"; // 支持类型变更
  ```
- **const与static的核心区别**：const无固定内存地址，编译期内联展开；static有唯一内存地址，可跨线程访问。

### 1.2 数据类型
Rust是强静态类型语言，编译期必须确定所有变量类型，分为**标量类型**和**复合类型**两大类。

#### 1.2.1 标量类型（单值类型）
| 类型分类 | 具体类型 | 说明 |
| :--- | :--- | :--- |
| 整数类型 | 有符号：`i8/i16/i32/i64/i128/isize` <br> 无符号：`u8/u16/u32/u64/u128/usize` | `isize/usize` 长度由平台决定（32位平台32位，64位平台64位）；<br>默认整数类型为`i32`；<br>字面量支持进制：`0b`二进制、`0o`八进制、`0x`十六进制，支持`_`分隔符（`1_000_000`） |
| 浮点类型 | `f32`/`f64` | 遵循IEEE 754标准，默认类型为`f64`，精度更高 |
| 布尔类型 | `bool` | 仅`true`/`false`两个值，占用1字节 |
| 字符类型 | `char` | Unicode标量值，占用4字节，单引号包裹，支持中文、emoji等所有Unicode字符 |

#### 1.2.2 复合类型（多值组合）
1. **元组（Tuple）**：固定长度、元素类型可不同，栈上分配
   ```rust
   let tup: (i32, f64, &str) = (10, 3.14, "rust");
   // 解构
   let (x, y, z) = tup;
   // 索引访问
   let first = tup.0;
   ```
   空元组`()`称为unit类型，代表无返回值，是函数默认返回类型。

2. **数组（Array）**：固定长度、元素类型必须相同，栈上分配
   ```rust
   let arr: [i32; 5] = [1,2,3,4,5]; // [类型; 长度]
   let arr2 = [0; 3]; // 等价于 [0,0,0]
   let first = arr[0]; // 索引访问
   ```

3. **切片（Slice）**：动态长度的连续序列引用，无所有权，格式为`&[T]`（不可变）/`&mut [T]`（可变）
   ```rust
   let arr = [1,2,3,4,5];
   let slice: &[i32] = &arr[1..3]; // 引用arr的第2、3个元素，长度2
   ```
   字符串切片`&str`是最常用的切片类型，是UTF-8编码的字节切片。

### 1.3 函数
Rust函数以`fn`关键字声明，参数必须标注类型，返回值通过`->`标注，无返回值默认返回`()`。
```rust
// 标准函数定义
fn add(a: i32, b: i32) -> i32 {
    a + b // 表达式结尾无分号，作为返回值；加;则返回()，编译报错
}

// 无返回值函数
fn print_hello() {
    println!("hello");
}

// 发散函数：!表示永远不会返回
fn never_return() -> ! {
    panic!("程序终止");
}
```
- **核心规则**：Rust是**基于表达式**的语言，语句执行操作无返回值，表达式求值并返回值。函数体最后一个表达式即为返回值，无需`return`；`return`仅用于提前返回。
- **关联函数**：定义在`impl`块中，无`self`参数，通过`类型名::函数名`调用，常用于构造函数（如`new()`）。
- **方法**：定义在`impl`块中，第一个参数为`&self`/`&mut self`/`self`，代表实例本身，通过`实例.方法名()`调用。

### 1.4 控制流
#### 1.4.1 分支结构
1. **if表达式**：可作为值赋值，条件必须是`bool`类型，无需括号包裹
   ```rust
   let number = 10;
   // 基础用法
   if number > 5 {
       println!("大于5");
   } else if number < 0 {
       println!("小于0");
   } else {
       println!("0-5之间");
   }

   // 作为表达式赋值
   let res = if number > 5 { "大" } else { "小" };
   ```

2. **if let**：简化单分支模式匹配，常用于枚举值处理
   ```rust
   let opt = Some(10);
   if let Some(v) = opt {
       println!("值为：{}", v);
   } else {
       println!("空值");
   }
   ```

#### 1.4.2 循环结构
1. **loop**：无限循环，可通过`break`带返回值退出
   ```rust
   let mut count = 0;
   let res = loop {
       count += 1;
       if count == 5 {
           break count * 2; // 退出循环并返回10
       }
   };
   ```

2. **while**：条件循环，条件为`true`时执行
   ```rust
   let mut n = 3;
   while n > 0 {
       println!("{}", n);
       n -= 1;
   }
   ```

3. **while let**：匹配模式成功时循环
   ```rust
   let mut stack = vec![1,2,3];
   while let Some(top) = stack.pop() {
       println!("{}", top);
   }
   ```

4. **for in**：迭代器循环，Rust最常用的循环方式，安全高效
   ```rust
   // 遍历范围
   for i in 1..5 { // 1-4，左闭右开
       println!("{}", i);
   }
   for i in 1..=5 { // 1-5，闭区间
       println!("{}", i);
   }

   // 遍历集合
   let arr = [10,20,30];
   for v in arr.iter() {
       println!("{}", v);
   }
   ```

## 二、Rust核心特性：所有权系统
所有权是Rust最核心的设计，无需GC即可保证内存安全，彻底避免空指针、野指针、二次释放等内存问题。

### 2.1 所有权核心规则
1.  Rust中每个值**有且仅有一个所有者**变量；
2.  当所有者离开作用域时，对应的值会被自动释放；
3.  禁止同一值被多个所有者同时持有，避免二次释放。

```rust
{
    let s = String::from("hello"); // s进入作用域，成为值的所有者
    // 操作s
} // s离开作用域，值被自动释放，内存回收
```

### 2.2 移动与复制语义
#### 2.2.1 移动（Move）
对于堆上分配的类型（如`String`、`Vec<T>`），赋值、传参、返回时默认执行**移动语义**：所有权转移给新变量，原变量立即失效，编译期禁止使用，避免二次释放。
```rust
let s1 = String::from("hello");
let s2 = s1; // s1的所有权转移给s2，s1失效
// println!("{}", s1); // 编译报错：s1已失去所有权
```

#### 2.2.2 复制（Copy）
对于栈上分配、大小固定的类型，赋值时执行**复制语义**：在栈上创建一份新的副本，原变量依然有效，所有权不转移。
- 实现了`Copy` trait的类型才支持复制，包括：所有标量类型、元素全为Copy类型的元组/数组。
- 实现了`Drop` trait的类型（堆上分配），无法实现`Copy` trait。
```rust
let x = 10;
let y = x; // i32实现了Copy，x复制给y，x依然有效
println!("{} {}", x, y); // 编译正常
```

#### 2.2.3 克隆（Clone）
如需深拷贝堆上的数据（同时复制栈和堆的内容），调用`clone()`方法，手动执行克隆。
```rust
let s1 = String::from("hello");
let s2 = s1.clone(); // 深拷贝，s1和s2各自持有独立的堆数据，均有效
println!("{} {}", s1, s2); // 编译正常
```

### 2.3 借用与引用
引用（`&T`/`&mut T`）允许使用值但**不获取所有权**，称为“借用”，是Rust中最常用的操作，核心规则保证内存安全。

#### 2.3.1 引用类型
| 类型 | 说明 | 示例 |
| :--- | :--- | :--- |
| 不可变引用 `&T` | 只读借用，可多次借用，无法修改值 | `let s = String::from("a"); let r = &s;` |
| 可变引用 `&mut T` | 可写借用，可修改值，有严格的排他性 | `let mut s = String::from("a"); let r = &mut s; r.push('b');` |

#### 2.3.2 借用核心规则（编译期强制检查）
1.  **同一时间，要么只能有一个可变引用，要么只能有多个不可变引用**，二者不可同时存在；
2.  **引用必须始终有效**，禁止悬垂引用（引用的值已被释放，引用依然存在）；
3.  引用的生命周期不能超过所有者的生命周期。

```rust
// 规则1示例：禁止可变引用与不可变引用共存
let mut s = String::from("hello");
let r1 = &s; // 不可变引用
let r2 = &s; // 多个不可变引用允许
// let r3 = &mut s; // 编译报错：不可变引用存在时，无法创建可变引用
println!("{} {}", r1, r2); // r1/r2作用域结束，后续可创建可变引用

// 规则2示例：禁止悬垂引用
// fn dangle() -> &String { // 编译报错：返回悬垂引用
//     let s = String::from("hello");
//     &s // s离开作用域被释放，引用无效
// }
```

#### 2.3.3 解引用
通过`*`运算符解引用，获取引用指向的值；Rust支持**自动解引用**（Deref强制转换），调用方法时无需手动写`*`。
```rust
let x = 10;
let r = &x;
println!("{}", *r); // 解引用，输出10
```

### 2.4 生命周期
生命周期是引用的**有效作用范围**，编译器通过生命周期标注，确保引用在整个有效期内都是合法的，彻底避免悬垂引用。生命周期标注不会改变引用的实际存活时间，仅用于编译器的约束检查。

#### 2.4.1 生命周期标注语法
生命周期标注以`'`开头，后跟标识符，惯例使用小写字母（如`'a`、`'b`）。
```rust
// 函数中的生命周期标注：标注输入与输出引用的生命周期关联
// 含义：返回值的生命周期与两个输入参数中生命周期最短的一致
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() { x } else { y }
}

// 结构体中的生命周期标注：标注结构体字段引用的生命周期
// 含义：结构体实例的生命周期不能超过其字段引用的生命周期
struct User<'a> {
    name: &'a str,
}
```

#### 2.4.2 生命周期省略规则
编译器内置3条省略规则，符合规则时无需手动标注生命周期，编译器自动推导：
1.  每个输入引用参数，都会被分配一个独立的生命周期参数；
2.  如果只有一个输入生命周期参数，该生命周期会被赋予所有输出生命周期参数；
3.  如果方法有多个输入生命周期参数，且其中一个是`&self`/`&mut self`，则`self`的生命周期会被赋予所有输出生命周期参数。

```rust
// 符合省略规则，无需手动标注
fn first_word(s: &str) -> &str { // 自动推导为 fn first_word<'a>(s: &'a str) -> &'a str
    s.split_whitespace().next().unwrap_or("")
}
```

#### 2.4.3 特殊生命周期
- **`'static`**：静态生命周期，引用在整个程序运行期间都有效，所有字符串字面量默认都是`'static`生命周期。
  ```rust
  let s: &'static str = "hello rust"; // 字符串字面量，存储在二进制文件中，全程有效
  ```
- **生命周期约束**：通过`'a: 'b`表示`'a`的生命周期比`'b`长，`'a`包含`'b`。

### 2.5 智能指针
智能指针是管理堆内存的类型，实现了`Deref`（解引用）和`Drop`（析构）trait，在所有权规则基础上提供额外功能，常用类型如下：

| 智能指针 | 核心功能 | 适用场景 | 线程安全 |
| :--- | :--- | :--- | :--- |
| `Box<T>` | 堆上分配值，单所有权，编译期检查借用规则 | 递归类型（解决大小不确定问题）、大体积数据转移所有权 | 是 |
| `Rc<T>` | 引用计数，单线程内多所有权，不可变引用 | 单线程内多个地方需要只读访问同一个值 | 否 |
| `Arc<T>` | 原子引用计数，多线程多所有权，不可变引用 | 多线程内多个地方需要只读访问同一个值 | 是 |
| `RefCell<T>` | 内部可变性，单线程，运行时检查借用规则 | 编译期无法满足借用规则，需要在不可变值内部修改数据 | 否 |
| `Mutex<T>` | 互斥锁，多线程内部可变性，运行时检查借用规则 | 多线程环境下修改共享数据 | 是 |
| `Weak<T>` | 弱引用，不增加引用计数，避免循环引用 | 解决Rc/Arc的循环引用内存泄漏问题 | 对应Rc/Arc的线程安全 |

```rust
// Box<T> 示例：解决递归类型大小不确定问题
enum List {
    Cons(i32, Box<List>),
    Nil,
}

// Rc<T> 示例：多所有权
use std::rc::Rc;
let a = Rc::new(String::from("hello"));
let b = Rc::clone(&a); // 引用计数+1，不深拷贝
let c = Rc::clone(&a); // 引用计数+1
println!("引用计数：{}", Rc::strong_count(&a)); // 输出3
```

## 三、类型系统与面向对象特性
### 3.1 结构体（struct）
结构体用于自定义复合类型，将多个相关字段组合在一起，分为3种类型：

#### 3.1.1 结构体类型
1.  **具名字段结构体**：最常用，每个字段有名称和类型
    ```rust
    #[derive(Debug)] // 派生Debug trait，支持打印
    struct User {
        name: String,
        age: u8,
        active: bool,
    }

    // 实例化
    let mut user = User {
        name: String::from("张三"),
        age: 20,
        active: true,
    };
    // 访问与修改
    user.age = 21;
    // 结构体更新语法
    let user2 = User {
        name: String::from("李四"),
        ..user // 剩余字段复用user的值
    };
    ```

2.  **元组结构体**：字段无名称，仅标注类型，适合简单包装类型
    ```rust
    struct Point(i32, i32);
    struct Color(u8, u8, u8);
    let p = Point(10, 20);
    println!("x: {}", p.0);
    ```

3.  **单元结构体**：无任何字段，常用于实现trait，不占用内存
    ```rust
    struct Unit;
    let u = Unit;
    ```

#### 3.1.2 结构体方法与关联函数
通过`impl`块为结构体定义方法和关联函数：
```rust
impl User {
    // 关联函数：无self参数，常用于构造函数
    fn new(name: String, age: u8) -> Self {
        User {
            name,
            age,
            active: true,
        }
    }

    // 方法：&self 不可变借用实例
    fn get_name(&self) -> &str {
        &self.name
    }

    // 方法：&mut self 可变借用实例，可修改字段
    fn set_age(&mut self, new_age: u8) {
        self.age = new_age;
    }
}

// 调用
let mut user = User::new(String::from("张三"), 20);
user.set_age(21);
println!("姓名：{}", user.get_name());
```

### 3.2 枚举（enum）
Rust的枚举是**代数数据类型（ADT）**，每个枚举变体可以携带不同类型和数量的数据，功能远超其他语言的枚举。

#### 3.2.1 枚举定义与使用
```rust
// 定义枚举，变体可携带不同数据
enum Message {
    Quit, // 无数据
    Move { x: i32, y: i32 }, // 具名字段，类结构体
    Write(String), // 携带单个String
    ChangeColor(u8, u8, u8), // 携带多个值，类元组
}

// 实例化
let msg1 = Message::Quit;
let msg2 = Message::Move { x: 10, y: 20 };
let msg3 = Message::Write(String::from("hello"));
let msg4 = Message::ChangeColor(255, 0, 0);
```

#### 3.2.2 核心内置枚举
Rust标准库内置两个核心枚举，是Rust无空指针、无异常的核心支撑：
1.  **`Option<T>`**：替代空值，避免空指针异常
    ```rust
    // 标准库定义
    enum Option<T> {
        Some(T), // 携带有效值
        None, // 空值
    }

    // 使用
    let some_num = Some(10);
    let none_num: Option<i32> = None;
    ```

2.  **`Result<T, E>`**：用于可恢复错误处理，替代异常
    ```rust
    // 标准库定义
    enum Result<T, E> {
        Ok(T), // 成功，携带返回值
        Err(E), // 失败，携带错误信息
    }

    // 使用
    let ok_res: Result<i32, String> = Ok(200);
    let err_res: Result<i32, String> = Err("请求失败".to_string());
    ```

#### 3.2.3 枚举方法
和结构体一样，可通过`impl`块为枚举定义方法：
```rust
impl Message {
    fn call(&self) {
        match self {
            Message::Quit => println!("退出"),
            Message::Move { x, y } => println!("移动到({}, {})", x, y),
            Message::Write(s) => println!("写入：{}", s),
            Message::ChangeColor(r, g, b) => println!("颜色：({}, {}, {})", r, g, b),
        }
    }
}

msg3.call(); // 输出：写入：hello
```

### 3.3 泛型
泛型是类型参数化，实现代码复用，无运行时开销（编译期单态化，生成具体类型的代码），可用于函数、结构体、枚举、方法中。

#### 3.3.1 泛型基础用法
```rust
// 泛型函数
fn max<T: PartialOrd>(a: T, b: T) -> T {
    if a > b { a } else { b }
}

// 泛型结构体
struct Point<T> {
    x: T,
    y: T,
}
// 多泛型参数
struct Point2<T, U> {
    x: T,
    y: U,
}

// 泛型枚举
enum MyOption<T> {
    Some(T),
    None,
}

// 为泛型结构体实现方法
impl<T> Point<T> {
    fn x(&self) -> &T {
        &self.x
    }
}
// 为具体类型实现专属方法
impl Point<f64> {
    fn distance_from_origin(&self) -> f64 {
        (self.x.powi(2) + self.y.powi(2)).sqrt()
    }
}
```

#### 3.3.2 Trait约束（Trait Bound）
泛型约束用于限制泛型类型必须实现指定的trait，确保泛型类型具备所需的行为，有3种写法：
1.  **基础约束**：函数参数中直接标注
    ```rust
    // T必须实现PartialOrd和Display两个trait
    fn print_max<T: PartialOrd + std::fmt::Display>(a: T, b: T) {
        println!("最大值：{}", if a > b { a } else { b });
    }
    ```

2.  **impl Trait语法**：简化单约束场景，适合参数和返回值
    ```rust
    // 参数必须实现Display trait
    fn print_val(val: impl std::fmt::Display) {
        println!("{}", val);
    }
    // 返回值必须实现Iterator trait，Item为i32
    fn get_iter() -> impl Iterator<Item = i32> {
        vec![1,2,3].into_iter()
    }
    ```

3.  **where子句**：简化多约束、多泛型场景，可读性更强
    ```rust
    fn some_function<T, U>(t: T, u: U)
    where
        T: std::fmt::Display + Clone,
        U: std::fmt::Debug + Clone,
    {
        // 函数体
    }
    ```

### 3.4 Trait
Trait是Rust的**接口抽象**，用于定义类型的共享行为，实现代码复用、多态、接口隔离，是Rust面向对象的核心。

#### 3.4.1 Trait定义与实现
```rust
// 定义Trait：声明行为签名，可提供默认实现
pub trait Summary {
    // 必须实现的方法签名
    fn summarize(&self) -> String;

    // 默认实现：类型可选择重写，不重写则使用默认
    fn default_summarize(&self) -> String {
        format!("(阅读更多：{})", self.summarize())
    }
}

// 为自定义类型实现Trait
pub struct Article {
    pub title: String,
    pub author: String,
    pub content: String,
}

impl Summary for Article {
    // 实现必须的方法
    fn summarize(&self) -> String {
        format!("《{}》- 作者：{}", self.title, self.author)
    }

    // 可选重写默认方法
    fn default_summarize(&self) -> String {
        format!("文章摘要：{}", self.summarize())
    }
}
```

#### 3.4.2 Trait高级用法
1.  **Trait对象与动态分发**：`dyn Trait`用于实现运行时多态，必须通过指针（`&dyn Trait`/`Box<dyn Trait>`）使用
    ```rust
    // 函数接收Trait对象，支持所有实现了Summary的类型，运行时动态分发
    fn print_summary(item: &dyn Summary) {
        println!("{}", item.summarize());
    }

    let article = Article {
        title: String::from("Rust入门"),
        author: String::from("张三"),
        content: String::from("内容"),
    };
    print_summary(&article); // 传入实现了Summary的类型实例
    ```

2.  **Trait继承**：一个Trait依赖另一个Trait的实现
    ```rust
    // SummaryPrint 继承 Summary 的所有行为
    pub trait SummaryPrint: Summary {
        fn print(&self) {
            println!("{}", self.summarize());
        }
    }
    ```

3.  **关联类型**：在Trait中定义占位类型，实现时指定具体类型，比泛型更简洁
    ```rust
    pub trait Iterator {
        type Item; // 关联类型，代表迭代器产出的值类型
        fn next(&mut self) -> Option<Self::Item>;
    }

    // 实现时指定关联类型
    impl Iterator for Counter {
        type Item = u32; // 固定类型，无需每次使用都标注泛型
        fn next(&mut self) -> Option<Self::Item> {
            // 实现逻辑
        }
    }
    ```

4.  **运算符重载**：通过实现标准库`std::ops`中的Trait，重载运算符
    ```rust
    use std::ops::Add;
    #[derive(Debug, PartialEq)]
    struct Point {
        x: i32,
        y: i32,
    }

    // 实现Add trait，重载+运算符
    impl Add for Point {
        type Output = Point;
        fn add(self, other: Point) -> Point {
            Point {
                x: self.x + other.x,
                y: self.y + other.y,
            }
        }
    }

    let p1 = Point { x: 1, y: 2 };
    let p2 = Point { x: 3, y: 4 };
    assert_eq!(p1 + p2, Point { x: 4, y: 6 });
    ```

### 3.5 模式匹配
模式匹配是Rust的核心语法，用于匹配值的结构，编译期强制**穷尽性检查**，确保所有情况都被覆盖，功能强大且安全。

#### 3.5.1 match表达式
match是模式匹配的核心，支持所有模式类型，语法如下：
```rust
let num = Some(5);
match num {
    Some(0) => println!("零"),
    Some(1..=10) => println!("1-10之间"),
    Some(v) => println!("值为：{}", v),
    None => println!("空值"),
}
```
- 核心规则：match分支必须覆盖所有可能的情况，否则编译报错；`_`通配符可匹配所有剩余情况。
- 分支可使用`=>`执行表达式，多语句用`{}`包裹。

#### 3.5.2 模式的适用场景
模式不仅可用于match，还可用于以下场景：
1.  **if let**：单分支匹配，无需穷尽性检查
2.  **while let**：匹配成功时循环
3.  **for in**：遍历迭代器时解构
4.  **let语句**：解构复合类型
5.  **函数参数**：解构元组等类型

```rust
// let解构
let (x, y, z) = (1, 2, 3);
// for in解构
let v = vec!['a', 'b', 'c'];
for (index, val) in v.iter().enumerate() {
    println!("{}: {}", index, val);
}
```

#### 3.5.3 常用模式类型
| 模式类型 | 示例 | 说明 |
| :--- | :--- | :--- |
| 字面量模式 | `1`、`"hello"` | 匹配具体的字面量值 |
| 变量模式 | `v` | 匹配任意值，并绑定到变量v |
| 通配符模式 | `_` | 匹配任意值，忽略不绑定 |
| 范围模式 | `1..=5`、`'a'..='z'` | 匹配闭区间内的值 |
| 解构模式 | `Point {x, y}`、`(a,b)` | 解构结构体、枚举、元组、数组 |
| 匹配守卫 | `Some(v) if v > 10` | 分支额外添加条件判断 |
| @绑定 | `val @ 1..=10` | 匹配范围的同时，将值绑定到val变量 |

```rust
// @绑定示例
match num {
    val @ 1..=10 => println!("匹配的值：{}", val),
    _ => println!("不匹配"),
}

// 匹配守卫示例
let opt = Some(15);
match opt {
    Some(v) if v > 10 => println!("大于10"),
    Some(v) => println!("小于等于10"),
    None => println!("空"),
}
```

## 四、错误处理
Rust没有异常机制，通过**返回值**明确区分可恢复错误和不可恢复错误，强制开发者处理错误，避免未处理的异常崩溃。

### 4.1 不可恢复错误：panic!
`panic!`宏用于处理不可恢复的错误，立即终止程序，打印错误信息，默认会展开栈并清理资源，可配置为直接终止。
```rust
// 主动触发panic
panic!("程序发生致命错误，无法继续运行");

// 带位置信息的panic
let v = vec![1,2,3];
v[10]; // 索引越界，自动触发panic
```

### 4.2 可恢复错误：Result<T, E>
可恢复错误通过`Result<T, E>`枚举返回，开发者可根据成功/失败分支分别处理，核心处理方式如下：

#### 4.2.1 match匹配处理
```rust
use std::fs::File;
use std::io::Error;

let f: Result<File, Error> = File::open("test.txt");
match f {
    Ok(file) => println!("文件打开成功"),
    Err(e) => println!("文件打开失败：{}", e),
}
```

#### 4.2.2 快捷处理方法
- `unwrap()`：成功则返回值，失败则触发panic，适合测试和确定不会出错的场景
- `expect(msg)`：和unwrap()一致，可自定义panic的错误信息，更易调试
- `unwrap_or(default)`：成功返回值，失败返回默认值
- `unwrap_or_else(|e| {})`：成功返回值，失败执行闭包处理

```rust
let f = File::open("test.txt").unwrap(); // 失败直接panic
let f = File::open("test.txt").expect("文件打开失败，请检查文件是否存在");
```

#### 4.2.3 错误传播：?运算符
`?`运算符是Rust错误传播的核心语法，自动处理Result：成功则提取值继续执行，失败则将错误返回给调用者，大幅简化错误传播代码。
- 限制：`?`只能用在**返回Result类型（或Option类型）**的函数中
- 特性：自动通过`From` trait转换错误类型，兼容不同的错误类型

```rust
use std::fs::File;
use std::io::{self, Read};

// 读取文件内容，自动传播错误
fn read_file_content() -> Result<String, io::Error> {
    let mut f = File::open("test.txt")?; // 失败则直接返回Err
    let mut content = String::new();
    f.read_to_string(&mut content)?; // 失败则直接返回Err
    Ok(content)
}
```

### 4.3 自定义错误类型
复杂项目中，可自定义错误类型，实现`std::error::Error` trait，统一管理不同的错误类型，支持错误溯源和格式化。

## 五、模块化与包管理
Rust通过模块化系统组织代码，通过Cargo管理包、依赖、构建、测试等全流程，是Rust工程化的核心。

### 5.1 Cargo包管理器
Cargo是Rust官方的包管理器和构建工具，内置核心命令：
| 命令 | 功能 |
| :--- | :--- |
| `cargo new 项目名` | 创建新的Rust项目，默认二进制项目，加`--lib`创建库项目 |
| `cargo build` | 构建项目，生成debug版本，加`--release`生成release优化版本 |
| `cargo run` | 构建并运行项目 |
| `cargo check` | 快速检查代码编译错误，不生成可执行文件，速度极快 |
| `cargo test` | 运行项目中的测试用例 |
| `cargo doc` | 生成项目文档，加`--open`自动打开 |
| `cargo add 依赖名` | 添加依赖到Cargo.toml |

项目核心配置文件`Cargo.toml`，用于声明项目信息、依赖、特性等：
```toml
[package]
name = "rust_demo"
version = "0.1.0"
edition = "2021" # Rust版本，推荐2021
description = "Rust语法演示项目"

[dependencies]
# 依赖声明，格式：包名 = 版本号
serde = { version = "1.0", features = ["derive"] } # 带特性的依赖
rand = "0.8"
```

### 5.2 模块系统
Rust模块系统通过`mod`、`pub`、`use`三个关键字，实现代码的组织、可见性控制、作用域引入。

#### 5.2.1 模块定义与结构
- 根模块：二进制项目根模块是`src/main.rs`，库项目根模块是`src/lib.rs`
- 子模块：通过`mod 模块名;`声明子模块，对应`src/模块名.rs`文件，或`src/模块名/mod.rs`文件
- 嵌套模块：模块内可通过`mod 模块名 {}`定义嵌套子模块

```rust
// src/lib.rs 根模块
// 声明子模块
pub mod user;
pub mod utils;

// 定义嵌套模块
mod config {
    // 子模块内容
    pub mod database {
        pub fn get_config() -> String {
            String::from("数据库配置")
        }
    }
}
```

#### 5.2.2 可见性控制
Rust模块内的项（函数、结构体、枚举等）**默认私有**，通过`pub`关键字控制可见性，支持精细化的可见性范围：
| 可见性修饰 | 作用范围 |
| :--- | :--- |
| 无修饰（默认私有） | 仅当前模块及其子模块可访问 |
| `pub` | 所有地方可访问 |
| `pub(crate)` | 仅当前整个crate内可访问 |
| `pub(super)` | 仅父模块可访问 |
| `pub(in path)` | 仅指定路径的模块可访问 |

```rust
mod user {
    // 公有结构体
    #[derive(Debug)]
    pub struct User {
        pub name: String, // 公有字段
        age: u8, // 私有字段，外部无法直接访问
    }

    impl User {
        // 公有构造函数
        pub fn new(name: String, age: u8) -> Self {
            User { name, age }
        }

        // 公有方法
        pub fn get_age(&self) -> u8 {
            self.age
        }
    }

    // 私有函数，仅当前模块可访问
    fn private_func() {}

    // crate内公有函数
    pub(crate) fn crate_func() {}
}
```

#### 5.2.3 use关键字：引入作用域
`use`关键字用于将模块、类型、函数等引入当前作用域，简化调用，支持别名、重导出。
```rust
// 基础引入
use std::fs::File;
// 别名引入
use std::io::Error as IoError;
// 批量引入
use std::io::{self, Read, Write};
// 通配符引入（谨慎使用，避免命名冲突）
use std::collections::*;
// 重导出：将引入的项作为当前模块的公有项导出
pub use crate::user::User;
```

## 六、高级语法特性
### 6.1 闭包
闭包是**匿名函数**，可捕获环境中的变量，语法简洁，支持类型自动推导，常用于迭代器、回调、线程等场景。

#### 6.1.1 闭包语法
```rust
// 基础闭包：|参数| 表达式
let add = |a: i32, b: i32| a + b;
println!("{}", add(1, 2)); // 输出3

// 多语句闭包：用{}包裹
let calculate = |x: i32| {
    let y = x * 2;
    y + 10
};

// 无参数闭包
let say_hello = || println!("hello");
say_hello();
```
- 闭包的参数和返回值类型可自动推导，无需手动标注，仅当编译器无法推导时需手动标注。

#### 6.1.2 闭包捕获环境
闭包可捕获当前作用域中的变量，有3种捕获方式，对应3个Trait：
| Trait | 捕获方式 | 说明 |
| :--- | :--- | :--- |
| `FnOnce` | 转移所有权 | 闭包获取变量的所有权，只能调用一次 |
| `FnMut` | 可变借用 | 闭包可变借用变量，可修改变量，可多次调用 |
| `Fn` | 不可变借用 | 闭包不可变借用变量，只读，可多次调用 |

`move`关键字可强制闭包获取环境变量的所有权，常用于线程场景，确保闭包生命周期超过变量生命周期。
```rust
let x = 10;
// move闭包：获取x的所有权
let closure = move || println!("x: {}", x);
closure();
// println!("{}", x); // 编译报错：x所有权已转移给闭包

// FnMut闭包：可变借用
let mut count = 0;
let mut inc = || {
    count += 1;
    println!("count: {}", count);
};
inc();
inc();
```

#### 6.1.3 闭包作为参数和返回值
```rust
// 闭包作为参数
fn apply<F: Fn(i32) -> i32>(f: F, x: i32) -> i32 {
    f(x)
}
let res = apply(|x| x * 2, 10); // 输出20

// 闭包作为返回值，必须用impl Trait
fn return_closure() -> impl Fn(i32) -> i32 {
    |x| x + 10
}
let closure = return_closure();
println!("{}", closure(5)); // 输出15
```

### 6.2 迭代器
迭代器是Rust中处理集合序列的核心抽象，惰性求值（只有消费时才会执行），安全高效，无运行时开销。所有实现了`Iterator` trait的类型都可作为迭代器。

#### 6.2.1 迭代器核心
`Iterator` trait的核心是`next()`方法，返回`Option<Item>`，有值时返回`Some(值)`，迭代结束返回`None`。
```rust
// 手动实现迭代器
struct Counter {
    count: u32,
    max: u32,
}

impl Counter {
    fn new(max: u32) -> Self {
        Counter { count: 0, max }
    }
}

impl Iterator for Counter {
    type Item = u32;
    fn next(&mut self) -> Option<Self::Item> {
        self.count += 1;
        if self.count <= self.max {
            Some(self.count)
        } else {
            None
        }
    }
}

// 使用
let mut counter = Counter::new(3);
assert_eq!(counter.next(), Some(1));
assert_eq!(counter.next(), Some(2));
assert_eq!(counter.next(), Some(3));
assert_eq!(counter.next(), None);
```

#### 6.2.2 迭代器分类
1.  **消费适配器（Consumer）**：消耗迭代器，产生最终结果，触发迭代执行
    常用：`collect()`、`sum()`、`count()`、`fold()`、`for_each()`、`max()`、`min()`
    ```rust
    let v = vec![1,2,3,4,5];
    let sum: i32 = v.iter().sum(); // 求和，sum=15
    let count = v.iter().count(); // 计数，count=5
    let new_v: Vec<i32> = v.iter().map(|x| x * 2).collect(); // 收集到Vec
    ```

2.  **迭代器适配器（Iterator Adapter）**：接收迭代器，返回新的迭代器，可链式调用，惰性执行
    常用：`map()`、`filter()`、`take()`、`skip()`、`zip()`、`enumerate()`、`flat_map()`
    ```rust
    let v = vec![1,2,3,4,5,6];
    // 链式调用：过滤偶数 -> 乘以2 -> 收集到Vec
    let res: Vec<i32> = v.into_iter()
        .filter(|x| x % 2 == 0)
        .map(|x| x * 2)
        .collect();
    // res = [4, 8, 12]
    ```

### 6.3 并发编程
Rust在编译期就保证了**线程安全**，彻底避免数据竞争，同时支持消息传递、共享状态等多种并发模型，无GC开销。

#### 6.3.1 线程创建与管理
通过`std::thread::spawn`创建新线程，通过`join()`等待线程执行完成。
```rust
use std::thread;
use std::time::Duration;

// 创建线程
let handle = thread::spawn(|| {
    for i in 1..5 {
        println!("子线程：{}", i);
        thread::sleep(Duration::from_millis(100));
    }
});

// 主线程执行
for i in 1..3 {
    println!("主线程：{}", i);
    thread::sleep(Duration::from_millis(200));
}

// 等待子线程执行完成，避免主线程提前退出
handle.join().unwrap();
```

线程中使用环境变量，需通过`move`闭包转移所有权，确保线程生命周期安全：
```rust
let v = vec![1,2,3];
let handle = thread::spawn(move || {
    println!("向量：{:?}", v);
});
handle.join().unwrap();
```

#### 6.3.2 消息传递：通道（Channel）
Rust遵循“通过通信共享内存，而非通过共享内存通信”，通过通道实现线程间消息传递，标准库提供`std::sync::mpsc`（多生产者，单消费者）通道。
```rust
use std::sync::mpsc;
use std::thread;

// 创建通道：发送者tx，接收者rx
let (tx, rx) = mpsc::channel();

// 子线程发送消息
thread::spawn(move || {
    let msg = String::from("hello from thread");
    tx.send(msg).unwrap(); // send会转移所有权
});

// 主线程接收消息
let received = rx.recv().unwrap(); // recv会阻塞等待消息
println!("收到：{}", received);

// 多生产者：克隆发送者
let tx2 = mpsc::Sender::clone(&tx);
thread::spawn(move || {
    tx2.send("第二条消息").unwrap();
});
```

#### 6.3.3 共享状态：互斥锁（Mutex）与原子引用计数（Arc）
多线程共享可变状态，通过`Mutex<T>`实现互斥访问，保证同一时间只有一个线程可修改数据；通过`Arc<T>`（原子引用计数）实现多线程多所有权，线程安全。
```rust
use std::sync::{Arc, Mutex};
use std::thread;

// Arc<Mutex<T>>：多线程共享可变数据
let counter = Arc::new(Mutex::new(0));
let mut handles = vec![];

// 创建10个线程，每个线程对计数器+1
for _ in 0..10 {
    let counter = Arc::clone(&counter);
    let handle = thread::spawn(move || {
        let mut num = counter.lock().unwrap(); // 获取锁
        *num += 1;
    }); // 锁自动释放
    handles.push(handle);
}

// 等待所有线程完成
for handle in handles {
    handle.join().unwrap();
}

println!("最终计数：{}", *counter.lock().unwrap()); // 输出10
```

#### 6.3.4 线程安全核心：Send与Sync Trait
- `Send` Trait：实现了`Send`的类型，所有权可在线程间安全转移，Rust中绝大多数类型都实现了`Send`，除了`Rc<T>`等。
- `Sync` Trait：实现了`Sync`的类型，可安全地被多个线程同时引用，`Arc<T>`实现了`Sync`，`RefCell<T>`未实现。
- 同时实现了`Send`和`Sync`的类型，是完全线程安全的。

### 6.4 宏系统
Rust宏是**元编程**工具，编译期展开生成代码，支持可变参数、语法扩展，分为声明宏和过程宏两大类。

#### 6.4.1 声明宏：macro_rules!
声明宏通过模式匹配生成代码，最常用，如`println!`、`vec!`都是声明宏。
```rust
// 自定义vec!宏
#[macro_export]
macro_rules! my_vec {
    // 匹配空参数：my_vec![]
    () => {
        Vec::new()
    };
    // 匹配多个元素：my_vec![1,2,3]
    ($($x:expr),+ $(,)?) => {
        {
            let mut temp_vec = Vec::new();
            $(
                temp_vec.push($x);
            )+
            temp_vec
        }
    };
}

// 使用
let v = my_vec![1,2,3];
assert_eq!(v, vec![1,2,3]);
```

#### 6.4.2 过程宏
过程宏操作编译期的AST（抽象语法树），功能更强大，分为3类：
1.  **派生宏（Derive Macro）**：通过`#[derive(宏名)]`为结构体/枚举自动生成代码，如`#[derive(Debug)]`、`#[derive(Serialize)]`。
2.  **属性宏（Attribute Macro）**：自定义属性，如`#[route(GET, "/")]`，可修改目标项的AST。
3.  **函数式宏**：类似声明宏，接收TokenStream作为输入，处理后返回新的TokenStream，语法更灵活。

### 6.5 Unsafe Rust
Rust提供`unsafe`关键字，关闭部分编译器安全检查，用于底层操作（如硬件访问、FFI、优化性能等），unsafe不代表代码一定不安全，而是要求开发者手动保证内存安全。

#### 6.5.1 unsafe的五大能力
unsafe块/函数中可执行以下操作：
1.  解引用裸指针
2.  调用unsafe函数/方法（包括FFI）
3.  访问或修改可变静态变量
4.  实现不安全的Trait
5.  访问union的字段

```rust
// 1. 解引用裸指针
let mut x = 10;
let r1 = &x as *const i32; // 不可变裸指针 *const T
let r2 = &mut x as *mut i32; // 可变裸指针 *mut T

unsafe {
    println!("r1: {}", *r1);
    *r2 = 20;
    println!("r2: {}", *r2);
}

// 2. 调用unsafe函数
unsafe fn dangerous() {
    println!("unsafe函数");
}

unsafe {
    dangerous();
}
```

#### 6.5.2 注意事项
- unsafe块应尽可能小，只包裹需要关闭检查的代码，其余代码仍受编译器安全检查保护。
- unsafe代码需严格遵守Rust的内存安全规则，否则会出现未定义行为。
- 对外暴露的unsafe函数，必须在文档中明确标注安全使用的前提条件。

## 七、常用补充语法
### 7.1 字符串处理
Rust字符串核心分为两种，均为UTF-8编码：
1.  **`&str`**：字符串切片，不可变，无所有权，可存储在栈、静态内存、堆中，字符串字面量默认是`&'static str`。
2.  **`String`**：堆上分配的可增长字符串，有所有权，底层是`Vec<u8>`封装。

常用方法：
```rust
// String创建与修改
let mut s = String::new();
s.push_str("hello"); // 追加字符串
s.push('!'); // 追加字符
let s2 = s.replace("hello", "hi"); // 替换
let s3 = s.trim(); // 去除首尾空白
let parts: Vec<&str> = s.split(' ').collect(); // 分割

// 类型转换
let s = "hello".to_string(); // &str -> String
let s_ref = s.as_str(); // String -> &str
let num = "123".parse::<i32>().unwrap(); // 字符串转数字
```
- 注意：Rust字符串不支持直接通过索引访问单个字符（`s[0]`），因为UTF-8是变长编码，需通过`s.chars()`遍历字符，`s.bytes()`遍历字节。

### 7.2 常用集合类型
| 集合类型 | 说明 | 核心场景 |
| :--- | :--- | :--- |
| `Vec<T>` | 动态数组，堆上分配，可增长 | 有序、可重复的序列存储 |
| `HashMap<K, V>` | 哈希表，键值对存储，无序 | 快速通过键查找值 |
| `HashSet<T>` | 哈希集合，无序、不可重复 | 去重、集合运算 |
| `BTreeMap<K, V>` | 有序键值对，基于B树，键必须可排序 | 有序的键值存储、范围查询 |
| `BTreeSet<T>` | 有序集合，基于B树，元素必须可排序 | 有序去重、范围查询 |

```rust
// Vec<T> 常用操作
let mut v = Vec::new();
v.push(1);
v.push(2);
v.pop();
println!("{}", v[0]);

// HashMap<K,V> 常用操作
use std::collections::HashMap;
let mut map = HashMap::new();
map.insert("name", "张三");
map.insert("age", "20");
let name = map.get("name");
map.remove("age");
```

### 7.3 常用内置Trait
| Trait | 核心功能 | 常用场景 |
| :--- | :--- | :--- |
| `Debug` | 格式化调试打印 | `#[derive(Debug)]`，支持`println!("{:?}", x)` |
| `Clone` | 显式深拷贝 | 调用`clone()`方法复制值 |
| `Copy` | 隐式栈上复制 | 赋值时自动复制，不转移所有权 |
| `PartialEq/Eq` | 相等性比较 | 支持`==`/`!=`运算符 |
| `PartialOrd/Ord` | 排序比较 | 支持`>`/`<`/`>=`/`<=`运算符 |
| `Default` | 默认值 | 生成类型的默认实例 |
| `Deref/DerefMut` | 解引用转换 | 智能指针、自动解引用 |
| `Drop` | 析构函数 | 值离开作用域时执行清理逻辑 |
| `Iterator` | 迭代器 | 实现for in循环、迭代器适配器 |
| `From/Into` | 类型转换 | 安全的类型间转换，`?`自动转换错误 |
| `Error` | 错误类型规范 | 自定义错误类型 |

### 7.4 命名规范
Rust官方推荐的命名规范，编译器会给出警告提示：
| 类型 | 命名规范 | 示例 |
| :--- | :--- | :--- |
| 变量、函数、模块、宏参数 | 蛇形命名（snake_case） | `user_name`、`add_num`、`utils` |
| 结构体、枚举、Trait、类型别名 | 大驼峰命名（PascalCase） | `User`、`Message`、`Summary` |
| 常量、静态变量 | 全大写蛇形命名（SHOUTING_SNAKE_CASE） | `MAX_NUM`、`GLOBAL_CONFIG` |
| 生命周期标注 | 单个小写字母 | `'a`、`'b` |


