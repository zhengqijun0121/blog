# Bash 语法详细总结


# Bash 语法详细总结

Bash（Bourne-Again Shell）是 Linux/Unix 系统默认的命令行解释器，也是 Shell 脚本开发的核心标准。本文从基础到进阶，全面覆盖 Bash 核心语法、特性、最佳实践与避坑指南。

## 一、脚本基础结构与执行
### 1. 脚本核心声明
脚本第一行必须指定解释器（Shebang 行），用于告诉系统用哪个 Shell 执行脚本：
```bash
#!/bin/bash
# 可选：开启严格模式，大幅提升脚本健壮性（推荐）
set -euo pipefail
```
- `set -e`：命令执行失败（退出码非0）时立即退出脚本，避免错误扩散
- `set -u`：使用未定义变量时直接报错，防止变量名写错导致的异常
- `set -o pipefail`：管道中任意命令失败，整个管道的退出码即为失败（默认仅取最后一条命令的退出码）

### 2. 脚本执行方式
| 执行方式 | 执行环境 | 权限要求 | 特点 |
|----------|----------|----------|------|
| `./script.sh` | 子Shell | 需要执行权限`chmod +x script.sh` | 标准执行方式，不影响当前Shell环境 |
| `bash script.sh` | 子Shell | 无需执行权限 | 可加调试参数，如`bash -x script.sh` |
| `source script.sh` / `. script.sh` | 当前Shell | 无需执行权限 | 脚本内的变量、函数会污染当前Shell，常用于加载环境变量 |

## 二、变量与参数
Bash 变量默认均为字符串类型，无需提前声明，赋值时`=`两边**绝对不能有空格**。

### 1. 变量定义与引用
```bash
# 基础定义
name="张三"
age=20
# 引用变量：$var 或 ${var}（花括号用于明确变量边界）
echo "姓名：$name，年龄：${age}岁"
# 带空格/特殊字符的内容必须加双引号
file_name="hello world.txt"
```

### 2. 变量类型
#### （1）自定义局部变量
默认仅在当前 Shell 生效，函数内用`local`声明的变量仅在函数内生效，避免全局污染。
```bash
test_func() {
    local local_var="我是局部变量"
    global_var="我是全局变量"
}
test_func
echo $global_var  # 正常输出
echo $local_var   # 空值，无法访问
```

#### （2）环境变量（全局变量）
通过`export`声明，当前 Shell 及其所有子 Shell 均可访问，系统内置常用环境变量如下：
| 变量 | 含义 |
|------|------|
| `$HOME` | 当前用户家目录 |
| `$PATH` | 命令搜索路径，冒号分隔 |
| `$USER` / `$UID` | 当前用户名/用户ID |
| `$PWD` | 当前工作目录 |
| `$SHELL` | 当前默认Shell解释器 |
| `$LANG` | 系统语言编码 |
| `$HOSTNAME` | 主机名 |

#### （3）特殊内置变量（高频使用）
| 变量 | 核心含义 |
|------|----------|
| `$0` | 脚本本身的文件名 |
| `$1`~`$n` | 脚本/函数的第1~n个位置参数，n>9时必须用`${10}`格式 |
| `$#` | 位置参数的总个数 |
| `$@` | 所有位置参数，`"$@"`会保留每个参数的独立边界（循环遍历推荐） |
| `$*` | 所有位置参数，`"$*"`会将所有参数合并为一个整体（以IFS分隔） |
| `$?` | 上一条命令的退出状态码（0=成功，非0=失败） |
| `$$` | 当前Shell进程的PID |
| `$!` | 后台运行的最后一个进程的PID |
| `$_` | 上一条命令的最后一个参数 |

### 3. 变量默认值与替换
用于处理变量为空/未定义的场景，是脚本健壮性的核心技巧：
| 表达式 | 执行逻辑 |
|--------|----------|
| `${var:-default}` | var为空/未定义，返回default，var本身不变 |
| `${var:=default}` | var为空/未定义，返回default，同时给var赋值default |
| `${var:+replace}` | var不为空，返回replace，var本身不变 |
| `${var:?error_msg}` | var为空/未定义，输出error_msg并强制退出脚本（必填参数校验） |

示例：
```bash
# 若用户未输入name，默认值为"未知"
username=${1:-未知}
# 若path未定义，强制退出并报错
path=${path:?必须指定path参数}
```

## 三、运算符与条件测试
### 1. 算术运算符
Bash 仅支持整数运算，不支持浮点数（浮点数需用`bc`/`awk`实现），核心运算方式如下：

#### （1）推荐用法：`$(( 表达式 ))`
无需转义，支持所有标准算术运算，是最通用的写法：
```bash
a=10
b=3
echo $((a + b))  # 加 13
echo $((a - b))  # 减 7
echo $((a * b))  # 乘 30
echo $((a / b))  # 整除 3
echo $((a % b))  # 取余 1
echo $((a ** b)) # 幂运算 1000
echo $((a++))    # 自增（后置）
echo $((++a))    # 自增（前置）
# 三元运算符
echo $(( a > b ? 100 : 200 )) # 100
```

#### （2）其他运算方式
- `let`命令：`let a=1+2; let a++`
- `$[]`：与`$(( ))`类似，兼容性稍差，如`a=$[1+2]`
- `expr`：老旧写法，需严格空格、特殊字符转义，不推荐，如`expr 1 + 2`

### 2. 条件测试语法
Bash 提供3种条件测试方式，优先级与功能：`(())` > `[[]]` > `[]`

#### （1）`[ ]`（test 命令）
POSIX 标准兼容，所有 Shell 均支持，功能最基础，注意：**括号内首尾必须加空格**。
```bash
# 正确写法
[ -f "test.txt" ]
# 错误写法（无空格）
[-f "test.txt"]
```

#### （2）`[[ ]]`（Bash 关键字）
Bash 专属扩展，功能更强大，无分词风险，支持正则匹配、通配符匹配，推荐日常使用。
```bash
# 无需引号也不会因空格分词，支持&&/||直接写在括号内
[[ $name == "张三" && $age -gt 18 ]]
# 通配符匹配（判断是否以test开头）
[[ $file_name == test* ]]
# 正则匹配（判断是否为纯数字）
[[ $num =~ ^[0-9]+$ ]]
```

#### （3）`(( ))`（算术测试）
专门用于数字比较与运算，支持`> < >= <= == !=`等常规符号，无需转义，数字场景优先使用。
```bash
a=20
if (( a > 18 && a < 30 )); then
    echo "符合年龄要求"
fi
```

### 3. 核心测试运算符
#### （1）数字比较运算符
| `[ ]`/`[[ ]]` 写法 | `(())` 写法 | 含义 |
|---------------------|-------------|------|
| `-eq` | `==` | 等于（equal） |
| `-ne` | `!=` | 不等于（not equal） |
| `-gt` | `>` | 大于（greater than） |
| `-ge` | `>=` | 大于等于（greater or equal） |
| `-lt` | `<` | 小于（less than） |
| `-le` | `<=` | 小于等于（less or equal） |

#### （2）字符串运算符
| 运算符 | 含义 |
|--------|------|
| `==` / `=` | 字符串相等（`=`是POSIX标准，`==`是Bash扩展） |
| `!=` | 字符串不相等 |
| `-z` | 字符串长度为0（zero） |
| `-n` | 字符串长度不为0（not zero） |
| `$str` | 字符串非空则为真 |

#### （3）文件测试运算符（高频使用）
用于判断文件/目录的属性与权限，是脚本最常用的功能之一：
| 运算符 | 核心含义 |
|--------|----------|
| `-e` | 文件/目录是否存在（exist） |
| `-f` | 是否为普通文件（非目录/设备） |
| `-d` | 是否为目录（directory） |
| `-s` | 文件是否非空（大小>0） |
| `-r` / `-w` / `-x` | 是否有读/写/执行权限 |
| `-L` | 是否为软链接（link） |
| `-nt` | 前者比后者更新（newer than） |
| `-ot` | 前者比后者更旧（older than） |
| `-ef` | 两个文件是否为同一个硬链接 |

#### （4）逻辑运算符
| 运算符 | 含义 | 适用场景 |
|--------|------|----------|
| `&&` | 逻辑与（前真后才执行） | `[[ ]]`、`(())`、命令行 |
| `||` | 逻辑或（前假后才执行） | `[[ ]]`、`(())`、命令行 |
| `!` | 逻辑非（取反） | 所有场景 |
| `-a` | 逻辑与 | 仅`[ ]`内使用 |
| `-o` | 逻辑或 | 仅`[ ]`内使用 |

示例：
```bash
# 命令行短路逻辑（常用）
# 前一条成功才执行后一条
cd /data && ls
# 前一条失败才执行后一条
[ -d test ] || mkdir test

# 多条件组合
[[ -f "test.txt" && -r "test.txt" ]] && echo "文件存在且可读"
```

## 四、流程控制语句
### 1. if 条件判断
```bash
# 单分支
if 条件; then
    执行命令
fi

# 双分支
if 条件; then
    执行命令1
else
    执行命令2
fi

# 多分支
if 条件1; then
    执行命令1
elif 条件2; then
    执行命令2
else
    执行命令3
fi
```

示例：
```bash
read -p "请输入你的年龄：" age
if (( age < 18 )); then
    echo "未成年"
elif (( age < 60 )); then
    echo "成年"
else
    echo "老年"
fi
```

### 2. for 循环
#### （1）for-in 循环（遍历列表）
```bash
for 变量 in 取值列表; do
    执行命令
done
```

常用示例：
```bash
# 遍历数字序列（{1..10} 生成1-10，{1..10..2} 步长2）
for i in {1..5}; do
    echo "序号：$i"
done

# 遍历所有txt文件
for file in *.txt; do
    echo "处理文件：$file"
done

# 遍历脚本入参（推荐"$@"，保留参数边界）
for arg in "$@"; do
    echo "参数：$arg"
done
```

#### （2）C语言风格 for 循环
```bash
for (( 初始化; 循环条件; 增量 )); do
    执行命令
done
```

示例：
```bash
for ((i=1; i<=5; i++)); do
    echo "序号：$i"
done
```

### 3. while / until 循环
#### （1）while 循环（条件为真时执行）
```bash
while 条件; do
    执行命令
done
```

高频场景：
```bash
# 计数循环
i=1
while (( i <= 5 )); do
    echo "序号：$i"
    ((i++))
done

# 逐行读取文件（标准写法，-r 防止反斜杠转义）
while read -r line; do
    echo "行内容：$line"
done < test.txt

# 无限循环
while :; do
    echo "循环执行"
    sleep 1
done
```

#### （2）until 循环（条件为假时执行，直到条件为真退出）
```bash
until 条件; do
    执行命令
done
```

示例：
```bash
i=1
until (( i > 5 )); do
    echo "序号：$i"
    ((i++))
done
```

### 4. case 分支选择
替代多分支 if，更简洁，支持通配符匹配，适合多选项场景：
```bash
case 变量 in
    模式1)
        执行命令1
        ;;
    模式2|模式3) # | 表示多模式匹配（或）
        执行命令2
        ;;
    *) # 通配符，匹配所有剩余情况，相当于default
        默认执行命令
        ;;
esac
```

示例：
```bash
read -p "请输入操作[start/stop/restart]：" opt
case $opt in
    start)
        echo "启动服务"
        ;;
    stop)
        echo "停止服务"
        ;;
    restart)
        echo "重启服务"
        ;;
    *)
        echo "无效操作"
        exit 1
        ;;
esac
```

### 5. 循环控制
- `break n`：跳出n层循环，默认n=1，直接终止循环
- `continue n`：跳过当前循环剩余代码，进入下n层循环的下一次迭代，默认n=1

## 五、函数
Bash 函数必须**先定义后调用**，无需声明参数类型，入参通过位置参数`$1/$2...`获取。

### 1. 函数定义与调用
```bash
# 标准写法（推荐）
function 函数名() {
    # 函数体
    执行命令
    # 可选：return 返回退出码（0-255，默认是最后一条命令的退出码）
    return 0
}

# 简化写法（省略function）
函数名() {
    执行命令
}

# 函数调用：直接写函数名，无需加()
函数名 参数1 参数2
```

### 2. 核心特性与示例
1.  **入参与返回值**：`return`仅能返回0-255的整数，需返回字符串/大数字时，用`echo`输出，通过`$()`接收。
2.  **局部变量**：函数内用`local`声明变量，避免污染全局环境。

示例：
```bash
# 加法函数
add() {
    # 声明局部变量
    local a=$1
    local b=$2
    # 输出结果，用于外部接收
    echo $((a + b))
}

# 调用函数，接收返回值
sum=$(add 10 20)
echo "求和结果：$sum" # 输出30

# 带返回码的函数
check_file() {
    local file=$1
    if [[ -f $file ]]; then
        echo "文件存在"
        return 0
    else
        echo "文件不存在"
        return 1
    fi
}

# 根据返回码判断执行结果
if check_file "test.txt"; then
    echo "后续处理"
fi
```

## 六、数组
Bash 支持**索引数组**（下标从0开始）和**关联数组**（键值对，Bash 4.0+支持）。

### 1. 索引数组
#### （1）数组定义
```bash
# 方式1：一次性赋值
arr=(apple banana orange "hello world")

# 方式2：单个元素赋值
arr[0]=apple
arr[1]=banana

# 方式3：带下标赋值
arr=([0]=apple [2]=orange [1]=banana)
```

#### （2）数组核心操作
```bash
# 访问单个元素
echo ${arr[0]}

# 访问所有元素（推荐"${arr[@]}"，保留元素边界）
echo "${arr[@]}"

# 获取数组长度（元素个数）
echo ${#arr[@]}

# 数组切片：${数组[@]:起始下标:长度}
echo ${arr[@]:1:2} # 从下标1开始，取2个元素

# 追加元素（不覆盖原有内容）
arr+=(grape pear)

# 删除单个元素
unset arr[1]

# 删除整个数组
unset arr
```

#### （3）数组遍历
```bash
# 方式1：遍历元素
for item in "${arr[@]}"; do
    echo "元素：$item"
done

# 方式2：遍历下标
for i in "${!arr[@]}"; do
    echo "下标$i：${arr[$i]}"
done
```

### 2. 关联数组（键值对）
必须先通过`declare -A`声明，仅 Bash 4.0+ 版本支持，适合存储映射关系。
```bash
# 声明关联数组
declare -A user

# 赋值
user[name]="张三"
user[age]=20
user[gender]="男"

# 一次性赋值
declare -A user=(
    [name]="张三"
    [age]=20
    [gender]="男"
)

# 核心操作
echo ${user[name]} # 访问单个值
echo "${!user[@]}" # 获取所有键
echo "${user[@]}"  # 获取所有值
echo ${#user[@]}   # 获取键值对个数

# 遍历关联数组（必须遍历键）
for key in "${!user[@]}"; do
    echo "$key：${user[$key]}"
done

# 删除元素
unset user[age]
```

## 七、字符串处理（内置操作）
Bash 内置丰富的字符串操作，无需调用外部命令，效率极高，是脚本开发的核心技能。

### 1. 基础操作
```bash
str="hello world"

# 获取字符串长度
echo ${#str} # 输出11

# 字符串截取
# 格式：${str:起始位置:长度}，起始位置从0开始，长度省略则截取到末尾
echo ${str:6:5}  # 从第6位开始，截取5个字符，输出world
echo ${str:6}     # 从第6位截取到末尾，输出world

# 从右往左截取（0- 不可省略）
echo ${str:0-5:3} # 从倒数第5位开始，截取3个字符，输出wor
echo ${str:0-5}    # 截取最后5个字符，输出world
```

### 2. 字符串替换
```bash
str="hello world hello"

# 替换第一个匹配的子串
echo ${str/hello/hi} # 输出hi world hello

# 替换所有匹配的子串
echo ${str//hello/hi} # 输出hi world hi

# 替换开头匹配的子串
echo ${str/#hello/hi} # 输出hi world hello

# 替换结尾匹配的子串
echo ${str/%hello/hi} # 输出hello world hi

# 删除匹配的子串（新子串留空）
echo ${str//hello/} # 输出 world
```

### 3. 字符串删除（路径处理高频使用）
```bash
path="/home/user/test.txt"

# 从开头删除最短匹配
echo ${path#*/} # 输出home/user/test.txt（删除第一个/及之前）

# 从开头删除最长匹配
echo ${path##*/} # 输出test.txt（删除最后一个/及之前，提取文件名）

# 从结尾删除最短匹配
echo ${path%/*} # 输出/home/user（删除最后一个/及之后，提取目录路径）

# 从结尾删除最长匹配
file="test.txt.bak"
echo ${file%%.*} # 输出test（删除第一个.及之后，提取文件名前缀）
```

### 4. 大小写转换（Bash 4.0+ 支持）
```bash
str="Hello World"

# 转全小写
echo ${str,,} # 输出hello world

# 转全大写
echo ${str^^} # 输出HELLO WORLD

# 首字母小写
echo ${str,} # 输出hello World

# 首字母大写
echo ${str^} # 输出Hello World
```

## 八、重定向与管道
重定向与管道是 Shell 的核心特性，用于控制命令的输入输出流向。

### 1. 标准文件描述符
| 文件描述符 | 名称 | 缩写 | 默认流向 |
|------------|------|------|----------|
| 0 | 标准输入 | stdin | 键盘 |
| 1 | 标准输出 | stdout | 终端屏幕 |
| 2 | 标准错误 | stderr | 终端屏幕 |

### 2. 输出重定向
| 语法 | 核心功能 |
|------|----------|
| `command > file` | 把stdout**覆盖**写入file，等价于`command 1> file` |
| `command >> file` | 把stdout**追加**写入file |
| `command 2> file` | 把stderr**覆盖**写入file |
| `command 2>> file` | 把stderr**追加**写入file |
| `command &> file` | 把stdout和stderr都**覆盖**写入file，等价于`command > file 2>&1` |
| `command &>> file` | 把stdout和stderr都**追加**写入file，等价于`command >> file 2>&1` |
| `command > /dev/null` | 丢弃stdout输出 |
| `command 2> /dev/null` | 丢弃stderr输出 |
| `command &> /dev/null` | 丢弃所有输出 |

核心技巧：`2>&1` 表示把stderr重定向到stdout的位置，是最常用的重定向组合。
```bash
# 把正常输出和错误都写入log.txt
./script.sh > log.txt 2>&1
```

### 3. 输入重定向
| 语法 | 核心功能 |
|------|----------|
| `command < file` | 从file读取内容作为command的stdin |
| `command << EOF` | Here Document，嵌入多行输入，直到遇到EOF结束符 |
| `command <<< "字符串"` | Here String，把单行字符串作为command的stdin |

示例：
```bash
# 从文件读取内容统计行数
wc -l < test.txt

# Here Document 写入多行内容到文件
cat > test.txt << EOF
第一行内容
第二行内容
第三行内容
EOF

# Here String 匹配字符串
grep "hello" <<< "hello world"
```

### 4. 管道 |
把前一个命令的stdout作为后一个命令的stdin，实现多命令流式处理，格式：
```bash
command1 | command2 | command3
```

示例：
```bash
# 统计当前目录下txt文件的个数
ls *.txt | wc -l

# 过滤包含error的日志行，统计个数
cat app.log | grep "error" | wc -l
```

核心注意点：
1.  管道默认仅传递stdout，不传递stderr，需传递用`|&`（Bash 4.0+），等价于`2>&1 |`
2.  管道中的每个命令都在**子Shell**中执行，循环内修改的变量在循环外不生效，解决方案：用输入重定向替代管道。

反例（变量不生效）：
```bash
count=0
cat test.txt | while read -r line; do
    ((count++))
done
echo $count # 输出0，子Shell修改不影响父Shell
```

正例（当前Shell执行）：
```bash
count=0
while read -r line; do
    ((count++))
done < test.txt
echo $count # 输出正确的行数
```

## 九、通配符与正则表达式
### 1. 通配符（Globbing）
由 Shell 直接解析，用于文件名匹配，**不是正则表达式**，核心符号如下：
| 通配符 | 功能 |
|--------|------|
| `*` | 匹配任意长度的任意字符（包括空字符），如`*.txt`匹配所有txt文件 |
| `?` | 匹配单个任意字符，如`a?.txt`匹配a1.txt、ab.txt |
| `[]` | 匹配括号内的任意单个字符，如`[0-9]`匹配数字，`[abc]`匹配a/b/c，`[^abc]`/`[!abc]`匹配非a/b/c的字符 |
| `{}` | 大括号扩展，生成序列，如`{1..10}`生成1-10，`file{1..3}.txt`生成file1.txt/file2.txt/file3.txt，`{a,b,c}`生成a b c |

### 2. 正则表达式
Bash 中主要用于`[[ ]]`的`=~`运算符，以及`grep/sed/awk`等命令，遵循 POSIX 扩展正则（ERE）标准，核心元字符如下：
| 元字符 | 功能 |
|--------|------|
| `^` | 匹配行开头 |
| `$` | 匹配行结尾 |
| `.` | 匹配单个任意字符 |
| `*` | 匹配前一个字符0次或多次 |
| `+` | 匹配前一个字符1次或多次 |
| `?` | 匹配前一个字符0次或1次 |
| `[]` | 匹配括号内的单个字符，`[^]`匹配非括号内的字符 |
| `()` | 分组，`\1`/`\2`引用分组内容 |
| `|` | 逻辑或 |
| `{n,m}` | 匹配前一个字符n到m次 |

示例（正则匹配）：
```bash
# 判断是否为纯数字
num="123456"
if [[ $num =~ ^[0-9]+$ ]]; then
    echo "是纯数字"
fi

# 判断是否为手机号
phone="13812345678"
if [[ $phone =~ ^1[3-9][0-9]{9}$ ]]; then
    echo "是合法手机号"
fi
```

## 十、常用内置命令
内置命令是 Bash 自带的命令，无需启动子进程，执行效率远高于外部命令，高频内置命令如下：

| 命令 | 核心功能与常用参数 |
|------|---------------------|
| `echo` | 输出字符串，`-n`不换行，`-e`解析转义字符（`\n`/`\t`） |
| `printf` | 格式化输出，与C语言printf一致，跨平台兼容性优于echo，如`printf "姓名：%s，年龄：%d\n" "张三" 20` |
| `read` | 读取用户输入/文件内容，`-p`指定提示语，`-r`不转义反斜杠，`-s`静默模式（输入不显示），`-t`超时时间，`-a`读取到数组 |
| `declare` | 声明变量/数组，`-i`整数，`-r`只读，`-a`索引数组，`-A`关联数组，`-x`环境变量 |
| `export` | 声明环境变量，`export var=value` |
| `readonly` | 声明只读变量，不可修改/删除 |
| `unset` | 删除变量/数组/函数，`unset var`，`unset arr[1]` |
| `shift` | 左移位置参数，`shift n`默认n=1，用于批量处理入参 |
| `source`/`.` | 在当前Shell执行脚本，用于加载环境变量/函数 |
| `exit` | 退出脚本，`exit n`指定退出码（0=成功，非0=失败） |
| `return` | 函数内返回退出码，`return n`（0-255） |
| `exec` | 替换当前Shell进程，或永久重定向文件描述符，如`exec &> log.log` |
| `set` | 设置Shell选项，`set -e/-u/-x/-o pipefail` |
| `shopt` | 设置Bash扩展选项，`shopt -s extglob`开启扩展通配符 |

## 十一、脚本调试与最佳实践
### 1. 脚本调试方法
1.  **语法检查**：`bash -n script.sh`，仅检查语法错误，不执行脚本
2.  **调试模式**：`bash -x script.sh`，执行时打印每一行命令与参数，定位问题
3.  **局部调试**：脚本内用`set -x`开启调试，`set +x`关闭调试，仅调试指定代码块
4.  **详细模式**：`bash -v script.sh`，执行前打印脚本的每一行内容

### 2. 脚本编写最佳实践
1.  开头必须加`#!/bin/bash`，推荐开启严格模式`set -euo pipefail`
2.  变量命名有意义，环境变量全大写，自定义变量小写+下划线，避免无意义的`a/b`命名
3.  变量引用必须加双引号`"$var"`，避免空格分词导致的异常
4.  函数内变量必须用`local`声明，避免全局环境污染
5.  用`$()`替代反引号`` ` ``执行命令，支持嵌套，可读性更强
6.  代码统一缩进（4个空格/tab），关键逻辑加注释，提升可维护性
7.  处理文件名/路径时，用`--`避免文件名以`-`开头的问题，如`rm -- "$file"`
8.  尽量使用 Bash 内置操作，减少`sed/awk/cut`等外部命令调用，提升执行效率

### 3. 新手高频踩坑点
1.  赋值`=`两边加空格，导致命令执行报错
2.  `[ ]`条件测试时，括号首尾不加空格
3.  变量引用不加双引号，导致带空格的文件名/参数被拆分
4.  管道内循环修改变量，循环外不生效
5.  混淆`$*`和`$@`，遍历入参时未用`"$@"`导致参数拆分
6.  位置参数n>9时，未用`${10}`格式，导致被解析为`$1`拼接`0`
7.  混淆通配符与正则表达式，在`[ ]`中使用通配符匹配字符串
8.  用`>`重定向时，误覆盖已有文件
9.  函数内未用`local`声明变量，导致全局变量被意外修改

