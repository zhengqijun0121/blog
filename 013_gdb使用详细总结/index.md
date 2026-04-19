# gdb 使用详细总结


# GDB 使用详细总结
GDB（GNU Project Debugger）是GNU开源组织发布的跨平台程序调试器，是Linux/Unix环境下C/C++、Go、Rust等编译型语言的核心调试工具，核心能力包括控制程序执行流程、查看/修改运行时数据、定位崩溃与逻辑bug、事后分析core dump文件，是后端开发、底层开发的必备技能。

## 一、基础入门：环境准备与启动
### 1.1 安装GDB
| 系统环境 | 安装命令 |
|----------|----------|
| Debian/Ubuntu 系列 | `sudo apt update && sudo apt install gdb` |
| CentOS/RHEL/Fedora 系列 | `sudo yum install gdb` 或 `sudo dnf install gdb` |
| macOS | `brew install gdb`（需额外完成代码签名才能attach进程） |
| 源码安装 | 从[GNU官网](sslocal://flow/file_open?url=https%3A%2F%2Fwww.gnu.org%2Fsoftware%2Fgdb%2F&flow_extra=eyJsaW5rX3R5cGUiOiJjb2RlX2ludGVycHJldGVyIn0=)下载源码编译，适配定制化场景 |

安装完成后，通过 `gdb --version` 验证安装是否成功。

### 1.2 调试前必备准备
GDB源码级调试依赖程序中的调试符号，**编译时必须添加 `-g` 参数**，同时建议关闭编译器优化（`-O0`），避免优化导致断点不命中、变量被优化、执行流程错乱的问题。
```bash
# 单文件编译（推荐）
gcc -g -O0 test.c -o test
# C++ 程序编译
g++ -g -O0 test.cpp -o test
# Makefile 项目：在 CFLAGS/CXXFLAGS 中添加 -g -O0
```
- `-g`：生成包含行号、变量类型、函数定义等信息的调试符号
- `-O0`：关闭所有编译器优化，保证执行流程和源码完全一致
- 进阶：线上部署可通过 `strip` 剥离符号减小程序体积，调试时用带符号的备份程序匹配core文件

### 1.3 GDB 4种核心启动方式
| 启动场景 | 命令格式 | 适用场景 |
|----------|----------|----------|
| 直接调试可执行文件 | `gdb <可执行程序>` | 从头启动程序，复现可稳定触发的bug |
| 带启动参数调试 | `gdb --args <可执行程序> <参数1> <参数2>...` | 程序依赖命令行参数启动 |
| attach到运行中进程 | `gdb -p <pid>` 或 `gdb <可执行程序> <pid>` | 调试线上正在运行的服务/进程，无需重启程序 |
| 调试core dump崩溃文件 | `gdb <可执行程序> <core文件>` | 程序崩溃后的事后分析，定位崩溃根因 |

- 附加启动参数：`gdb -q <程序>` 安静模式启动，不打印版本冗余信息；`gdb -tui <程序>` 启动分屏源码界面，调试更直观。

## 二、核心调试命令体系
### 2.1 程序执行控制命令
控制程序的启动、停止、单步执行，是调试的基础操作。

| 命令 | 缩写 | 功能说明与示例 |
|------|------|----------------|
| run | r | 启动程序，从头开始执行；可直接跟启动参数 `r arg1 arg2`；程序已运行时会提示重启 |
| continue | c | 从暂停位置继续执行，直到遇到下一个断点/观察点/程序结束；`c 5` 跳过接下来5个断点 |
| next | n | 单步执行，**不进入函数内部**（将函数调用视为整体执行），俗称“单步跳过”；`n 3` 连续执行3行 |
| step | s | 单步执行，**进入函数内部**（遇到函数调用会进入函数体第一行），俗称“单步进入”；`s 2` 连续单步2次 |
| finish | fin | 执行完当前函数的剩余代码，跳出函数体，停在函数调用的下一行，用于快速跳出误进入的函数 |
| until | u | 执行到指定行号/地址，`u 100` 执行到第100行停下；核心用法是跳出循环，避免逐行循环执行 |
| return | - | 强制当前函数立即返回，可指定返回值 `return 0`，跳过函数剩余代码，用于绕过错误逻辑 |
| kill | k | 终止当前正在调试的程序，不退出GDB，可重新run启动 |
| quit | q | 退出GDB调试器 |
| Ctrl+C | - | 中断正在运行的程序，回到GDB命令行，用于调试卡死/死循环的程序 |

### 2.2 断点、观察点与捕获点管理
断点是控制程序暂停位置的核心，观察点（数据断点）用于监控内存/变量变化，捕获点用于监控特定事件。

#### 2.2.1 断点（Breakpoint，缩写 b）
##### 基础断点设置
```bash
# 按行号设置（当前文件）
b 20
# 按文件+行号设置（多文件项目必备）
b main.c:20
# 按函数名设置
b main
b test_func
# C++ 类成员函数设置
b namespace::class::method
# 按内存地址设置（无源码调试）
b *0x400520
```

##### 进阶断点用法
- **条件断点**：仅当条件满足时断点生效，解决循环内断点的痛点，是高频使用的进阶功能
  ```bash
  # 当i等于100时，第30行断点才触发
  b main.c:30 if i==100
  # 函数入参为NULL时断住，定位空指针崩溃
  b process_data if ptr==NULL
  ```
- **临时断点（tbreak/tb）**：仅生效一次，命中后自动删除，适合单次触发的场景
  ```bash
  tb main.c:50
  ```
- **断点绑定自动命令**：断点命中后自动执行一组命令，无需人工干预
  ```bash
  b main.c:30
  commands 1  # 1为断点编号
  > print i    # 命中后自动打印i的值
  > bt         # 打印调用栈
  > continue   # 自动继续执行
  > end        # 结束命令列表
  ```

##### 断点管理命令
| 命令 | 功能说明 |
|------|----------|
| info breakpoints (info b) | 查看所有断点/观察点/捕获点列表，包含编号、位置、命中次数、启用状态 |
| disable <断点编号> | 禁用指定断点，不填编号则禁用所有 |
| enable <断点编号> | 启用指定断点 |
| delete <断点编号> (d) | 删除指定断点，不填编号则删除所有 |
| clear <位置> | 删除指定位置的断点，示例 `clear main.c:20` |
| ignore <断点编号> <次数> | 忽略断点N次，示例 `ignore 1 10` 1号断点接下来10次命中不触发 |

#### 2.2.2 观察点（Watchpoint，数据断点）
用于定位变量被异常修改、内存被篡改、野指针越界等问题，是调试内存问题的核心神器。

| 命令 | 功能说明与示例 |
|------|----------------|
| watch <变量/表达式> | 写观察点（默认），当变量/表达式的值被修改时，程序立即断住<br>示例：`watch i`、`watch *0x7fffffffde40`（监控指定内存地址） |
| rwatch <变量/表达式> | 读观察点，当变量/表达式被读取时，程序断住 |
| awatch <变量/表达式> | 读写观察点，当变量被读/写时，都会触发断住 |

- 管理：观察点和断点共用 `info b`、`disable`、`enable`、`delete` 命令，在列表中有独立编号。

#### 2.2.3 捕获点（Catchpoint）
捕获特定事件发生时断住程序，常用场景：
```bash
catch throw   # 捕获C++异常抛出时断住
catch catch   # 捕获C++异常被捕获时断住
catch fork    # 捕获程序调用fork系统调用时断住
catch exec    # 捕获程序调用exec系统调用时断住
catch syscall # 捕获所有系统调用时断住
```

### 2.3 数据查看与修改
调试的核心是验证程序运行时数据是否符合预期，GDB提供了丰富的变量、内存、寄存器查看与修改能力。

#### 2.3.1 打印变量/表达式（print，缩写 p）
```bash
# 基础打印
p i             # 打印变量i的值
p &i            # 打印变量i的内存地址
p a+b*c         # 计算并打印表达式结果
p test_func(5)  # 调用函数并打印返回值

# 格式化打印（核心）
p /x i  # 十六进制格式打印
p /d i  # 十进制有符号整数
p /u i  # 十进制无符号整数
p /t i  # 二进制格式
p /c buf[0] # 字符格式
p /f score  # 浮点数格式
p /s buf    # 字符串格式
p /a ptr    # 地址格式
```

- 必备优化：`set print pretty on` 开启后，C/C++结构体、类对象会格式化分层输出，层级清晰，建议写入~/.gdbinit永久生效。
- 宏调试：编译时添加 `-g3` 参数（`gcc -g3 -O0`），可在GDB中打印宏定义、展开宏。

#### 2.3.2 自动显示变量（display，缩写 disp）
程序每次停下时，自动打印指定变量/表达式，适合循环内跟踪变量变化，无需反复执行print命令。
```bash
# 基础用法
display i
display /x ptr
# 管理命令
info display          # 查看所有自动显示条目
undisplay <编号>      # 删除指定条目
disable display <编号> # 禁用自动显示
enable display <编号>  # 启用自动显示
```

#### 2.3.3 内存查看（examine，缩写 x）
查看指定内存地址的内容，定位缓冲区溢出、内存越界、野指针问题必备。
- 格式：`x /<数量><格式符><单位> <内存地址/指针>`
  - 数量：要查看的内存单元个数
  - 格式符：和print的格式符一致（x/d/u/c/s等）
  - 单位：b(1字节)、h(2字节，半字)、w(4字节，字)、g(8字节，双字)

```bash
# 常用示例
x /10xw 0x7fffffffde40  # 从指定地址开始，查看10个4字节单元，十六进制显示
x /20cb buf              # 查看buf指向的20个字节，以字符格式显示
x /s str                 # 查看字符串完整内容
x /i $pc                 # 查看当前程序计数器（rip/eip）指向的汇编指令
```

#### 2.3.4 数据修改
调试时动态修改数据，复现边界场景、绕过错误逻辑、验证修复方案。
```bash
# 修改变量（必须加var关键字，避免和GDB自身命令冲突）
set var i=100
set var ptr=NULL
set var str="test"

# 修改指定内存地址的值
set {int}0x7fffffffde40=200  # 把指定地址的int型数据改为200

# 修改寄存器值
set $rax=0  # 修改rax寄存器的值为0
```

#### 2.3.5 寄存器查看
```bash
info registers (info r)  # 查看所有通用寄存器的值
info all-registers        # 查看所有寄存器（包括浮点、SIMD寄存器）
p $rip                    # 打印指定寄存器的值，$rip是当前指令指针
```

### 2.4 调用栈与函数调试
程序崩溃时，第一时间通过栈回溯定位崩溃位置和调用链路，是排查崩溃问题的核心。

| 命令 | 缩写 | 功能说明与示例 |
|------|------|----------------|
| backtrace | bt | 打印完整的函数调用栈，从栈顶（当前函数）到栈底（main函数），快速定位崩溃位置 |
| bt full | - | 打印完整调用栈+每层栈帧的局部变量、入参，定位问题的核心命令 |
| bt N | - | 只打印栈顶N层调用栈，适合调用栈过长的场景 |
| frame | f | 切换到指定栈帧，`f 3` 切换到第3层栈帧；栈帧编号从0开始（0是当前栈顶） |
| up | - | 向上切换一层栈帧（往函数调用者方向） |
| down | - | 向下切换一层栈帧（往函数被调用者方向） |
| list | l | 查看当前行附近的源码（默认10行）；`l 20` 查看第20行附近；`l test_func` 查看指定函数源码 |
| info locals | - | 查看当前栈帧的所有局部变量 |
| info args | - | 查看当前函数的入参值 |

- 核心流程：程序崩溃后，先执行 `bt full` 找到崩溃栈帧，再用 `f <编号>` 切换到对应栈帧，通过 `list`、`info locals`、`info args` 查看源码和变量，定位根因。

### 2.5 多线程与多进程调试
并发程序调试的核心难点是执行流的管控，GDB提供了完整的线程/进程调试能力。

#### 2.5.1 多线程调试
| 命令 | 功能说明 |
|------|----------|
| info threads | 查看当前进程的所有线程，包含编号、线程ID、执行状态，*号标记当前调试线程 |
| thread <线程编号> | 切换到指定线程，切换后bt、print等命令均作用于该线程 |
| thread apply <编号> <命令> | 对指定线程执行GDB命令，示例：`thread apply 1 3 bt` |
| thread apply all bt | 对所有线程打印调用栈，**死锁排查的核心命令** |
| break <位置> thread <线程编号> | 断点仅对指定线程生效 |
| set scheduler-locking on | 开启线程调度锁定，continue/next/step时**只有当前线程执行**，其他线程暂停，彻底解决多线程调试时断点乱跑的问题 |
| set scheduler-locking off | 关闭调度锁定（默认），所有线程同步执行 |
| set scheduler-locking step | 单步时仅当前线程执行，continue时所有线程执行 |

#### 2.5.2 多进程调试
GDB默认只调试父进程，fork出的子进程不会被跟踪，需通过以下配置调整调试模式：

| 配置命令 | 功能说明 |
|----------|----------|
| set follow-fork-mode parent | 默认模式，fork后继续调试父进程，子进程正常运行 |
| set follow-fork-mode child | fork后切换到子进程调试，父进程正常运行 |
| set detach-on-fork on | 默认模式，fork后只调试一个进程，另一个进程脱离GDB控制正常运行 |
| set detach-on-fork off | fork后，GDB同时跟踪父进程和子进程，未被调试的进程暂停运行 |
| info inferiors | 查看所有被GDB跟踪的进程（inferior） |
| inferior <编号> | 切换到指定进程调试 |
| attach <pid> | 附着到运行中的子进程调试 |
| detach | 分离当前调试的进程，进程恢复正常运行 |

两种模式组合效果：
| follow-fork-mode | detach-on-fork | 效果说明 |
|-------------------|----------------|----------|
| parent | on | 默认模式，只调试父进程，子进程正常运行 |
| child | on | 只调试子进程，父进程正常运行 |
| parent | off | 同时调试父子进程，gdb跟随父进程，子进程阻塞在fork位置 |
| child | off | 同时调试父子进程，gdb跟随子进程，父进程阻塞在fork位置 |

## 三、高级调试场景实战
### 3.1 Core Dump 崩溃调试（线上崩溃排查核心）
Core Dump是程序崩溃时，操作系统把程序的内存、寄存器、运行状态完整转储到文件中，可通过GDB事后分析，精准定位崩溃根因。

#### 3.1.1 前置配置（必须开启，否则不生成core文件）
1. 开启core文件大小限制
```bash
# 临时开启（当前终端生效）
ulimit -c unlimited
# 永久开启：修改 /etc/security/limits.conf，添加以下两行
* soft core unlimited
* hard core unlimited
# 保存后重新登录或执行 sysctl -p 生效
```

2. 配置core文件存储路径与命名（可选，推荐）
```bash
# 修改 /etc/sysctl.conf，添加以下配置
kernel.core_pattern = /var/coredumps/core_%e_%p_%t
kernel.core_uses_pid = 0
# 创建目录并授权
mkdir -p /var/coredumps
chmod 777 /var/coredumps
# 生效配置
sysctl -p
```
- 命名参数：`%e` 程序名，`%p` 进程PID，`%t` 崩溃时间戳，避免core文件被覆盖。

#### 3.1.2 Core文件调试完整流程
1. 启动GDB加载core文件
```bash
gdb <崩溃程序的可执行文件> <core文件>
# 示例
gdb ./test /var/coredumps/core_test_1234_1713000000
```
> 关键要求：可执行文件必须和生成core文件的程序是**同一个编译版本**，且带-g调试符号，否则会出现符号不匹配，无法正常调试。

2. 核心分析步骤
```bash
# 1. 第一步：查看完整调用栈，定位崩溃位置
bt full
# 2. 第二步：切换到崩溃的栈帧（0号栈帧是崩溃发生的函数）
f 0
# 3. 第三步：查看崩溃行的源码、局部变量、入参
list
info locals
info args
# 4. 第四步：查看寄存器状态、崩溃地址的内存，定位空指针/野指针/越界问题
info registers
x /xw $rsp  # 查看栈顶内存
p *ptr      # 查看指针指向的内容
```

3. 自动化批量分析
```bash
# 批量导出崩溃调用栈，无需进入交互界面
gdb -batch -ex "bt full" ./test core_test_1234 > crash_backtrace.txt
```

### 3.2 Attach 运行中进程调试
适用于线上服务卡死、死锁、CPU占用高、内存泄漏等场景，无需重启程序，在线调试不影响业务。

#### 3.2.1 前置条件
- 进程的可执行文件编译时带-g调试符号
- 执行GDB的用户是root或进程所属用户，具备调试权限
- 关闭系统ptrace限制（部分系统默认开启）：
  ```bash
  # 临时关闭
  echo 0 > /proc/sys/kernel/yama/ptrace_scope
  # 永久关闭：修改 /etc/sysctl.d/10-ptrace.conf，设置 kernel.yama.ptrace_scope = 0
  ```

#### 3.2.2 调试完整流程
1. 找到目标进程PID
```bash
ps -ef | grep <进程名>
```

2. 启动GDB attach进程
```bash
# 方式1：直接attach
gdb -p <pid>
# 方式2：启动后attach并加载符号
gdb
(gdb) file <可执行程序路径>
(gdb) attach <pid>
```
> attach成功后，进程会被GDB暂停，此时可执行调试命令。

3. 核心调试操作
```bash
# 死锁排查：查看所有线程的调用栈，找到锁等待的线程
thread apply all bt
# 查看程序运行状态：打印关键变量、全局状态
p global_status
info threads
# 设置断点/观察点，复现问题
b process_data if data==NULL
# 继续执行程序
continue
```

4. 安全退出
```bash
# 先分离进程，让程序恢复正常运行，再退出GDB，禁止直接quit！
detach
quit
```
> 直接quit会终止被调试的进程，线上环境务必先detach再退出。

### 3.3 反汇编与底层调试
适用于无源码调试、编译器优化问题排查、汇编级逻辑分析。
```bash
# 反汇编指定函数
disassemble (disas) main
# 反汇编当前函数
disas
# 反汇编指定地址范围
disas 0x400520,0x400550
# 切换汇编语法为Intel格式（默认AT&T），x86架构推荐
set disassembly-flavor intel
# 查看源码行对应的汇编指令地址范围
info line main.c:20
# 汇编级单步执行
stepi (si)  # 单步执行，进入汇编函数调用
nexti (ni)  # 单步执行，不进入汇编函数调用
# 查看当前执行的汇编指令
x /i $pc
```

### 3.4 GDB自动化与脚本
GDB支持脚本编写，批量执行调试命令，提升重复调试场景的效率。

1. 全局配置文件 `~/.gdbinit`
GDB启动时会自动加载该文件，可写入常用配置、别名、自定义命令，示例：
```bash
# ~/.gdbinit
# 美化结构体打印
set print pretty on
# 切换Intel汇编语法
set disassembly-flavor intel
# 设置list默认显示20行
set listsize 20
# 命令别名
alias ll = list
alias ta = thread apply all
# 开启日志记录
set logging on overwrite gdb.log
```

2. 自定义脚本执行
```bash
# 编写脚本文件 debug.gdb
b main
run
bt full
# 启动时执行脚本
gdb -x debug.gdb ./test
# 调试时加载脚本
source debug.gdb
```

## 四、实用技巧与避坑指南
### 4.1 高频效率快捷键
| 快捷键 | 功能说明 |
|--------|----------|
| 回车 | 重复执行上一条命令，单步调试时无需反复输入n/s |
| Tab | 命令、变量名、函数名自动补全 |
| Ctrl+L | 清屏 |
| Ctrl+D | 退出GDB，等效quit |
| Ctrl+X A | 开启/关闭TUI分屏界面，上半部分显示源码，下半部分是命令行 |

### 4.2 常见问题与避坑方案
| 问题现象 | 根因分析 | 解决方案 |
|----------|----------|----------|
| 断点不生效、无法查看变量 | 1. 编译未加-g参数；2. 开启了-O优化，代码流程/变量被优化 | 重新编译，添加 `-g -O0` 参数，关闭优化 |
| 无法生成core文件 | 1. ulimit -c 为0，未开启core dump；2. 运行目录无写权限；3. 磁盘空间不足；4. 程序设置了setuid/setgid | 按3.1节步骤开启core dump，检查目录权限与磁盘空间 |
| 多线程调试时断点乱跑、程序直接退出 | 默认scheduler-locking off，continue时所有线程同步执行，其他线程触发了信号/退出 | 执行 `set scheduler-locking on`，锁定仅当前线程执行 |
| attach进程失败，提示Operation not permitted | 1. 无进程调试权限；2. 系统开启了ptrace限制；3. macOS下GDB未完成代码签名 | 切换root用户，关闭ptrace限制，macOS完成GDB代码签名 |
| 调试时看不到源码、提示No such file or directory | 可执行文件和源码路径不匹配，或编译后源码被移动/删除 | 用 `dir <源码目录>` 命令添加源码搜索路径 |
| 打印变量提示optimized out | 编译器优化把变量优化掉了，无运行时数据 | 关闭编译器优化，重新用 `-O0` 编译 |

### 4.3 进阶效率技巧
1. 配合AddressSanitizer快速定位内存问题：编译时添加 `-fsanitize=address -g`，可直接定位内存越界、野指针、内存泄漏、释放后使用等问题，比纯GDB调试效率提升数倍。
2. 用cgdb替代原生GDB：cgdb是GDB的增强终端界面，默认分屏显示源码和调试命令，操作更直观，安装命令 `sudo apt install cgdb`。
3. 远程调试：嵌入式开发、跨平台调试场景，目标机运行 `gdbserver <ip:port> <程序>`，本地GDB通过 `target remote <ip:port>` 连接远程目标机调试。
4. 日志断点：不暂停程序，仅打印日志，通过 `commands` 实现断点命中后打印信息并自动continue，不影响程序运行。

## 五、GDB标准调试流程（新手快速上手）
1. 编译程序：`gcc -g -O0 test.c -o test`
2. 启动GDB：`gdb ./test`
3. 设置入口断点：`b main`
4. 运行程序：`run`
5. 流程控制：通过 `n/s/c` 控制程序执行，配合断点定位问题位置
6. 数据验证：通过 `p/display/x` 查看变量、内存，验证程序运行状态
7. 根因定位：通过 `bt/f` 查看调用栈，切换栈帧分析崩溃/逻辑错误
8. 退出调试：`quit`


