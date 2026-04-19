# Zsh 语法详细总结


# Zsh 语法详细总结
Zsh（Z Shell）是一款兼容POSIX标准的Unix Shell，完全兼容Bash基础语法，同时提供了更强大的扩展特性、更灵活的编程能力和更友好的交互体验，是目前主流的交互式Shell和脚本开发环境。本文档系统梳理Zsh核心语法、独有特性及与Bash的关键差异，兼顾入门使用和高级脚本开发。

## 一、脚本基础与执行规范
### 1. 脚本声明
脚本首行需指定解释器，推荐两种写法：
```zsh
#!/usr/bin/env zsh  # 推荐，自动适配zsh路径
#!/bin/zsh          # 固定路径写法
```

### 2. 执行方式与核心区别
| 执行方式 | 执行环境 | 特点 |
|----------|----------|------|
| `chmod +x script.zsh && ./script.zsh` | 子Shell | 独立进程，变量不污染当前Shell，脚本需执行权限 |
| `zsh script.zsh` | 子Shell | 无需执行权限，直接通过zsh解释器执行 |
| `source script.zsh` / `. script.zsh` | 当前Shell | 在当前Shell中执行，变量/函数会保留，常用于配置加载 |

### 3. 注释语法
- 单行注释：以`#`开头，行内`#`后内容均为注释
  ```zsh
  # 这是单行注释
  echo "hello" # 这是行尾注释
  ```
- 多行注释：两种常用写法
  ```zsh
  # 写法1：单引号包裹（推荐）
  : '
  这是多行注释
  第二行注释
  '

  # 写法2：Here Document
  <<'COMMENT'
  这是多行注释
  第二行注释
  COMMENT
  ```

### 4. 引号与转义规则
| 引号类型 | 特性 | 适用场景 |
|----------|------|----------|
| 单引号 `' '` | 强引用，所有字符均作为普通字符，不解析变量、命令、转义（仅单引号本身需转义） | 固定字符串、正则表达式、避免意外解析 |
| 双引号 `" "` | 弱引用，解析`$`变量、命令替换、转义字符，不解析通配符 | 包含变量的字符串、保留空格的场景 |
| 反斜杠 `\` | 转义特殊字符（`$`、```、`"`、`'`、`*`等），`\n`换行、`\t`制表符 | 单个特殊字符转义 |

> 核心差异：Zsh中未加引号的变量**默认不进行IFS单词分割**，Bash会默认分割，这是最常见的踩坑点。
> 示例：`var="a b c"; echo $var` 在Zsh中视为单个参数，Bash中会拆分为3个参数。

### 5. 命令替换
将命令的输出赋值给变量或嵌入字符串，推荐使用`$()`写法，支持嵌套：
```zsh
# 推荐写法
now=$(date +%Y-%m-%d)
file_count=$(ls *.txt | wc -l)

# 兼容写法（反引号，嵌套需转义）
now=`date +%Y-%m-%d`
```

## 二、变量与数据类型
Zsh支持字符串、整数、浮点数、数组、关联数组等数据类型，提供丰富的变量操作能力。

### 1. 变量定义与基础操作
- 基础赋值：**等号两侧不能有空格**
  ```zsh
  # 普通赋值
  name="zsh"
  version=5.9

  # 只读变量（不可修改）
  readonly ROOT_UID=0
  typeset -r ROOT_PATH="/root"

  # 导出为环境变量（子进程可访问）
  export PATH="$HOME/bin:$PATH"
  typeset -x JAVA_HOME="/usr/lib/jvm/default-java"
  ```
- 变量引用：`$var` 或 `${var}`，复杂场景必须用花括号包裹
  ```zsh
  echo "Hello $name"
  echo "Version ${version}_release" # 花括号避免歧义
  ```

### 2. 字符串处理（Zsh核心扩展）
Zsh内置丰富的字符串操作，无需依赖`sed`/`awk`/`tr`等外部命令。

| 操作 | 语法 | 示例与说明 |
|------|------|------------|
| 字符串拼接 | 直接拼接 | `str1="hello"; str2="$str1 world"` → `hello world` |
| 长度获取 | `${#str}` | `${#"hello zsh"}` → 9 |
| 切片截取 | `${str:start:length}` | **索引默认从1开始**，`str="abcdef"; ${str:2:3}` → `bcd`；`${str:-3}` 取最后3个字符 |
| 前缀删除 | `${str#pattern}`（最短匹配）<br>`${str##pattern}`（最长匹配） | `str="/home/user/file.txt"; ${str##*/}` → `file.txt` |
| 后缀删除 | `${str%pattern}`（最短匹配）<br>`${str%%pattern}`（最长匹配） | `str="file.tar.gz"; ${str%%.*}` → `file` |
| 内容替换 | `${str/old/new}`（替换首个）<br>`${str//old/new}`（全局替换） | `str="a b c a"; ${str//a/x}` → `x b c x` |
| 首尾替换 | `${str/#old/new}`（行首替换）<br>`${str/%old/new}`（行尾替换） | `str="hello.txt"; ${str/%.txt/.md}` → `hello.md` |
| 大小写转换 | `${str:l}` 全小写<br>`${str:u}` 全大写<br>`${str:c}` 首字母大写 | `str="Hello Zsh"; ${str:u}` → `HELLO ZSH` |

### 3. 数值运算
Zsh**原生支持整数和浮点数运算**，这是与Bash的核心差异（Bash仅支持整数，浮点需依赖`bc`/`awk`）。
```zsh
# 整数运算
(( a = 1 + 2 * 3 ))  # a=7
(( a++ ))             # 自增，a=8
(( a-- ))             # 自减，a=7
echo $(( 10 / 3 ))   # 整数除法，输出3

# 浮点数运算（Zsh独有）
(( f = 1.5 + 2.5 ))  # f=4.0
(( pi = 3.1415926 * 2 ))
echo $(( 10.0 / 3 )) # 浮点除法，输出3.3333333333333335

# 显式声明数值类型
typeset -i int_var=10    # 声明整数
typeset -F float_var=3.14 # 声明浮点数
```

### 4. 特殊变量与内置变量
| 变量 | 含义 |
|------|------|
| `$0` | 脚本/函数名称 |
| `$1~$n` | 第n个位置参数 |
| `$#` | 传入的参数个数 |
| `$@` | 所有参数（数组形式，保留参数边界） |
| `$*` | 所有参数（单个字符串形式） |
| `$$` | 当前Shell进程PID |
| `$?` | 上一条命令的退出码（0=成功，非0=失败） |
| `$!` | 最后一个后台进程的PID |
| `$PWD` | 当前工作目录 |
| `$HOME` | 当前用户家目录 |
| `$PATH` | 可执行文件查找路径 |
| `$ZSH_VERSION` | Zsh版本号 |
| `$RANDOM` | 0~32767的随机整数 |
| `$SECONDS` | 脚本/Shell启动后的运行秒数 |
| `$LINENO` | 当前执行的行号 |

### 5. 变量修饰符与作用域
通过`typeset`（别名`declare`）声明变量属性，控制作用域和类型：
| 修饰符 | 作用 |
|--------|------|
| `-i` | 声明为整数类型 |
| `-F` | 声明为浮点数类型 |
| `-a` | 声明为索引数组 |
| `-A` | 声明为关联数组 |
| `-r` | 声明为只读变量 |
| `-x` | 导出为环境变量 |
| `-l` | 赋值时自动转为小写 |
| `-u` | 赋值时自动转为大写 |
| `-U` | 数组自动去重 |
| `-g` | 函数内声明全局变量（默认函数内为局部变量） |

## 三、索引数组（Array）核心语法
Zsh的数组功能远强于Bash，默认索引从1开始（Bash从0开始），支持负索引、灵活切片、自动去重等特性。

### 1. 数组定义与初始化
```zsh
# 基础定义（空格分隔元素）
arr=(apple banana cherry)

# 空数组
empty_arr=()

# 带索引赋值
arr[1]="apple"
arr[2]="banana"

# 范围创建
num_arr=( {1..10} )       # 1~10的整数数组
char_arr=( {a..z} )        # a~z的字母数组
seq_arr=( {0..10..2} )     # 步长2，0 2 4 6 8 10

# 声明数组并自动去重
typeset -U unique_arr=(a b a c b) # 最终为(a b c)
```

### 2. 数组访问与切片
```zsh
arr=(apple banana cherry date)

# 单个元素访问（索引从1开始）
echo ${arr[1]}  # apple
echo ${arr[-1]} # date（负索引，最后一个元素）

# 所有元素访问
echo ${arr[@]}  # 所有元素（数组形式，推荐）
echo ${arr[*]}  # 所有元素（单个字符串）

# 切片：${arr[@]:起始位置:长度}
echo ${arr[@]:2:2} # banana cherry（从第2个开始，取2个）
echo ${arr[@]:2}    # 从第2个取到末尾

# 数组长度
echo ${#arr[@]} # 4
```

### 3. 数组增删改操作
```zsh
arr=(a b c)

# 追加元素（Zsh独有简洁写法）
arr+=(d e) # 最终为(a b c d e)

# 修改元素
arr[2]="x" # 最终为(a x c d e)

# 删除元素
unset arr[3] # 删除第3个元素，最终为(a x d e)
arr=(${arr:#x}) # 删除匹配"x"的元素，最终为(a d e)

# 清空数组
arr=()
```

### 4. 数组遍历与合并
```zsh
# 数组合并
arr1=(a b c)
arr2=(d e f)
arr3=(${arr1[@]} ${arr2[@]}) # (a b c d e f)

# 遍历数组（推荐，保留带空格的元素）
for val in "${arr[@]}"; do
  echo "元素：$val"
done

# 带索引遍历
for i in {1..${#arr[@]}}; do
  echo "索引$i：${arr[$i]}"
done
```

## 四、关联数组（哈希/字典）语法
Zsh原生支持键值对形式的关联数组，提供比Bash更便捷的遍历和操作能力。

### 1. 声明与初始化
**必须先通过`typeset -A`/`declare -A`声明**，再进行赋值：
```zsh
# 声明空关联数组
typeset -A user_map

# 批量赋值（key1 value1 key2 value2）
user_map=(
  name "zhangsan"
  age 20
  shell "zsh"
)

# 单个键值赋值
user_map[email]="zhangsan@example.com"

# 键值对完整写法
user_map=( [name]="lisi" [age]=25 [shell]="bash" )
```

### 2. 访问与遍历
```zsh
typeset -A user_map=(name "zhangsan" age 20 shell "zsh")

# 单个值访问
echo ${user_map[name]} # zhangsan

# 获取所有key
echo ${(k)user_map[@]}  # Zsh原生写法
echo ${!user_map[@]}     # Bash兼容写法

# 获取所有value
echo ${(v)user_map[@]}  # Zsh原生写法
echo ${user_map[@]}      # 简写

# 元素个数
echo ${#user_map[@]} # 3

# 遍历key（通用写法）
for key in "${(k)user_map[@]}"; do
  echo "$key : ${user_map[$key]}"
done

# Zsh独有：直接遍历键值对（无需二次取值）
for key value in "${(@kv)user_map}"; do
  echo "$key -> $value"
done
```

### 3. 增删改与存在性判断
```zsh
typeset -A user_map=(name "zhangsan" age 20)

# 修改值
user_map[age]=21

# 删除键值对
unset user_map[age]

# 判断key是否存在
if [[ -n ${user_map[name]+x} ]]; then
  echo "key存在"
else
  echo "key不存在"
fi
```

## 五、流程控制语句
Zsh完全兼容Bash的流程控制语法，同时提供`repeat`等扩展循环，推荐使用`[[ ]]`进行条件判断，避免单中括号的语法陷阱。

### 1. 条件判断：if/elif/else
#### 基础语法
```zsh
if 条件表达式; then
  # 条件成立执行
elif 条件表达式2; then
  # 条件2成立执行
else
  # 所有条件不成立执行
fi
```

#### 核心条件表达式
- **双中括号 `[[ ]]`**：Zsh推荐用法，支持通配符、正则匹配，无单词分割问题
- **双括号 `(( ))`**：专用数值比较，支持`> < >= <= == != && ||`运算符

#### 常用判断符
| 分类 | 判断符 | 含义 |
|------|--------|------|
| 文件判断 | `-f` | 存在且为普通文件 |
| | `-d` | 存在且为目录 |
| | `-e` | 文件/目录存在 |
| | `-r/-w/-x` | 可读/可写/可执行 |
| | `-s` | 存在且非空 |
| | `-L` | 存在且为符号链接 |
| 字符串判断 | `==` | 相等（支持通配符匹配） |
| | `!=` | 不相等 |
| | `=~` | 正则匹配 |
| | `-z` | 字符串为空 |
| | `-n` | 字符串非空 |
| 数值判断（(( ))） | `> < >= <= == !=` | 大于/小于/大于等于/小于等于/等于/不等于 |
| 数值判断（[[ ]]） | `-gt/-lt/-ge/-le/-eq/-ne` | 对应上述数值比较 |

#### 示例
```zsh
# 数值判断
num=10
if (( num > 5 )); then
  echo "num大于5"
fi

# 字符串匹配
filename="test.txt"
if [[ $filename == *.txt ]]; then
  echo "是txt文件"
fi

# 正则匹配
str="abc123"
if [[ $str =~ ^[a-z]+[0-9]+$ ]]; then
  echo "格式匹配"
fi

# 文件判断
if [[ -f ~/.zshrc ]]; then
  echo "zsh配置文件存在"
fi
```

### 2. 循环语句
#### for循环（最常用）
```zsh
# 写法1：遍历列表
for i in {1..5}; do
  echo "数字：$i"
done

# 写法2：C语言风格
for (( i=0; i<5; i++ )); do
  echo "数字：$i"
done

# 写法3：遍历文件
for file in *.txt; do
  echo "处理文件：$file"
done
```

#### while循环
条件为真时循环执行：
```zsh
i=1
while (( i <= 5 )); do
  echo "数字：$i"
  ((i++))
done

# 逐行读取文件
while read -r line; do
  echo "行内容：$line"
done < file.txt
```

#### until循环
条件为假时循环执行（与while相反）：
```zsh
i=1
until (( i > 5 )); do
  echo "数字：$i"
  ((i++))
done
```

#### repeat循环（Zsh独有）
固定循环次数，语法更简洁：
```zsh
# 重复执行5次
repeat 5; do
  echo "hello zsh"
done
```

#### 循环控制符
- `break`：跳出当前整个循环
- `continue`：跳过本次循环，进入下一次循环
- `return`：函数内返回
- `exit`：退出整个脚本

### 3. case分支语句
支持多模式匹配、通配符，Zsh扩展支持分支穿透：
```zsh
case $var in
  pattern1)
    # 匹配pattern1执行
    ;; # 终止分支
  pattern2|pattern3)
    # 匹配pattern2或pattern3执行
    ;& # 穿透：继续执行下一个分支的代码（不判断匹配）
  pattern4)
    # 上一个分支使用;&，会执行到这里
    ;| # 继续匹配下一个模式
  *)
    # 默认分支，所有模式不匹配时执行
    ;;
esac
```

示例：
```zsh
read -r -p "输入一个字母：" char
case $char in
  [a-z])
    echo "小写字母"
    ;;
  [A-Z])
    echo "大写字母"
    ;;
  [0-9])
    echo "数字"
    ;;
  *)
    echo "其他字符"
    ;;
esac
```

## 六、函数定义与使用
Zsh支持多种函数定义语法，提供灵活的参数传递、作用域控制和返回值机制。

### 1. 函数定义语法
支持3种兼容写法，功能完全一致：
```zsh
# 写法1：推荐，兼容Bash
function func_name() {
  # 函数体
  echo "hello function"
}

# 写法2：Zsh简洁写法
func_name() {
  echo "hello function"
}

# 写法3：无括号写法
function func_name {
  echo "hello function"
}
```

### 2. 参数传递与接收
函数内的位置参数与脚本一致，调用时直接在函数名后加参数：
```zsh
# 定义加法函数
add() {
  echo "参数个数：$#"
  echo "第一个参数：$1"
  echo "第二个参数：$2"
  (( sum = $1 + $2 ))
  echo "和：$sum"
}

# 调用函数
add 1 2
```

### 3. 返回值与结果接收
- `return`：只能返回0~255的整数退出码，0代表成功，非0代表失败
- 字符串/数值结果：通过`echo`输出，用`$()`接收返回值

```zsh
# 退出码返回
is_even() {
  (( $1 % 2 == 0 ))
  return $? # 返回判断结果的退出码
}

if is_even 4; then
  echo "是偶数"
fi

# 通用结果返回
get_greeting() {
  local name=$1
  echo "Hello, $name" # 输出结果
}

# 接收返回值
greeting=$(get_greeting "zsh")
echo $greeting # 输出 Hello, zsh
```

### 4. 变量作用域
- 全局变量：函数外定义的变量，函数内可直接访问和修改
- 局部变量：函数内用`local`/`typeset`声明，仅在函数内生效，避免污染全局
- 全局声明：函数内用`typeset -g`声明全局变量

```zsh
global_var="全局变量"

test_scope() {
  local local_var="局部变量"
  typeset -g new_global="函数内声明的全局变量"

  global_var="修改后的全局变量"
  echo $local_var
}

test_scope
echo $global_var  # 正常输出
echo $new_global   # 正常输出
echo $local_var    # 空，局部变量外部无法访问
```

### 5. 高级特性
- 匿名函数：Zsh支持直接执行匿名函数
  ```zsh
  () {
    echo "匿名函数执行"
    local temp="临时变量"
  }
  ```
- 递归调用：函数支持递归
  ```zsh
  # 阶乘函数
  factorial() {
    (( $1 <= 1 )) && echo 1 && return
    echo $(( $1 * $(factorial $(( $1 - 1 ))) ))
  }
  factorial 5 # 输出120
  ```

## 七、重定向与管道
Zsh完全兼容Bash的重定向语法，同时提供扩展管道、进程替换等高级特性。

### 1. 标准IO重定向
Linux系统默认3个IO流：`stdin(0，标准输入)`、`stdout(1，标准输出)`、`stderr(2，标准错误)`。

| 语法 | 作用 |
|------|------|
| `command > file` | 覆盖stdout到file |
| `command >> file` | 追加stdout到file |
| `command 2> file` | 覆盖stderr到file |
| `command 2>> file` | 追加stderr到file |
| `command &> file` | 覆盖stdout+stderr到file（推荐） |
| `command &>> file` | 追加stdout+stderr到file |
| `command > file 2>&1` | 兼容写法，覆盖stdout+stderr到file |
| `command < file` | 从file读取stdin |
| `command > /dev/null` | 丢弃stdout |
| `command 2> /dev/null` | 丢弃stderr |
| `command &> /dev/null` | 丢弃所有输出 |

示例：
```zsh
# 正常输出写入文件，错误输出打印到终端
echo "hello" > output.log

# 错误输出写入文件，正常输出打印到终端
ls non_exist_file 2> error.log

# 所有输出都写入文件
ls &> all_output.log
```

### 2. 管道与扩展管道
- 基础管道：`command1 | command2`，将command1的stdout传递给command2的stdin
  ```zsh
  # 统计txt文件数量
  ls *.txt | wc -l
  # 筛选包含zsh的进程
  ps aux | grep zsh
  ```
- 扩展管道（Zsh独有）：`command1 |& command2`，将stdout+stderr同时传递给command2，等同于Bash的`2>&1 |`
  ```zsh
  # 同时筛选正常输出和错误输出
  ls non_exist_file |& grep "No such"
  ```

### 3. 进程替换
将命令的输入/输出模拟为临时文件，解决管道无法传递多文件的问题：
- `<(command)`：将command的输出作为可读文件
- `>(command)`：将command的输入作为可写文件

```zsh
# 对比两个目录的文件列表
diff <(ls dir1) <(ls dir2)

# 将输出同时传递给多个命令
echo "hello world" > >(grep hello) > >(wc -l)
```

### 4. 多行输入
- Here Document：多行文本输入，常用于脚本中生成配置文件
  ```zsh
  # EOF为结束符，单引号包裹EOF会禁用变量解析
  cat > test.txt << 'EOF'
  第一行内容
  第二行内容，$var不会被解析
  第三行内容
  EOF
  ```
- Here String：单行字符串输入，将字符串作为stdin传递给命令
  ```zsh
  # 等同于 echo "hello zsh" | grep zsh
  grep zsh <<< "hello zsh"
  ```

## 八、Zsh核心利器：通配符与扩展（Globbing）
通配符是Zsh最强大的特性之一，默认开启扩展通配符，支持递归匹配、排除匹配、文件属性过滤等，远超Bash的能力。

### 1. 基础通配符
| 通配符 | 作用 |
|--------|------|
| `*` | 匹配任意长度的任意字符（包括空） |
| `?` | 匹配单个任意字符 |
| `[abc]` | 匹配括号内的任意单个字符 |
| `[a-z0-9]` | 匹配范围内的任意单个字符 |
| `[^abc]`/`[!abc]` | 匹配非括号内的任意单个字符 |

### 2. 扩展通配符（默认开启）
| 语法 | 作用 | 示例 |
|------|------|------|
| `**` | 递归匹配所有子目录（Bash需手动开启globstar） | `ls **/*.txt` 递归查找所有txt文件 |
| `^pattern` | 否定匹配，匹配不符合pattern的内容 | `ls ^*.txt` 匹配所有非txt文件 |
| `pattern1~pattern2` | 排除匹配，匹配pattern1但排除pattern2 | `ls *.c~test.c` 匹配所有c文件，排除test.c |
| `(pat1|pat2)` | 分组或匹配，匹配pat1或pat2 | `ls (a|b)*.txt` 匹配a或b开头的txt文件 |
| `pattern#` | 匹配pattern零次或多次 | |
| `pattern##` | 匹配pattern一次或多次 | |

### 3. 全局限定符（Glob Qualifiers，Zsh独有）
Zsh独有的语法，在通配符后加`(限定符)`，可按文件类型、大小、时间、权限等精准过滤，无需结合`find`命令。

#### 常用限定符速览
| 分类 | 限定符 | 作用 |
|------|--------|------|
| 文件类型 | `.` | 普通文件 |
| | `/` | 目录 |
| | `@` | 符号链接 |
| | `*` | 可执行文件 |
| | `p` | 管道文件 |
| 大小过滤 | `Lk+N` | 大于N KB的文件 |
| | `Lm-N` | 小于N MB的文件 |
| | `L0` | 空文件 |
| 时间过滤 | `mmin-N` | N分钟内修改过的文件 |
| | `mtime-N` | N天内修改过的文件 |
| | `amin-N` | N分钟内访问过的文件 |
| 排序 | `on` | 按文件名升序 |
| | `On` | 按文件名降序 |
| | `om` | 按修改时间降序（最新在前） |
| | `oL` | 按文件大小升序 |
| | `OL` | 按文件大小降序 |
| 权限 | `r/w/x` | 可读/可写/可执行（当前用户） |
| | `W` | 全局可写文件 |
| | `U` | 当前用户拥有的文件 |
| 其他 | `D` | 匹配隐藏文件（.开头） |
| | `F` | 非空目录 |
| | `[1,N]` | 取前N个匹配结果 |

#### 实用示例
```zsh
# 递归查找所有普通txt文件
ls **/*.txt(.)

# 查找当前目录大于100MB的文件
ls *(Lm+100)

# 查找最近7天内修改过的php文件
ls **/*.php(mtime-7)

# 查找最近修改的5个文件
ls *(om[1,5])

# 只列出当前目录的子目录
ls -d *(/)

# 递归查找所有隐藏文件
ls **/*(D)

# 查找当前目录所有空文件并删除
rm *(L0)
```

### 4. 模式修饰符
用于调整匹配规则，常用：
- `(#i)`：不区分大小写匹配
  ```zsh
  # 匹配hello.txt、HELLO.TXT、Hello.Txt等
  [[ "Hello.txt" == (#i)hello.txt ]]
  ls (#i)*.txt
  ```
- `(#b)`：开启分组捕获，配合正则使用
  ```zsh
  [[ "abc123" == (#b)([a-z]*)([0-9]*) ]]
  echo $match[1] # abc
  echo $match[2] # 123
  ```

## 九、别名与快捷键
Zsh提供3种别名类型，远超Bash的普通别名，同时支持丰富的快捷键自定义。

### 1. 别名分类与定义
#### 普通别名
与Bash完全兼容，仅在命令开头生效，常用于简化长命令：
```zsh
# 定义别名
alias ll='ls -lAh'
alias gs='git status'
alias grep='grep --color=auto'

# 取消别名
unalias ll

# 查看所有别名
alias
```

#### 全局别名（Zsh独有）
用`alias -g`定义，在命令行任意位置都生效，常用于简化管道、重定向等固定片段：
```zsh
# 定义全局别名
alias -g G='| grep'
alias -g L='| less'
alias -g H='--help'
alias -g N='&> /dev/null'

# 使用示例
ps aux G zsh # 等同于 ps aux | grep zsh
cat file.log L # 等同于 cat file.log | less
ls non_exist N # 等同于 ls non_exist &> /dev/null
```

#### 后缀别名（Zsh独有）
用`alias -s`定义，绑定文件后缀与打开程序，输入文件名直接用指定程序打开：
```zsh
# 定义后缀别名
alias -s txt='vim'
alias -s md='typora'
alias -s py='python3'
alias -s zip='unzip -l'

# 使用示例
test.txt   # 直接用vim打开test.txt
script.py  # 直接用python3执行script.py
```

### 2. 常用内置快捷键
Zsh默认使用Emacs按键模式，可通过`bindkey -v`切换为Vi模式，`bindkey -e`切回Emacs模式。

| 快捷键 | 功能 |
|--------|------|
| `Ctrl+A` | 跳到行首 |
| `Ctrl+E` | 跳到行尾 |
| `Ctrl+U` | 删除光标到行首的内容 |
| `Ctrl+K` | 删除光标到行尾的内容 |
| `Ctrl+W` | 删除光标前一个单词 |
| `Alt+D` | 删除光标后一个单词 |
| `Ctrl+R` | 反向搜索历史命令 |
| `Ctrl+L` | 清屏，等同于clear命令 |
| `!!` | 执行上一条命令 |
| `!$` | 引用上一条命令的最后一个参数 |
| `!n` | 执行历史中第n条命令 |

## 十、正则表达式与模式匹配
Zsh内置正则匹配能力，无需依赖外部命令，支持扩展正则语法和分组捕获。

### 1. 基础正则匹配
使用`[[ str =~ regex ]]`语法进行正则匹配，匹配成功返回0，失败返回非0：
```zsh
# 匹配手机号
phone="13812345678"
if [[ $phone =~ ^1[3-9][0-9]{9}$ ]]; then
  echo "手机号格式正确"
fi

# 匹配邮箱
email="test@example.com"
if [[ $email =~ ^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$ ]]; then
  echo "邮箱格式正确"
fi
```

### 2. 匹配结果捕获
Zsh会自动将匹配结果存入内置变量：
- `$MATCH`：匹配到的完整字符串
- `$match`：数组，存储分组捕获的内容
- `$MBEGIN`/`$MEND`：匹配内容的起始/结束位置

示例：
```zsh
str="name:zhangsan,age:20"
if [[ $str =~ name:([a-z]+),age:([0-9]+) ]]; then
  echo "完整匹配：$MATCH"
  echo "姓名：$match[1]"
  echo "年龄：$match[2]"
fi
```

### 3. 扩展正则配置
- 开启扩展正则：`setopt EXTENDED_REGEX`
- 关闭大小写敏感：`setopt NO_CASE_GLOB`

## 十一、脚本选项与调试
Zsh通过`setopt`/`unsetopt`开启/关闭选项，对应Bash的`set`/`shopt`，用于控制脚本行为、错误处理和调试。

### 1. 核心脚本选项
脚本开头推荐配置，提升脚本健壮性：
```zsh
#!/usr/bin/env zsh
# 推荐生产环境配置
setopt ERR_EXIT       # 命令执行失败时立即退出脚本（等同于set -e）
setopt NO_UNSET       # 使用未定义变量时报错（等同于set -u）
setopt PIPE_FAIL      # 管道中任意命令失败，整个管道退出码为失败
setopt NO_NOMATCH     # 通配符匹配不到文件时，保留原字符串，不报错
```

| 选项 | 简写 | 作用 |
|------|------|------|
| `ERR_EXIT` | `set -e` | 命令非0退出时，立即终止脚本 |
| `NO_UNSET` | `set -u` | 使用未定义变量时报错 |
| `PIPE_FAIL` | - | 管道失败判断，默认仅看最后一条命令 |
| `XTRACE` | `set -x` | 调试模式，打印执行的每一条命令 |
| `NO_NOMATCH` | - | 通配符无匹配时不报错，避免脚本中断 |
| `EXTENDED_GLOB` | - | 开启扩展通配符（默认开启） |
| `NO_CASE_GLOB` | - | 通配符不区分大小写 |

### 2. 脚本调试方法
```zsh
# 方式1：命令行直接调试执行
zsh -x script.zsh

# 方式2：脚本内开启/关闭调试
set -x # 开启调试
# 要调试的代码段
set +x # 关闭调试
```

### 3. 退出码与错误处理
- 脚本退出：`exit N`，N为退出码（0=成功，1~255=错误）
- 错误捕获：使用`||`处理命令失败场景
  ```zsh
  # 命令失败时执行兜底逻辑
  ls non_exist_file || echo "文件不存在"
  # 命令失败时直接退出
  cd /data || exit 1
  ```

## 十二、环境配置文件加载规则
Zsh的配置文件分全局配置（/etc下）和用户配置（~下），按Shell类型不同，加载顺序不同。

### 1. 配置文件加载顺序
| Shell类型 | 加载顺序 |
|-----------|----------|
| 登录Shell | `/etc/zshenv` → `~/.zshenv` → `/etc/zprofile` → `~/.zprofile` → `/etc/zshrc` → `~/.zshrc` → `/etc/zlogin` → `~/.zlogin` |
| 非登录交互式Shell | `/etc/zshenv` → `~/.zshenv` → `/etc/zshrc` → `~/.zshrc` |
| 非交互式Shell（脚本执行） | 仅加载 `/etc/zshenv` → `~/.zshenv` |

### 2. 核心配置文件适用场景
| 配置文件 | 适用场景 |
|----------|----------|
| `~/.zshenv` | 总是加载，用于设置全局环境变量（如`PATH`），无论是否交互式 |
| `~/.zshrc` | 交互式Shell加载，最常用的配置文件，用于定义别名、函数、快捷键、补全、主题等 |
| `~/.zprofile` | 登录Shell加载，用于登录时执行一次的命令（如启动服务、环境初始化） |
| `~/.zlogout` | 退出登录Shell时执行，用于清理操作（如清除临时文件、打印退出提示） |

## 十三、Zsh与Bash核心语法差异对照表
| 特性 | Zsh | Bash |
|------|-----|------|
| 数组索引 | 默认从1开始，支持负索引 | 默认从0开始，4.0+版本支持负索引 |
| 单词分割 | 变量默认不进行IFS单词分割 | 变量默认会进行IFS单词分割，需加双引号避免 |
| 浮点数运算 | 原生支持`(( 1.5+2.5 ))` | 不原生支持，需依赖`bc`/`awk`等外部工具 |
| 递归通配符 | `**`默认开启，直接使用 | 需手动执行`shopt -s globstar`开启 |
| 通配符限定符 | 原生支持`(.)` `(/)`等强大的文件过滤限定符 | 不支持，需结合`find`命令 |
| 别名类型 | 支持普通别名、全局别名、后缀别名 | 仅支持普通别名 |
| 字符串大小写转换 | 内置`${str:l}` `${str:u}`，简洁高效 | 4.0+版本支持`${str,,}` `${str^^}`，语法更繁琐 |
| 数组去重 | `typeset -U arr` 声明后自动去重 | 不原生支持，需额外代码处理 |
| 关联数组遍历 | 支持`for k v in "${(@kv)hash}"`直接遍历键值对 | 只能先遍历key，再通过key取value |
| 循环扩展 | 支持`repeat N`固定次数循环 | 无此语法，需用for/while实现 |
| 管道扩展 | 支持`|&`同时传递stdout和stderr | 需用`2>&1 |`实现 |

## 十四、常用参数扩展修饰符速览
Zsh提供丰富的参数扩展修饰符，用于快速处理变量内容，无需外部命令：
| 修饰符 | 作用 | 示例 |
|--------|------|------|
| `${(j:,:)arr}` | 用逗号拼接数组元素为字符串 | `arr=(a b c); ${(j:,:)arr}` → `a,b,c` |
| `${(s:,:)str}` | 用逗号分割字符串为数组 | `str="a,b,c"; ${(s:,:)str}` → `(a b c)` |
| `${(t)var}` | 显示变量的类型和属性 | `${(t)user_map}` → `association` |
| `${(r)arr}` | 反转数组顺序 | |
| `${(o)arr}` | 对数组进行升序排序 | |
| `${(O)arr}` | 对数组进行降序排序 | |
| `${(n)arr}` | 按数字自然排序 | |


