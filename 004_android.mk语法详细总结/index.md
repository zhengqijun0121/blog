# Android.mk 语法详细总结


# Android.mk 语法详细总结
Android.mk 是 Android NDK 构建系统基于 GNU Make 封装的原生代码构建脚本，用于描述 C/C++ 源码、库、可执行文件的编译规则，是 NDK 开发中 native 代码构建的核心配置文件，本质是 Makefile 语法的子集，与 Android 平台源码构建语法高度兼容。

## 一、基础核心结构（必写模板）
一个合法的 Android.mk 必须遵循固定的结构范式，每个模块的定义都需遵循该流程，最小可运行模板如下：
```makefile
# 1. 必须放在文件最开头：定义当前mk文件所在目录，my-dir是NDK内置函数
LOCAL_PATH := $(call my-dir)

# 2. 清空LOCAL_系列变量（仅保留LOCAL_PATH），每个新模块定义前必须调用
include $(CLEAR_VARS)

# 3. 模块核心属性定义（模块名、源码、编译选项等）
LOCAL_MODULE := hello_jni  # 模块名，全局唯一
LOCAL_SRC_FILES := hello_jni.c  # 待编译的源码文件，路径相对LOCAL_PATH

# 4. 指定构建目标类型，收尾模块定义
include $(BUILD_SHARED_LIBRARY)
```
> 核心规则：一个 Android.mk 可定义多个模块，**每个新模块前必须重新执行 `include $(CLEAR_VARS)`**，LOCAL_PATH 仅需在文件开头定义一次。

## 二、系统预定义核心变量
NDK 构建系统提前定义好的全局变量，无需用户赋值，直接引用即可，分为构建目标、平台架构两大类。

### 1. 构建目标类型变量
| 变量名 | 核心作用 |
|--------|----------|
| CLEAR_VARS | 指向系统清空脚本，**每个新模块前必须执行 `include $(CLEAR_VARS)`**，会清空除 LOCAL_PATH 外的所有 LOCAL_* 变量，避免模块间变量污染 |
| BUILD_SHARED_LIBRARY | 构建动态共享库（.so），将源码编译为可打包到APK的共享库，是JNI开发最常用的类型 |
| BUILD_STATIC_LIBRARY | 构建静态库（.a），编译后的静态库会被完整嵌入到依赖它的模块二进制文件中，不会单独打包到APK |
| BUILD_EXECUTABLE | 构建原生可执行文件，生成可在Android终端shell中直接运行的二进制程序 |
| PREBUILT_SHARED_LIBRARY | 预编译共享库模板，用于引入第三方已编译好的.so文件，无需重新编译源码 |
| PREBUILT_STATIC_LIBRARY | 预编译静态库模板，用于引入第三方已编译好的.a文件 |
| BUILD_PREBUILT | 预编译库通用底层模板，是上述两个预编译变量的底层实现 |

### 2. 目标平台与架构变量
用于差异化适配不同CPU架构、Android系统版本，常用于条件编译。
| 变量名 | 核心作用与取值 |
|--------|----------------|
| TARGET_ARCH | 目标CPU架构，取值：`arm`、`arm64`、`x86`、`x86_64` |
| TARGET_ARCH_ABI | 目标应用二进制接口(ABI)，常用取值：`armeabi-v7a`、`arm64-v8a`、`x86`、`x86_64` |
| TARGET_PLATFORM | 目标Android API级别，格式为 `android-XX`，如 `android-21` |
| TARGET_ABI | 目标平台+ABI的完整组合，如 `android-21-arm64-v8a`，用于精准区分目标环境 |

## 三、模块定义核心 LOCAL_* 变量
这是 Android.mk 最核心的部分，用于描述单个模块的编译规则，所有变量均以 `LOCAL_` 开头，必须在 `include $(CLEAR_VARS)` 之后、`include $(BUILD_XXX)` 之前定义。

### 1. 模块基础标识变量
| 变量名 | 作用与用法规范 |
|--------|----------------|
| LOCAL_MODULE | **必填项**，模块名称，必须全局唯一，无空格。<br>命名规则：共享库/静态库默认自动加 `lib` 前缀和对应后缀，如模块名 `hello` 生成 `libhello.so`；**若模块名已以 `lib` 开头，系统不会重复添加前缀** |
| LOCAL_MODULE_FILENAME | 可选，自定义生成文件的完整名称，覆盖系统默认命名规则。<br>示例：`LOCAL_MODULE_FILENAME := myhello` 可生成 `myhello.so`，而非默认的 `libhello.so` |
| LOCAL_MODULE_TAGS | 可选，模块标签，控制模块的编译场景，常用值：<br>`optional`（默认）：所有编译版本均编译；`debug`：仅debug版本编译；`eng`：仅工程版本编译 |
| LOCAL_PATH | **每个Android.mk必须在开头定义一次**，指定当前mk文件所在的目录，固定写法：`LOCAL_PATH := $(call my-dir)`<br>该变量不会被 `CLEAR_VARS` 清空，一个mk文件仅需定义一次 |
| LOCAL_MODULE_CLASS | 模块分类，预编译模块、系统级编译常用，可选值：`SHARED_LIBRARIES`、`STATIC_LIBRARIES`、`EXECUTABLES` |

### 2. 源码与头文件相关变量
| 变量名 | 作用与用法规范 |
|--------|----------------|
| LOCAL_SRC_FILES | **必填项**，待编译的C/C++源文件列表，空格分隔，路径**相对于LOCAL_PATH**。<br>⚠️ 禁止放入头文件，构建系统自动处理头文件依赖；必须使用Unix风格正斜杠 `/`，不支持Windows反斜杠 `\`；支持多目录、多后缀文件<br>示例：`LOCAL_SRC_FILES := hello.c test.cpp utils/string.c` |
| LOCAL_CPP_EXTENSION | 可选，自定义C++文件的扩展名，默认 `.cpp`，可指定多个后缀。<br>示例：`LOCAL_CPP_EXTENSION := .cc .cxx .cpp` |
| LOCAL_C_INCLUDES | 头文件搜索路径，空格分隔，路径可相对于LOCAL_PATH或NDK根目录。<br>优先级高于 `LOCAL_CFLAGS` 中的 `-I` 选项，且会被用于ndk-gdb调试的头文件路径解析，推荐优先使用。<br>示例：`LOCAL_C_INCLUDES := $(LOCAL_PATH)/include $(LOCAL_PATH)/3rdparty/inc` |
| LOCAL_EXPORT_C_INCLUDES | 导出头文件路径，**核心高频用法**。<br>当其他模块依赖当前模块时，系统会自动将此处的路径添加到依赖模块的 `LOCAL_C_INCLUDES` 中，无需手动重复配置，常用于库的对外头文件声明 |

### 3. 编译选项相关变量
| 变量名 | 作用与用法规范 |
|--------|----------------|
| LOCAL_CFLAGS | C/C++ 通用编译选项，空格分隔，常用场景：宏定义、优化等级、警告配置。<br>示例：`LOCAL_CFLAGS := -DDEBUG -O2 -Wall -Werror` |
| LOCAL_CONLYFLAGS | 仅对C语言生效的编译选项，编译C++文件时不会传递该参数，用于C/C++混合代码的差异化编译 |
| LOCAL_CPPFLAGS | 仅对C++生效的编译选项，追加在 `LOCAL_CFLAGS` 之后，常用场景：C++标准指定。<br>示例：`LOCAL_CPPFLAGS := -std=c++17 -fno-rtti` |
| LOCAL_CPP_FEATURES | 极简开启C++标准特性，替代手动配置编译FLAGS，常用值：`rtti`（运行时类型识别）、`exceptions`（异常处理）。<br>示例：`LOCAL_CPP_FEATURES := rtti exceptions` |
| LOCAL_ARM_MODE | ARM架构专属，指定指令集模式：<br>`thumb`（默认）：16位精简指令集，代码体积更小；`arm`：32位ARM指令集，性能更强<br>单文件指定：给文件加 `.arm` 后缀，如 `foo.c.arm` |
| LOCAL_ARM_NEON | ARM架构专属，开启NEON SIMD指令集，设置为 `on` 全局开启；单文件指定加 `.neon` 后缀，如 `foo.c.neon` |
| LOCAL_EXPORT_CFLAGS | 导出C/C++编译选项，依赖当前模块的模块会自动继承这些编译参数 |
| LOCAL_EXPORT_CPPFLAGS | 导出C++专属编译选项，依赖当前模块的模块会自动继承 |
| LOCAL_SHORT_COMMANDS | 针对源码/依赖极多的大模块，设置为 `true` 开启短命令模式，解决Windows/Make命令行长度限制问题 |

### 4. 链接与依赖相关变量
| 变量名 | 作用与用法规范 |
|--------|----------------|
| LOCAL_LDFLAGS | 链接器通用选项，空格分隔，用于自定义链接规则、链接脚本、符号导出配置等 |
| LOCAL_LDLIBS | 链接Android系统预编译的动态库，格式为 `-lxxx`，**仅可用于系统库，不可链接自定义模块**。<br>常用系统库：`-llog`（Android日志库）、`-landroid`（Android原生API）、`-lGLESv2`（OpenGL ES 2.0）、`-lz`（zlib压缩库）、`-lm`（数学库）<br>示例：`LOCAL_LDLIBS := -llog -landroid -lGLESv2` |
| LOCAL_SHARED_LIBRARIES | 当前模块依赖的**共享库模块名列表**，空格分隔。<br>链接时会关联依赖的so库，运行时必须加载依赖库；自动继承依赖模块的 `LOCAL_EXPORT_*` 所有配置（头文件、编译选项、链接参数等） |
| LOCAL_STATIC_LIBRARIES | 当前模块依赖的**静态库模块名列表**，空格分隔。<br>静态库会被编译进当前模块的二进制文件中，适合小体积工具库，默认会丢弃未被直接引用的符号 |
| LOCAL_WHOLE_STATIC_LIBRARIES | 全量链接静态库，与 `LOCAL_STATIC_LIBRARIES` 核心区别：<br>不会丢弃静态库中未被当前模块直接引用的符号，常用于静态库嵌套依赖、需要导出静态库全部符号的场景，避免运行时符号找不到 |
| LOCAL_EXPORT_LDFLAGS | 导出链接选项，依赖当前模块的模块会自动继承这些链接参数 |
| LOCAL_EXPORT_LDLIBS | 导出系统库依赖，依赖当前模块的模块会自动继承这些系统库链接配置 |
| LOCAL_ALLOW_UNDEFINED_SYMBOLS | 是否允许未定义符号，默认关闭。开启后链接时不会报未定义符号错误，仅特殊场景使用，不推荐开启 |

### 5. 预编译库专属变量
用于引入第三方已编译好的 `.so`/`.a` 文件，无需重新编译源码，核心配置如下：
| 变量名 | 作用与用法规范 |
|--------|----------------|
| LOCAL_SRC_FILES | 预编译库文件的路径，相对于LOCAL_PATH，支持按ABI区分路径<br>示例：`LOCAL_SRC_FILES := libs/$(TARGET_ARCH_ABI)/libffmpeg.so` |
| LOCAL_EXPORT_C_INCLUDES | 预编译库对应的头文件路径，必须配置，否则依赖模块无法找到头文件 |
| LOCAL_MODULE_CLASS | 预编译模块分类，共享库填 `SHARED_LIBRARIES`，静态库填 `STATIC_LIBRARIES` |
| 收尾指令 | 共享库：`include $(PREBUILT_SHARED_LIBRARY)`<br>静态库：`include $(PREBUILT_STATIC_LIBRARY)` |

### 6. 其他常用变量
| 变量名 | 作用与用法规范 |
|--------|----------------|
| LOCAL_MODULE_PATH | 自定义模块生成后的输出路径 |
| LOCAL_STRIP_MODE | 符号剥离配置，默认自动strip；设置为 `nostrip` 保留符号表，用于调试 |
| LOCAL_THIN_ARCHIVE | 静态库专属，设置为 `true` 生成瘦静态库，仅保留符号索引，不打包目标文件，大幅减小库体积 |
| LOCAL_DISABLE_FORMAT_STRING_CHECKS | 关闭格式化字符串安全检查，默认开启，不推荐关闭 |

## 四、NDK 内置常用函数
所有函数均通过 `$(call 函数名, 参数)` 格式调用，核心高频函数如下：
1.  **my-dir**
    无参数，返回当前 Android.mk 文件所在的目录路径，**每个mk文件开头必须调用**，用于定义 LOCAL_PATH。
    示例：`LOCAL_PATH := $(call my-dir)`

2.  **all-subdir-makefiles**
    无参数，返回当前目录下所有子目录中的 Android.mk 文件路径，用于递归包含子目录的构建脚本。
    示例：`include $(call all-subdir-makefiles)`

3.  **all-makefiles-under**
    参数为指定目录路径，返回该目录下所有子目录中的 Android.mk 文件路径，比 all-subdir-makefiles 更灵活。
    示例：`include $(call all-makefiles-under, $(LOCAL_PATH)/modules)`

4.  **this-makefile**
    无参数，返回当前正在执行的 Makefile 的完整路径。

5.  **parent-makefile**
    无参数，返回包含当前 Makefile 的父 Makefile 路径。

6.  **grand-parent-makefile**
    无参数，返回包含当前 Makefile 的祖父级 Makefile 路径。

7.  **import-module**
    参数为模块的相对路径，用于导入外部模块，根据 `NDK_MODULE_PATH` 环境变量搜索模块目录，常用于引入第三方NDK模块。
    示例：`$(call import-module, thirdparty/ffmpeg)`

## 五、条件控制语法
Android.mk 完全兼容 GNU Make 的条件判断语法，用于实现按架构、平台、编译模式等场景的差异化编译，核心语法如下。

### 1. 相等/不等判断：ifeq / ifneq
```makefile
# 按CPU架构区分源码
ifeq ($(TARGET_ARCH), arm64)
    LOCAL_SRC_FILES += arm64_utils.c
else ifeq ($(TARGET_ARCH), x86_64)
    LOCAL_SRC_FILES += x64_utils.c
else
    LOCAL_SRC_FILES += default_utils.c
endif

# 按编译模式配置优化选项
ifeq ($(APP_OPTIM), debug)
    LOCAL_CFLAGS += -O0 -g -DDEBUG_MODE
else
    LOCAL_CFLAGS += -O2 -DNDEBUG
endif
```

### 2. 变量定义判断：ifdef / ifndef
```makefile
# 若定义了ENABLE_LOG宏，则开启日志功能
ifdef ENABLE_LOG
    LOCAL_CFLAGS += -DENABLE_LOG
    LOCAL_LDLIBS += -llog
endif

# 若未定义DISABLE_SECURITY，则开启安全编译选项
ifndef DISABLE_SECURITY
    LOCAL_CFLAGS += -fstack-protector-strong
endif
```

## 六、典型场景完整示例
### 示例1：基础JNI共享库（最常用）
```makefile
LOCAL_PATH := $(call my-dir)
include $(CLEAR_VARS)

# 模块基础信息
LOCAL_MODULE := hello_jni
LOCAL_SRC_FILES := hello_jni.c
# 头文件路径
LOCAL_C_INCLUDES := $(LOCAL_PATH)/include
# 链接Android日志库
LOCAL_LDLIBS := -llog
# 开启C++17标准
LOCAL_CPPFLAGS := -std=c++17
# 开启所有编译警告
LOCAL_CFLAGS := -Wall

include $(BUILD_SHARED_LIBRARY)
```

### 示例2：静态库+依赖静态库的共享库
```makefile
LOCAL_PATH := $(call my-dir)

# 1. 编译工具静态库
include $(CLEAR_VARS)
LOCAL_MODULE := utils
LOCAL_SRC_FILES := utils/string.c utils/math.c
LOCAL_C_INCLUDES := $(LOCAL_PATH)/utils
# 导出头文件，依赖模块自动继承
LOCAL_EXPORT_C_INCLUDES := $(LOCAL_PATH)/utils
include $(BUILD_STATIC_LIBRARY)

# 2. 编译主共享库，依赖上述静态库
include $(CLEAR_VARS)
LOCAL_MODULE := main_lib
LOCAL_SRC_FILES := main.c
# 依赖静态库
LOCAL_STATIC_LIBRARIES := utils
LOCAL_LDLIBS := -llog
include $(BUILD_SHARED_LIBRARY)
```

### 示例3：引入第三方预编译共享库
```makefile
LOCAL_PATH := $(call my-dir)

# 引入预编译的FFmpeg库
include $(CLEAR_VARS)
LOCAL_MODULE := ffmpeg
LOCAL_SRC_FILES := libs/$(TARGET_ARCH_ABI)/libffmpeg.so
# 导出头文件路径
LOCAL_EXPORT_C_INCLUDES := $(LOCAL_PATH)/ffmpeg/include
include $(PREBUILT_SHARED_LIBRARY)

# 主库依赖FFmpeg
include $(CLEAR_VARS)
LOCAL_MODULE := video_player
LOCAL_SRC_FILES := player.c decoder.c
# 依赖预编译的FFmpeg库
LOCAL_SHARED_LIBRARIES := ffmpeg
LOCAL_LDLIBS := -llog -landroid -lGLESv2
include $(BUILD_SHARED_LIBRARY)
```

### 示例4：编译原生可执行文件
```makefile
LOCAL_PATH := $(call my-dir)
include $(CLEAR_VARS)

LOCAL_MODULE := test_shell
LOCAL_SRC_FILES := test.c
# 静态链接，减少运行时依赖
LOCAL_FORCE_STATIC_EXECUTABLE := true
LOCAL_STATIC_LIBRARIES := utils
include $(BUILD_EXECUTABLE)
```

## 七、最佳实践与高级用法
1.  **模块化导出配置**
    库模块必须通过 `LOCAL_EXPORT_C_INCLUDES` 导出对外头文件，通过 `LOCAL_EXPORT_LDLIBS`/`LOCAL_EXPORT_CFLAGS` 导出依赖的系统库和编译选项，避免依赖模块重复配置，减少出错。

2.  **单文件多模块规范**
    一个 Android.mk 中可定义多个模块，**每个模块前必须执行 `include $(CLEAR_VARS)`**，LOCAL_PATH 仅需在开头定义一次，禁止多个模块共用一个 CLEAR_VARS 作用域。

3.  **ABI差异化适配**
    针对不同架构使用专属源码/库，通过 `TARGET_ARCH_ABI` 条件判断实现，避免无效代码编译，减小包体积。

4.  **递归包含子目录**
    项目源码分多目录时，在根目录 Android.mk 末尾添加 `include $(call all-subdir-makefiles)`，自动包含所有子目录的mk文件，无需手动逐个引入。

5.  **调试与发布分离**
    通过 `APP_OPTIM` 区分debug和release模式，debug模式关闭优化、保留调试信息，release模式开启优化、strip符号，减小体积。

## 八、常见坑与避坑指南
1.  **变量污染**：每个新模块前必须加 `include $(CLEAR_VARS)`，否则前一个模块的变量会污染后一个模块，导致诡异的编译错误。
2.  **LOCAL_PATH 位置错误**：必须在 Android.mk 最开头定义 LOCAL_PATH，且仅定义一次，避免被 CLEAR_VARS 清空。
3.  **依赖库链接错误**：`LOCAL_LDLIBS` 只能链接系统库，自定义模块必须用 `LOCAL_SHARED_LIBRARIES`/`LOCAL_STATIC_LIBRARIES`，否则会报符号找不到。
4.  **静态库符号丢失**：静态库嵌套依赖、需要导出全部符号时，必须用 `LOCAL_WHOLE_STATIC_LIBRARIES`，否则未被直接引用的符号会被链接器丢弃，运行时崩溃。
5.  **头文件路径错误**：对外头文件必须用 `LOCAL_EXPORT_C_INCLUDES` 导出，否则依赖模块需要手动添加头文件路径，否则会报头文件找不到。
6.  **模块名重复**：模块名必须全局唯一，否则后定义的模块会覆盖先定义的，导致编译异常。
7.  **路径格式错误**：LOCAL_SRC_FILES 必须使用Unix风格正斜杠 `/`，Windows反斜杠 `\` 会导致构建系统无法识别路径。

## 九、与 Application.mk 的配合
Android.mk 是**模块级**构建配置，描述单个模块的编译规则；Application.mk 是**项目级**全局配置，作用于当前项目所有模块，二者配合完成NDK构建，Application.mk 核心常用配置如下：
| 配置项 | 核心作用 |
|--------|----------|
| APP_ABI | 指定编译的目标ABI，如 `arm64-v8a armeabi-v7a x86_64`，默认编译所有支持的ABI |
| APP_PLATFORM | 指定最低支持的Android API级别，与TARGET_PLATFORM对应 |
| APP_STL | 指定C++标准库，NDK r18+默认使用 `c++_shared`，可选 `c++_static`、`none` |
| APP_OPTIM | 编译模式，`debug`（调试模式）或 `release`（发布模式） |
| APP_CFLAGS/APP_CPPFLAGS | 全局编译选项，作用于项目中所有模块 |

> 补充说明：Android.mk 是传统NDK构建系统，目前Google推荐新工程使用 CMake 构建，但Android.mk 在大量存量项目、系统级native开发中仍广泛使用，是Android NDK开发的必备基础。


