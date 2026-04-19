# awk 命令使用详细总结


# awk 命令使用详细总结

awk 是 Unix/Linux 生态中**模式驱动的文本处理编程语言**，由 Alfred Aho、Peter Weinberger、Brian Kernighan 三位开发者创造，是 Shell 三剑客（grep/sed/awk）中功能最全面的工具，核心用于文本扫描、字段提取、数据统计、格式化输出，原生支持变量、数组、循环、函数等完整编程能力，是运维、开发人员的核心文本处理工具。

Linux 系统中默认的 `awk` 是 GNU awk（gawk），是标准 awk 的超集，提供了大量扩展功能，本文默认基于 gawk 讲解，同时标注 POSIX 标准兼容内容。

---

## 一、核心语法与执行流程

### 1. 基础命令格式

```Bash

# 单行命令模式（最常用）
awk [选项] '模式{动作}' 输入文件1 输入文件2...

# 脚本文件模式（适合复杂逻辑）
awk [选项] -f 脚本文件.awk 输入文件
```

- **模式（Pattern）**：筛选条件，只有匹配模式的行才会执行对应的动作，省略则匹配所有行

- **动作（Action）**：对匹配行执行的代码，用 `{}` 包裹，省略则默认执行 `{print $0}`（打印整行）

- 输入文件省略时，默认从标准输入（stdin）读取数据

### 2. 脚本三大核心块

一个完整的 awk 脚本由三个可选块组成，分工明确：

```Bash

awk '
BEGIN { 预处理动作 }       # 1. 读取输入前执行，仅执行1次
模式{ 逐行处理动作 }       # 2. 逐行读取输入，匹配模式则执行，循环执行
END { 收尾汇总动作 }       # 3. 所有输入读取完成后执行，仅执行1次
' 输入文件
```

- **BEGIN 块**：常用于初始化变量、设置分隔符、打印表头、加载配置

- **主体块**：核心处理逻辑，逐行扫描文本的核心循环

- **END 块**：常用于统计结果输出、数据汇总、格式化报表

### 3. 完整执行流程

1. 解析命令行选项与参数

2. 执行 BEGIN 块中的所有代码

3. 循环执行：读取一行输入 → 拆分字段 → 匹配模式 → 执行对应动作，直到输入读取完毕

4. 执行 END 块中的所有代码

5. 退出程序，返回退出码

### 4. 高频命令行选项

|选项|功能说明|典型示例|
|---|---|---|
|`-F 分隔符`|指定输入字段分隔符，支持字符串/正则，等价于内置变量 `FS`|`awk -F: '{print $1}' /etc/passwd`（冒号分隔）|
|`-v 变量名=值`|执行脚本前定义/传递变量，支持外部 Shell 变量传入|`awk -v num=10 'BEGIN{print num+5}'`|
|`-f 脚本文件`|从指定文件读取 awk 脚本，适合复杂逻辑封装|`awk -f handle.awk data.txt`|
|`-W posix`|严格遵循 POSIX 标准，禁用 gawk 扩展特性|`awk -W posix '{print $1}' file`|
|`-W compat`|兼容传统 awk 语法，禁用 gawk 扩展|`awk -W compat '{print $1}' file`|
---

## 二、核心内置变量

awk 提供了丰富的内置变量，无需声明即可直接使用，按用途分为以下几类：

### 1. 记录（行）相关变量

|变量|功能说明|核心用法|
|---|---|---|
|`$0`|当前读取的整行完整内容|`awk '{print $0}' file` 打印整行|
|`NR`|Number of Record，已读取的总记录数（全局行号），多文件连续计数|`awk 'NR==5' file` 打印第5行|
|`FNR`|File Number of Record，当前文件的记录数（文件内行号），多文件单独计数|`awk 'FNR==1{print FILENAME}' file1 file2` 打印每个文件名|
|`RS`|Record Separator，输入记录分隔符，**默认值为换行符 ** **`\n`**|`awk 'BEGIN{RS=""} {print $1}' file` 按空行分隔段落|
|`ORS`|Output Record Separator，输出记录分隔符，**默认值为换行符 ** **`\n`**|`awk 'BEGIN{ORS=" "} {print $0}' file` 将所有行合并为一行|
### 2. 字段（列）相关变量

|变量|功能说明|核心用法|
|---|---|---|
|`$1~$n`|当前行的第 n 个字段，由 `FS` 拆分|`awk '{print $1, $3}' file` 打印第1、3列|
|`NF`|Number of Fields，当前行的字段总数（列数）|`awk '{print $NF}' file` 打印最后一列；`$(NF-1)` 打印倒数第二列|
|`FS`|Field Separator，输入字段分隔符，**默认值为连续空白符（空格/制表符/换行符）**|`awk 'BEGIN{FS=":"} {print $1}' /etc/passwd`|
|`OFS`|Output Field Separator，输出字段分隔符，**默认值为单个空格**|`awk 'BEGIN{OFS=","} {print $1,$3}' file` 输出用逗号分隔|
### 3. 文件与参数相关变量

|变量|功能说明|核心用法|
|---|---|---|
|`FILENAME`|当前正在处理的输入文件名|`awk '{print FILENAME, FNR, $0}' file1 file2`|
|`ARGC`|命令行参数的总个数|`awk 'END{print ARGC}' file1 file2`|
|`ARGV`|命令行参数数组，`ARGV[0]` 固定为 `awk`，`ARGV[1]` 开始为输入文件/参数|`awk 'BEGIN{print ARGV[1]}' file1`|
|`ENVIRON`|系统环境变量的关联数组|`awk 'BEGIN{print ENVIRON["HOME"]}'` 打印用户家目录|
### 4. 正则与格式相关变量

|变量|功能说明|
|---|---|
|`RSTART`|`match()` 函数匹配到的字符串的起始位置，匹配失败为0|
|`RLENGTH`|`match()` 函数匹配到的字符串的长度，匹配失败为-1|
|`SUBSEP`|多维数组下标的分隔符，默认值为不可打印字符 `\034`|
|`OFMT`|数字的默认输出格式，默认值为 `%.6g`|
|`CONVFMT`|数字转字符串的默认格式，默认值为 `%.6g`|
|`IGNORECASE`|gawk 扩展，设置为1时，正则匹配忽略大小写，默认0|
|`FPAT`|gawk 扩展，定义字段的正则匹配规则，替代分隔符模式，适合处理 CSV 等复杂格式|
---

## 三、运算符与模式匹配

### 1. 完整运算符体系（优先级从高到低）

|运算符类型|运算符列表|说明|
|---|---|---|
|字段引用|`$`|字段引用，如 `$1`、`$NF`|
|自增自减|`++` `--`|前缀/后缀自增、自减，同C语言|
|幂运算|`^` `**`|幂运算，`2^3=8`，`**` 为 gawk 扩展|
|算术运算|`*` `/` `%`|乘、除、取模|
||`+` `-`|加、减|
|字符串拼接|空格|隐式拼接，`"a" "b"` 等价于 `"ab"`|
|关系运算|`>` `<` `>=` `<=`|数值/字符串比较|
||`==` `!=`|精确等于、不等于|
||`~` `!~`|正则匹配、正则不匹配（核心）|
|逻辑运算|`!`|逻辑非|
||`&&`|逻辑与（短路）|
||`||
|三元运算|`?:`|`条件?表达式1:表达式2`，同C语言|
|赋值运算|`=` `+=` `-=` `*=` `/=` `%=` `^=`|赋值与复合赋值|
|数组成员|`in`|判断下标是否存在于数组中|
### 2. 核心模式类型

awk 是模式驱动的语言，模式决定了动作是否执行，支持以下6种模式：

#### （1）空模式

省略模式，匹配所有输入行，每行都会执行动作，是最基础的用法：

```Bash

awk '{print $1}' file  # 所有行都打印第1列
```

#### （2）正则表达式模式

格式：`/正则表达式/`，匹配内容符合正则的行，支持 POSIX 正则规范：

```Bash

awk '/error/' file  # 打印包含 error 的行
awk '$1 ~ /^root/' /etc/passwd  # 第1列以 root 开头的行
awk '$7 !~ /nologin$/' /etc/passwd  # 第7列不以 nologin 结尾的行
```

#### （3）关系表达式模式

用关系运算符构成的条件，条件为真时执行动作：

```Bash

awk 'NR>5 && NR<10' file  # 打印第6-9行
awk -F: '$3>=1000' /etc/passwd  # 打印普通用户（UID>=1000）
awk 'NF==5' file  # 打印字段数为5的行
```

#### （4）范围模式

格式：`模式1,模式2`，从匹配模式1的行开始，到匹配模式2的行结束，中间所有行都执行动作：

```Bash

awk '/start/,/end/' file  # 打印 start 到 end 之间的所有行
awk 'NR==10,NR==20' file  # 打印第10到20行
```

注意：范围模式是贪婪匹配，若没有匹配到模式2，会一直匹配到文件末尾。

#### （5）特殊模式

- `BEGIN`：读取输入前执行，仅执行1次

- `END`：所有输入读取完成后执行，仅执行1次

- `BEGINFILE`/`ENDFILE`：gawk 扩展，处理每个文件的前后分别执行，适合多文件处理的初始化/收尾

#### （6）模式组合

支持用逻辑运算符 `&&` `||` `!` 组合多个模式，实现复杂筛选：

```Bash

awk 'NR>10 && /error/ && $NF>500' file  # 10行后、包含error、最后一列大于500的行
```

---

## 四、流程控制语句

awk 支持类C语言的流程控制语法，可实现复杂的逻辑处理。

### 1. 条件判断

```Bash

# 单分支
if (条件) {
    动作
}

# 双分支
if (条件) {
    动作1
} else {
    动作2
}

# 多分支
if (条件1) {
    动作1
} else if (条件2) {
    动作2
} else {
    动作3
}
```

示例：

```Bash

awk -F: '{
    if($3==0) print "管理员用户："$1;
    else if($3>=1000) print "普通用户："$1;
    else print "系统用户："$1;
}' /etc/passwd
```

### 2. 循环语句

```Bash

# for 循环（固定次数）
for (初始化; 条件; 递增) {
    动作
}

# for 循环（遍历数组）
for (下标 in 数组) {
    动作
}

# while 循环
while (条件) {
    动作
}

# do-while 循环（先执行一次，再判断）
do {
    动作
} while (条件)
```

示例1：遍历字段

```Bash

awk '{for(i=1;i<=NF;i++) print "第"i"列："$i}' file
```

示例2：遍历数组

```Bash

awk '{arr[$1]++} END{for(i in arr) print i, arr[i]}' access.log
```

### 3. 流程控制关键字

|关键字|功能说明|
|---|---|
|`break`|立即跳出当前循环，执行循环后的代码|
|`continue`|跳过本次循环剩余代码，直接进入下一次循环|
|`next`|跳过当前行的所有后续处理，直接读取下一行，进入下一轮主循环|
|`exit n`|立即终止主循环，执行 END 块，n 为退出码（省略则为0）|
示例：

```Bash

awk '/^#/{next} {print $0}' file  # 跳过注释行，打印其他行
awk 'NR>100{exit} {print NR, $0}' file  # 打印前100行后退出
```

---

## 五、内置函数大全

awk 提供了丰富的内置函数，覆盖字符串、数值、时间、IO 等场景，是文本处理的核心能力。

### 1. 高频字符串处理函数

|函数|语法|功能说明|
|---|---|---|
|`length`|`length(str)`|返回字符串 str 的长度，无参数则返回 `$0` 的长度|
|`index`|`index(str, substr)`|返回 substr 在 str 中首次出现的位置，找不到返回0|
|`substr`|`substr(str, start, len)`|从 str 的 start 位置截取 len 个字符，len 省略则截取到末尾|
|`split`|`split(str, arr, sep)`|将 str 按 sep 拆分存入数组 arr，返回字段总数，sep 省略则使用 `FS`|
|`match`|`match(str, regex)`|在 str 中匹配正则 regex，返回匹配起始位置，设置 `RSTART` 和 `RLENGTH`，找不到返回0|
|`sub`|`sub(regex, repl, str)`|替换 str 中**第一个**匹配 regex 的内容为 repl，str 省略则为 `$0`，支持 `&` 引用匹配内容|
|`gsub`|`sub(regex, repl, str)`|全局替换，替换 str 中**所有**匹配 regex 的内容，其他同 `sub`|
|`tolower`|`tolower(str)`|将字符串 str 全部转为小写|
|`toupper`|`toupper(str)`|将字符串 str 全部转为大写|
|`sprintf`|`sprintf(format, args)`|格式化字符串，返回格式化后的结果，用法同C语言 `sprintf`|
示例：

```Bash

# 全局替换
awk '{gsub(/foo/, "bar"); print}' file

# 截取字符串
awk '{print substr($0, 1, 10)}' file  # 截取每行前10个字符

# 字符串拆分
awk '{split($0, arr, ","); print arr[1], arr[3]}' file.csv
```

### 2. 数值计算函数

|函数|功能说明|
|---|---|
|`int(x)`|对 x 取整，向0截断，如 `int(3.9)=3`，`int(-3.9)=-3`|
|`sqrt(x)`|返回 x 的平方根|
|`exp(x)`|返回 e 的 x 次方|
|`log(x)`|返回 x 的自然对数|
|`sin(x)`/`cos(x)`/`atan2(y,x)`|三角函数|
|`rand()`|返回 0~1 之间的随机浮点数|
|`srand(x)`|设置随机数种子，x 省略则使用当前系统时间，返回旧种子|
示例：

```Bash

# 生成0-100的随机整数
awk 'BEGIN{srand(); print int(rand()*100)}'
```

### 3. 时间处理函数（gawk 扩展）

|函数|语法|功能说明|
|---|---|---|
|`systime()`|`systime()`|返回当前时间的 Unix 时间戳（秒级）|
|`strftime`|`strftime(format, timestamp)`|将时间戳格式化为指定格式的字符串，format 同 date 命令|
|`mktime`|`mktime("YYYY MM DD HH MM SS")`|将时间字符串转为 Unix 时间戳|
示例：

```Bash

# 打印当前时间
awk 'BEGIN{print strftime("%Y-%m-%d %H:%M:%S", systime())}'
```

### 4. IO 与系统函数

|函数|功能说明|
|---|---|
|`getline`|读取下一行输入，可赋值给变量，成功返回1，EOF返回0，出错返回-1|
|`close(filename)`|关闭打开的文件/管道，避免 too many open files 错误|
|`system(cmd)`|执行 Shell 命令，返回命令的退出码|
|`fflush()`|刷新输出缓冲区，实时打印内容|
示例：

```Bash

# 执行Shell命令
awk 'BEGIN{system("ls -l")}'

# 读取下一行
awk '{getline next_line; print $0, next_line}' file
```

---

## 六、数组（关联数组）

awk 的数组是**关联数组（哈希表）**，是其核心特色之一，下标可以是数字、字符串，无需提前声明大小，支持动态扩容。

### 1. 数组定义与赋值

```Bash

# 直接赋值
arr["name"] = "zhangsan"
arr[1] = 100
arr["ip_192.168.1.1"] = 5

# 批量赋值（拆分字符串）
split("a,b,c,d", arr, ",")  # arr[1]=a, arr[2]=b, arr[3]=c, arr[4]=d
```

### 2. 数组遍历

awk 数组遍历只能用 `for (下标 in 数组)` 的格式，遍历的是数组的下标，而非值，且遍历顺序不固定：

```Bash

awk '{arr[$1]++} END{
    for (key in arr) {
        print "key:", key, "value:", arr[key]
    }
}' access.log
```

### 3. 成员判断与删除

- **成员判断**：必须用 `if (key in arr)`，避免访问不存在的下标时自动创建元素

- **删除元素**：用 `delete` 关键字

```Bash

# 判断成员是否存在
if ("root" in user_arr) {
    print "root 存在"
}

# 删除单个元素
delete arr["root"]

# 删除整个数组
delete arr
```

### 4. 多维数组模拟

awk 不支持真正的多维数组，通过 `SUBSEP` 分隔符模拟多维数组：

```Bash

# 二维数组赋值
arr["a", "b"] = 100
# 等价于 arr["a" SUBSEP "b"] = 100

# 二维数组遍历
for (key in arr) {
    split(key, idx, SUBSEP)
    print "第一维："idx[1], "第二维："idx[2], "值："arr[key]
}
```

---

## 七、高频实战场景（可直接复制使用）

### 1. 基础列提取与格式化

```Bash

# 提取/etc/passwd的用户名和登录Shell
awk -F: '{print $1, $7}' /etc/passwd

# 格式化输出，左对齐用户名，右对齐UID
awk -F: '{printf "%-20s %5d\n", $1, $3}' /etc/passwd

# 打印最后一列
awk '{print $NF}' file

# 打印除最后一列外的所有列
awk '{$NF=""; print $0}' file
```

### 2. 行过滤与筛选

```Bash

# 打印第5行
awk 'NR==5' file

# 打印前10行（替代head）
awk 'NR<=10' file

# 打印包含error的行（替代grep）
awk '/error/' file

# 打印不包含comment的行（替代grep -v）
awk '!/comment/' file

# 打印空行的行号
awk '/^$/{print NR}' file

# 打印行号大于5且字段数等于6的行
awk 'NR>5 && NF==6' file
```

### 3. 数值统计与计算

```Bash

# 统计文件行数（替代wc -l）
awk 'END{print NR}' file

# 统计第二列的总和
awk '{sum+=$2} END{print "总和：", sum}' file

# 统计第三列的平均值
awk '{sum+=$3; n++} END{print "平均值：", sum/n}' file

# 统计最大值、最小值
awk '
BEGIN{max=0; min=999999}
{
    if($2>max) max=$2;
    if($2<min) min=$2;
}
END{print "最大值：", max, "最小值：", min}
' file
```

### 4. 文本去重与频次统计

```Bash

# 整行去重，保留第一次出现的行（替代uniq，无需排序）
awk '!arr[$0]++' file

# 统计第一列的不重复值出现次数
awk '{arr[$1]++} END{for(i in arr) print i, arr[i]}' file

# 统计Nginx访问日志中IP的访问次数，按次数降序排序
awk '{arr[$1]++} END{for(i in arr) print i, arr[i]}' access.log | sort -rnk2

# 统计HTTP状态码的出现次数
awk '{arr[$9]++} END{for(k in arr) print "状态码："k, "次数："arr[k]}' access.log
```

### 5. 文本替换与修改

```Bash

# 将所有foo替换为bar（替代sed）
awk '{gsub(/foo/, "bar"); print}' file

# 仅在包含error的行中替换foo为bar
awk '/error/{gsub(/foo/, "bar")} {print}' file

# 去掉行首空格
awk '{sub(/^[ \t]+/, ""); print}' file

# 去掉行尾空格
awk '{sub(/[ \t]+$/, ""); print}' file

# 去掉行首行尾空格
awk '{gsub(/^[ \t]+|[ \t]+$/, ""); print}' file
```

### 6. 日志分析实战

```Bash

# 找出Nginx日志中响应时间超过500ms的慢请求
awk '{
    gsub(/cost=|ms/, "", $4);
    if($4 > 500) print "慢请求：" $0
}' access.log

# 打印匹配error的行及其前后2行
awk '{a[NR]=$0} /error/{for(i=NR-2;i<=NR+2;i++) if(i>0) print a[i]}' file

# 统计SSH登录失败的IP地址
awk '/Invalid user/{arr[$10]++} END{for(i in arr) print i, arr[i]}' /var/log/secure
```

### 7. 多文件处理

```Bash

# 分别统计每个文件的行数
awk '
FNR==1{
    if(NR>1) print fname, cnt;
    fname=FILENAME;
    cnt=0;
}
{cnt++}
END{print fname, cnt}
' file1 file2 file3

# 合并两个文件，按关键字匹配
awk -F: 'NR==FNR{a[$1]=$3; next} {print $1, a[$1]}' file1 file2
```

---

## 八、高级用法

### 1. 自定义函数

awk 支持自定义函数，实现代码复用，语法如下：

```Bash

function 函数名(参数列表) {
    函数体
    return 返回值  # 可选
}
```

示例：

```Bash

awk '
# 定义求最大值的函数
function max(a, b) {
    return a > b ? a : b
}
# 主逻辑
{
    print max($1, $2)
}
' file
```

### 2. 复杂格式处理（CSV）

gawk 提供 `FPAT` 变量，可定义字段的正则规则，完美处理带引号、包含分隔符的 CSV 文件：

```Bash

# 处理标准CSV，正确解析带引号的字段
awk '
BEGIN{
    FPAT = "([^,]+)|(\"[^\"]+\")"  # 定义CSV字段规则
    OFS = ","
}
{
    print $1, $3, $5  # 提取第1、3、5列
}
' data.csv
```

### 3. 与Shell交互

#### （1）向awk传递Shell变量

```Bash

# 方法1：-v 选项（推荐，支持特殊字符）
name="root"
awk -v user="$name" -F: '$1==user{print $0}' /etc/passwd

# 方法2：环境变量
export name="root"
awk -F: '$1==ENVIRON["name"]{print $0}' /etc/passwd
```

#### （2）awk中调用Shell命令

```Bash

# 用system()执行命令
awk 'BEGIN{system("echo 当前时间：$(date +%Y-%m-%d)")}'

# 用管道获取命令输出
awk 'BEGIN{
    while("ls -l" | getline line) {
        print line
    }
    close("ls -l")
}'
```

### 4. 大文件处理优化

1. **流式处理**：避免用数组缓存所有行，尽量逐行处理，减少内存占用

2. **提前终止**：满足条件时用 `exit` 提前退出，避免读取整个文件

3. **减少正则**：能用字符串精确匹配（`==`）就不用正则匹配（`~`）

4. **及时关闭文件/管道**：用 `close()` 关闭打开的文件，避免句柄泄漏

5. **前置筛选**：用 `grep` 先过滤核心行，再用 awk 处理，减少 awk 处理的数据量

---

## 九、避坑指南与注意事项

1. **字段分隔符的坑**

    - 默认 `FS` 是**连续空白符**，多个空格/制表符会合并为一个分隔符，行首行尾的空白会被忽略；而 `-F" "` 是单个空格作为分隔符，二者行为不同

    - 特殊字符分隔符（如 `|` `*` `.`）是正则元字符，需要转义，如 `-F"\|"` 或 `-F"[|]"`

    - 多分隔符用正则字符集，如 `-F"[:,;]"` 同时用冒号、逗号、分号分隔

2. **变量与类型的坑**

    - awk 是弱类型语言，字符串和数字自动转换，非数字字符串参与运算时会被视为0，如 `"abc"+123=123`

    - 字符串精确匹配必须用双引号包裹，如 `$1=="root"`，`$1==root` 会把 `root` 视为变量，而非字符串

    - 浮点数运算存在精度问题，如 `0.1+0.2!=0.3`，处理金额时建议放大为整数运算

3. **数组使用的坑**

    - 访问不存在的数组下标时，会**自动创建该下标**，值为空字符串，因此判断成员是否存在必须用 `if (key in arr)`，而非 `if (arr[key]!="")`

    - `for (i in arr)` 遍历的是数组的下标，而非值，且遍历顺序不固定，需要有序输出需自行排序

4. **正则匹配的坑**

    - `~` 是**包含匹配**，`==` 是**精确匹配**，二者不可混用。如 `$1 ~ /root/` 匹配包含 root 的内容，`$1 == "root"` 仅匹配完全等于 root 的内容

    - 正则匹配默认区分大小写，忽略大小写需设置 `IGNORECASE=1`（gawk 扩展）

    - 范围模式 `/start/,/end/` 是贪婪匹配，没有结束标记会匹配到文件末尾

5. **跨平台兼容性**

    - `IGNORECASE`、`FPAT`、`BEGINFILE`、`systime()` 等是 gawk 扩展，POSIX 标准 awk 不支持，跨平台脚本需避免使用

    - 不同系统的 awk 版本差异较大，Linux 用 gawk，macOS 默认用 BSD awk，很多扩展功能不支持

6. **其他常见坑**

    - 空行的 `NF=0`，此时 `$NF` 等价于 `$0`，也就是空行，使用前需判断 `NF>0`

    - awk 脚本放在单引号中，单引号内无法直接使用单引号，需用 `'''` 或 `\x27` 替代

    - `next` 仅跳过当前行的主循环处理，不会跳过 END 块；`exit` 会终止主循环，但仍会执行 END 块
> （注：文档部分内容可能由 AI 生成）
