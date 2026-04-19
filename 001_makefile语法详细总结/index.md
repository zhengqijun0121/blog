# Makefile 语法详细总结


# Makefile 语法详细总结
Makefile 是 GNU make 工具的配置文件，核心是**基于文件时间戳的依赖管理**，仅当依赖文件更新时才重新构建目标，大幅提升大型项目的构建效率，广泛用于C/C++项目，也可用于任意自动化任务。

---

## 一、核心基础结构
### 1. 核心规则（最基础单元）
Makefile 的核心是**规则(Rule)**，一条完整规则的标准格式如下，**命令行必须以Tab字符开头，不能用空格替代**（新手最常见报错原因）：
```makefile
目标名: 依赖文件列表
    命令1
    命令2
    ...
```
- **目标名**：要生成的文件，或要执行的动作名（如clean）
- **依赖**：生成目标所需的文件/其他目标，依赖不存在时会先递归构建对应目标
- **命令**：构建目标要执行的shell命令，可多行

### 2. 核心执行逻辑
1. make 默认执行文件中**第一个目标**（行业惯例命名为`all`）
2. 检查目标是否存在，以及所有依赖的时间戳是否比目标新
3. 若依赖更新/目标不存在，执行对应命令重新生成目标
4. 依赖会递归执行上述检查，直到所有依赖满足条件

### 3. 伪目标 .PHONY
伪目标是不对应实际文件的动作目标（如清理、安装），**必须用`.PHONY`声明**，避免目录下出现同名文件时，make误判目标已最新而不执行命令。
```makefile
.PHONY: all clean install

all: main
clean:
    rm -rf *.o main
install:
    cp main /usr/local/bin
```
声明后，make会无条件执行伪目标的命令，不会检查同名文件。

---

## 二、变量详解
Makefile 变量本质是文本替换，类似C语言宏，分为自定义变量、自动化变量、内置变量三类。

### 1. 变量赋值运算符
| 运算符 | 类型 | 核心特性 | 示例 |
|--------|------|----------|------|
| `=`    | 递归展开赋值 | 变量在**使用时**才展开，支持后定义的变量引用 | `VAR = $(OTHER_VAR)` |
| `:=`   | 立即展开赋值 | 变量在**定义时**直接展开，后续修改不影响已赋值的变量 | `VAR := $(shell pwd)` |
| `?=`   | 条件赋值 | 仅当变量未定义时才赋值，已定义则不生效 | `CC ?= gcc` |
| `+=`   | 追加赋值 | 给已定义的变量追加内容，保留原有值 | `CFLAGS += -Wall` |

**关键区别示例**：
```makefile
# 递归展开 = ：使用时才解析，A最终是 hello world
A = $(B)
B = hello world

# 立即展开 := ：定义时B为空，C最终是空
C := $(B)
B = hello world
```

### 2. 变量引用规则
- 标准格式：`$(变量名)` 或 `${变量名}`，推荐用括号
- 单字符变量可简写：`$X`，多字符变量必须用括号
- 特殊字符`$`需要转义：`$$` 表示shell中的`$`符号

### 3. 自动化变量（核心高频）
自动化变量是make在规则执行时，根据目标和依赖自动赋值的变量，**仅在规则的命令中生效**，是编写通用规则的核心。

| 自动化变量 | 含义 |
|------------|------|
| `$@`       | 当前规则的**目标名** |
| `$<`       | 第一个依赖文件的名称 |
| `$^`       | 所有依赖文件的列表，自动去重 |
| `$?`       | 所有比目标新的依赖文件列表，去重 |
| `$+`       | 所有依赖文件的列表，不去重，保留重复项 |
| `$*`       | 模式匹配中，`%`匹配到的部分 |

**高频示例**：
```makefile
main: main.o func.o
    gcc $^ -o $@  # 等价于 gcc main.o func.o -o main

%.o: %.c
    gcc -c $< -o $@  # 通用编译规则，匹配所有.c生成对应.o
```

### 4. 常用内置变量
make预定义了大量内置变量，可直接使用或修改，高频如下：
| 变量名 | 默认值 | 用途 |
|--------|--------|------|
| `CC`   | `cc`（通常链接到gcc） | C语言编译器 |
| `CXX`  | `g++` | C++编译器 |
| `CFLAGS` | 空 | C语言编译选项（如-Wall、-O2、-I头文件路径） |
| `CXXFLAGS` | 空 | C++编译选项 |
| `LDFLAGS` | 空 | 链接选项（如-L库文件路径） |
| `LDLIBS` | 空 | 链接库（如-lm、-lpthread） |
| `RM`   | `rm -f` | 删除命令 |

---

## 三、规则进阶
### 1. 显式规则
用户手动明确指定目标、依赖、命令的规则，是最基础的规则写法，适合目标少、依赖固定的场景。

### 2. 静态模式规则
核心是**模式匹配**，用`%`通配符匹配目标，批量生成规则，比普通隐式规则更精准，仅作用于指定的目标列表。
语法格式：
```makefile
目标列表: 目标模式: 依赖模式
    命令
```
**示例**：将指定的.c文件编译为对应的.o文件
```makefile
# 定义所有要生成的.o文件
OBJS = main.o func.o utils.o

# 静态模式规则：匹配OBJS中的所有.o，依赖为对应的.c
$(OBJS): %.o: %.c
    $(CC) $(CFLAGS) -c $< -o $@
```

### 3. 隐式规则（内置规则）
make内置了大量默认规则，无需手动编写，比如：
- .c文件自动编译为.o文件：默认执行 `$(CC) $(CFLAGS) $(CPPFLAGS) -c $< -o $@`
- .cpp/.cc文件自动编译为.o文件：默认执行 `$(CXX) $(CXXFLAGS) $(CPPFLAGS) -c $< -o $@`

可通过修改内置变量（如CFLAGS）调整隐式规则行为，也可手动重写规则覆盖默认行为。

### 4. 多目标规则
一条规则生成多个目标，多个目标共享依赖和命令，示例：
```makefile
a.o b.o: %.o: %.c
    $(CC) $(CFLAGS) -c $< -o $@
```

---

## 四、条件判断语法
用于根据变量值、平台、编译器等条件，动态决定Makefile的执行逻辑，所有条件判断都以`endif`结尾，必须成对出现。

| 指令 | 作用 |
|------|------|
| `ifeq (A, B)` / `ifeq "A" "B"` | 判断A和B是否相等，相等则执行后续逻辑 |
| `ifneq (A, B)` | 判断A和B是否不相等，不相等则执行后续逻辑 |
| `ifdef VAR` | 判断变量VAR是否已定义（非空），已定义则执行 |
| `ifndef VAR` | 判断变量VAR是否未定义，未定义则执行 |
| `else` | 条件不满足时的分支 |
| `endif` | 条件判断结束 |

**示例1：判断编译器类型**
```makefile
CC ?= gcc
ifeq ($(CC), gcc)
    CFLAGS += -fgnu89-inline
else ifeq ($(CC), clang)
    CFLAGS += -Wno-deprecated-declarations
endif
```

**示例2：调试/发布模式切换**
```makefile
ifndef DEBUG
    CFLAGS += -O2  # 未定义DEBUG，默认开启优化
else
    CFLAGS += -g -O0 -DDEBUG  # 定义DEBUG，开启调试信息
endif
```

---

## 五、常用函数详解
Makefile 提供了大量内置函数，用于文本处理、文件名操作、流程控制等，函数调用统一格式：
```makefile
$(函数名 参数1,参数2,参数3,...)
```
- 参数之间用**逗号**分隔，空格会被当作参数内容的一部分
- 函数名和参数之间用空格分隔

### 1. 高频字符串处理函数
| 函数 | 语法 | 作用 | 示例 | 结果 |
|------|------|------|------|------|
| `subst` | `$(subst 旧字符串,新字符串,文本)` | 全局替换文本中的旧字符串为新字符串 | `$(subst .c,.o,main.c func.c)` | `main.o func.o` |
| `patsubst` | `$(patsubst 模式,替换模式,文本)` | 模式替换，支持%通配符 | `$(patsubst %.c,%.o,main.c func.c)` | `main.o func.o` |
| `strip` | `$(strip 文本)` | 去除首尾空格，合并中间多个空格为单个 | `$(strip  hello   world  )` | `hello world` |
| `filter` | `$(filter 模式,文本)` | 过滤出文本中匹配模式的内容 | `$(filter %.c,main.c func.o test.h)` | `main.c` |
| `filter-out` | `$(filter-out 模式,文本)` | 过滤掉匹配模式的内容，保留剩余 | `$(filter-out %.o,main.c func.o test.h)` | `main.c test.h` |
| `sort` | `$(sort 文本)` | 对单词按字典序排序，自动去重 | `$(sort b a c a b)` | `a b c` |
| `word` | `$(word 序号,文本)` | 提取文本中第N个单词（序号从1开始） | `$(word 2,main.c func.c test.c)` | `func.c` |
| `words` | `$(words 文本)` | 统计文本中单词的个数 | `$(words a b c)` | `3` |

### 2. 高频文件名处理函数
| 函数 | 语法 | 作用 | 示例 | 结果 |
|------|------|------|------|------|
| `wildcard` | `$(wildcard 通配符模式)` | 匹配当前目录下符合模式的所有文件，返回文件名列表 | `$(wildcard *.c)` | 所有.c文件的列表 |
| `dir` | `$(dir 文件名列表)` | 提取文件路径中的目录部分 | `$(dir src/main.c /home/test.c)` | `src/ /home/` |
| `notdir` | `$(notdir 文件名列表)` | 提取文件名，去除路径 | `$(notdir src/main.c test.h)` | `main.c test.h` |
| `suffix` | `$(suffix 文件名列表)` | 提取文件的后缀名 | `$(suffix main.c test.h)` | `.c .h` |
| `basename` | `$(basename 文件名列表)` | 提取文件名前缀，去除后缀 | `$(basename main.c test.h)` | `main test` |
| `addprefix` | `$(addprefix 前缀,文件名列表)` | 给每个文件名添加前缀 | `$(addprefix src/,main.c func.c)` | `src/main.c src/func.c` |
| `addsuffix` | `$(addsuffix 后缀,文件名列表)` | 给每个文件名添加后缀 | `$(addsuffix .o,main func)` | `main.o func.o` |

**高频组合示例**：自动获取当前目录所有.c文件，生成对应的.o文件列表
```makefile
# 1. 获取所有.c文件
SRCS = $(wildcard *.c)
# 2. 将.c替换为.o，生成目标文件列表
OBJS = $(patsubst %.c,%.o,$(SRCS))
```

### 3. 流程控制函数
| 函数 | 语法 | 作用 |
|------|------|------|
| `foreach` | `$(foreach 变量,列表,表达式)` | 遍历列表中的每个元素，赋值给变量，执行表达式，返回所有结果 |
| `if` | `$(if 条件,成立表达式,不成立表达式)` | 条件判断，条件非空则执行成立分支，否则执行不成立分支 |
| `call` | `$(call 函数名,参数1,参数2,...)` | 自定义函数调用，将参数依次替换给函数中的$(1),$(2)... |
| `eval` | `$(eval 文本)` | 将文本作为Makefile代码动态解析执行，用于动态生成规则 |

**foreach示例**：遍历多个目录，获取所有.c文件
```makefile
DIRS = src utils test
SRCS = $(foreach dir,$(DIRS),$(wildcard $(dir)/*.c))
```

### 4. 其他常用函数
| 函数 | 语法 | 作用 |
|------|------|------|
| `shell` | `$(shell 命令)` | 执行shell命令，返回命令的标准输出结果 |
| `info` | `$(info 文本)` | 打印信息到控制台，不中断执行 |
| `warning` | `$(warning 文本)` | 打印警告信息，不中断执行 |
| `error` | `$(error 文本)` | 打印错误信息，**立即终止make执行** |

**shell示例**：获取当前工作路径
```makefile
CUR_DIR := $(shell pwd)
$(info 当前目录: $(CUR_DIR))
```

---

## 六、命令前缀与执行特性
### 1. 命令前缀
可给命令行添加前缀，控制命令的执行行为：
| 前缀 | 作用 | 示例 |
|------|------|------|
| `@` | 静默执行，不打印命令本身，只输出执行结果 | `@echo "编译完成"` |
| `-` | 忽略命令执行的错误，继续执行后续命令 | `-rm -rf *.o`（文件不存在时不报错） |
| `+` | 强制命令在make递归调用、-n/-t/-q参数下仍执行 | `+$(MAKE) -C subdir` |

### 2. 命令执行的关键特性
**每个命令行在独立的shell进程中执行**，这是最常见的坑！比如cd、环境变量等操作，仅在当前行生效，下一行会重置：
```makefile
# 错误写法：cd只在第一行生效，第二行仍在原目录
test:
    cd src
    pwd

# 正确写法：用&&或反斜杠续行，让命令在同一个shell中执行
test:
    cd src && pwd
    # 或者
    cd src; \
    pwd
```
- 命令执行失败（返回非0退出码），make会立即终止，除非加了`-`前缀
- 可用`\`续行，将多行命令合并为一行，在同一个shell中执行

---

## 七、其他核心语法
### 1. 包含其他Makefile：include
用于拆分大型Makefile，引入其他配置文件，语法：
```makefile
include 文件名1 文件名2 ...
# 忽略不存在的文件，不报错
-include 可选文件
```
- make会先读取include的文件，再执行后续逻辑
- 常用场景：引入自动生成的依赖文件（.d文件）、子目录Makefile、配置文件

### 2. override指令
用于强制覆盖用户通过命令行传入的变量，语法：
```makefile
# 即使用户执行 make CFLAGS=-O0，也会强制追加-Wall
override CFLAGS += -Wall
```
- 默认情况下，命令行传入的变量优先级高于Makefile中定义的变量，override可打破该规则

### 3. 递归调用make
用于构建子目录的项目，语法：
```makefile
# 进入subdir目录，执行该目录下的Makefile
.PHONY: subdir
subdir:
    $(MAKE) -C subdir
```
- 必须用`$(MAKE)`变量，而非直接写make，可传递make的参数和环境变量
- `-C`参数用于切换到指定目录

---

## 八、make 执行与常用命令行参数
### 1. 基本执行
```bash
# 执行当前目录下的Makefile，默认执行第一个目标
make

# 执行指定目标
make clean
make install

# 指定Makefile文件
make -f MyMakefile
```

### 2. 高频命令行参数
| 参数 | 作用 |
|------|------|
| `-f 文件` | 指定要执行的Makefile文件 |
| `-C 目录` | 切换到指定目录后执行make |
| `-n` |  dry run，只打印要执行的命令，不实际执行，用于调试 |
| `-j N` | 并行执行N个任务，大幅提升多核编译速度，如`make -j8` |
| `-B` | 强制重新构建所有目标，忽略时间戳检查 |
| `-p` | 打印所有内置规则、变量定义，用于调试 |
| `-d` | 打印详细的调试信息，查看依赖检查、规则匹配过程 |
| `-s` | 静默执行，不打印任何命令，类似给所有命令加@前缀 |
| `变量名=值` | 给Makefile传递变量，覆盖内部定义，如`make CC=clang DEBUG=1` |

---

## 九、常见坑与避坑指南
1. **Tab与空格问题**：命令行必须以Tab开头，不能用空格，否则会报`missing separator`错误
2. **shell隔离问题**：每行命令在独立shell执行，cd、环境变量等仅在当前行生效，必须用`&&`或`\`续行合并
3. **伪目标未声明**：clean、all等目标未加.PHONY，目录下出现同名文件时，make不执行命令
4. **递归展开变量坑**：用`=`定义的变量在使用时才展开，循环引用会导致栈溢出，推荐优先用`:=`立即展开
5. **通配符失效**：变量定义中`*`不会自动展开，必须用`wildcard`函数
6. **路径空格问题**：Makefile对路径中的空格支持很差，尽量避免路径和文件名中出现空格

---

## 十、通用Makefile示例
适用于中小型C/C++项目，自动扫描源文件，支持调试/发布模式，自动追踪头文件依赖：
```makefile
# 编译配置
CC ?= gcc
CFLAGS = -Wall -Wextra
LDFLAGS =
LDLIBS =

# 目录配置
SRC_DIR = src
OBJ_DIR = obj
BIN_DIR = bin
TARGET = $(BIN_DIR)/main

# 源文件与目标文件
SRCS = $(wildcard $(SRC_DIR)/*.c)
OBJS = $(patsubst $(SRC_DIR)/%.c,$(OBJ_DIR)/%.o,$(SRCS))
# 依赖文件，自动追踪头文件修改
DEPS = $(OBJS:.o=.d)

# 调试模式：make DEBUG=1
ifdef DEBUG
    CFLAGS += -g -O0 -DDEBUG
else
    CFLAGS += -O2
endif

# 默认目标
.PHONY: all
all: $(TARGET)

# 链接生成可执行文件
$(TARGET): $(OBJS) | $(BIN_DIR)
    $(CC) $(LDFLAGS) $^ -o $@ $(LDLIBS)

# 编译生成.o文件，同时生成.d依赖文件
$(OBJ_DIR)/%.o: $(SRC_DIR)/%.c | $(OBJ_DIR)
    $(CC) $(CFLAGS) -MMD -MP -c $< -o $@

# 自动创建目录
$(BIN_DIR) $(OBJ_DIR):
    mkdir -p $@

# 引入依赖文件
-include $(DEPS)

# 清理构建产物
.PHONY: clean
clean:
    $(RM) -rf $(OBJ_DIR) $(BIN_DIR)

# 打印配置信息
.PHONY: info
info:
    $(info 源文件: $(SRCS))
    $(info 目标文件: $(OBJS))
    $(info 编译器: $(CC))
    $(info 编译选项: $(CFLAGS))
```


