# 现代 CMake 语法详细总结


# Modern CMake 语法详细总结
Modern CMake 特指 CMake 3.0+（工业级项目推荐最低 3.15+）的构建范式，核心设计哲学是**面向目标（Target）编程、属性作用域隔离、依赖可传递**，彻底摒弃传统 CMake 全局变量污染的写法，实现「声明式描述构建需求，而非硬编码编译步骤」，是跨平台 C/C++ 项目的事实标准构建方案。

## 一、核心基础概念
### 1.1 核心范式：一切皆目标
CMake 中所有构建行为都围绕「目标」展开，目标是构建的最小独立单元，所有编译、链接、头文件、宏定义等配置均作为目标的属性，而非全局设置。

目标分为三大核心类型：
| 目标类型 | 定义命令 | 核心用途 |
|----------|----------|----------|
| 可执行目标 | `add_executable()` | 生成可执行程序 |
| 库目标 | `add_library()` | 生成静态库/动态库/头文件-only库/对象库 |
| 别名目标 | `add_library(xxx::yyy ALIAS)` | 给目标添加命名空间别名，避免名称冲突，与导出目标格式统一 |

### 1.2 核心传递性关键字（PRIVATE/PUBLIC/INTERFACE）
这是 Modern CMake 最核心的设计，决定目标属性是否传递给依赖它的其他目标，**所有 `target_*` 命令必须显式指定，禁止省略**。

| 关键字 | 作用规则 | 典型使用场景 |
|--------|----------|--------------|
| PRIVATE | 属性仅对当前目标生效，不传递给依赖该目标的下游目标 | 库内部使用的头文件路径、内部编译选项、仅内部使用的依赖 |
| PUBLIC | 属性对当前目标生效，同时完整传递给所有依赖该目标的下游目标 | 库对外暴露的头文件路径、API 依赖的 C++ 标准、下游必须继承的宏定义 |
| INTERFACE | 属性不对当前目标生效，仅传递给依赖该目标的下游目标 | 纯头文件模板库的配置、仅下游需要使用的编译选项 |

**传递性示例**：
A 链接 B，B 链接 C：
- 若 B 对 C 是 `PRIVATE`：A 看不到 C 的任何属性，也不会链接 C
- 若 B 对 C 是 `PUBLIC`：A 自动链接 C，完整继承 C 的 PUBLIC/INTERFACE 属性
- 若 B 对 C 是 `INTERFACE`：B 自身不链接 C，A 自动链接 C 并继承其属性

### 1.3 传统 CMake vs Modern CMake 核心区别
| 传统 CMake 全局命令 | Modern CMake 目标级替代命令 | 核心问题 |
|----------------------|------------------------------|----------|
| `include_directories()` | `target_include_directories()` | 全局头文件路径污染，易出现头文件冲突、作用域混乱 |
| `add_definitions()` | `target_compile_definitions()` | 全局宏定义泄露给所有目标 |
| `add_compile_options()` | `target_compile_options()` | 全局编译选项影响所有目标，跨平台适配困难 |
| `link_directories()` | `target_link_directories()`（极少推荐） | 全局库搜索路径混乱，链接优先级不可控 |
| `link_libraries()` | `target_link_libraries()` | 全局链接库强制所有目标链接，依赖关系不清晰 |
| 手动设置 `CMAKE_CXX_FLAGS` | `target_compile_features()`/`target_compile_options()` | 编译器相关硬编码，跨编译器兼容性差 |

## 二、项目基础配置命令
### 2.1 `cmake_minimum_required`：指定最低 CMake 版本
**必须放在 CMakeLists.txt 的第一行**，先于 `project()` 命令，否则会触发策略警告。

**完整语法**：
```cmake
cmake_minimum_required(VERSION <min>[...<max>] [FATAL_ERROR])
```

**关键参数与示例**：
```cmake
# 基础用法：最低 3.15 版本
cmake_minimum_required(VERSION 3.15 FATAL_ERROR)

# 推荐用法：3.12+ 支持版本范围，兼容新老版本策略
cmake_minimum_required(VERSION 3.15...3.30)
```
- `FATAL_ERROR`：3.12+ 已废弃，低版本不满足时默认直接报错
- 版本范围：下限指定最低兼容版本，上限指定最高测试过的版本，避免新版本 CMake 策略变更导致的构建失败

### 2.2 `project`：定义项目信息
**完整语法**：
```cmake
project(<PROJECT-NAME>
        [VERSION <major>[.<minor>[.<patch>[.<tweak>]]]]
        [DESCRIPTION <project-description-string>]
        [HOMEPAGE_URL <url-string>]
        [LANGUAGES <language-name>...])
```

**关键参数与示例**：
```cmake
project(MyDemo
        VERSION 1.2.3
        DESCRIPTION "A modern CMake demo project"
        HOMEPAGE_URL "https://github.com/xxx/mydemo"
        LANGUAGES CXX)
```
- `LANGUAGES`：指定项目使用的编程语言，默认 `C CXX`，仅用 C++ 时显式指定可减少配置时间
- 执行后自动生成核心变量：`PROJECT_NAME`、`PROJECT_VERSION`、`PROJECT_SOURCE_DIR`（当前项目源码目录）、`PROJECT_BINARY_DIR`（当前项目构建目录）

### 2.3 全局 C++ 标准配置（可选，推荐）
若项目所有目标统一使用相同 C++ 标准，可在 `project()` 后全局设置，优先级低于目标级 `target_compile_features()`：
```cmake
# 强制使用 C++17，不允许降级
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
# 禁用 GNU 编译器扩展（如 -std=gnu++17），保证跨编译器兼容性
set(CMAKE_CXX_EXTENSIONS OFF)
```

## 三、目标定义核心命令
### 3.1 `add_executable`：定义可执行目标
**完整语法**：
```cmake
add_executable(<name> [WIN32] [MACOSX_BUNDLE]
               [EXCLUDE_FROM_ALL]
               [source1] [source2 ...])
```

**关键参数与示例**：
```cmake
# 基础用法
add_executable(my_app main.cpp utils.cpp)

# Windows 窗口程序（WinMain 入口，而非 main）
add_executable(my_win_app WIN32 win_main.cpp)

# macOS 应用 Bundle
add_executable(my_mac_app MACOSX_BUNDLE mac_main.cpp)
```
- `EXCLUDE_FROM_ALL`：该目标不会被默认构建，仅当被其他目标依赖或手动构建时才会编译，常用于测试、示例代码
- **最佳实践**：显式列出源文件，禁止用 `file(GLOB)` 自动收集（除非使用 `CONFIGURE_DEPENDS`），避免新增文件后 CMake 无法自动感知

### 3.2 `add_library`：定义库目标
**基础语法**：
```cmake
add_library(<name> [STATIC | SHARED | MODULE | INTERFACE | OBJECT]
            [EXCLUDE_FROM_ALL]
            [source1] [source2 ...])
```

**库类型详解与示例**：
1. **STATIC 静态库**：编译为归档文件（`.a`/`.lib`），链接时直接嵌入到可执行文件中
   ```cmake
   add_library(my_static STATIC my_lib.cpp)
   ```

2. **SHARED 动态/共享库**：编译为动态链接库（`.so`/`.dll`/`.dylib`），运行时加载
   ```cmake
   add_library(my_shared SHARED my_lib.cpp)
   # 动态库默认开启 PIC（位置无关代码），静态库如需被动态库链接需手动开启
   set_target_properties(my_shared PROPERTIES POSITION_INDEPENDENT_CODE ON)
   ```

3. **MODULE 模块库**：运行时动态加载的插件库，仅通过 `dlopen`/`LoadLibrary` 加载，Windows 下无法被其他目标链接
   ```cmake
   add_library(my_plugin MODULE plugin.cpp)
   ```

4. **INTERFACE 头文件-only 库**：无编译源文件，不生成二进制文件，仅用于传递属性，专为纯模板/纯头文件库设计
   ```cmake
   add_library(my_header_only INTERFACE)
   # 仅传递属性给下游目标
   target_include_directories(my_header_only INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}/include)
   target_compile_features(my_header_only INTERFACE cxx_std_17)
   ```

5. **OBJECT 对象库**：仅编译源文件为目标文件（`.o`/`.obj`），不归档/链接，可被多个目标复用
   ```cmake
   add_library(my_obj OBJECT common.cpp)
   # 3.12+ 直接链接即可复用对象文件，无需 $<TARGET_OBJECTS>
   add_executable(app1 main1.cpp)
   target_link_libraries(app1 PRIVATE my_obj)

   add_executable(app2 main2.cpp)
   target_link_libraries(app2 PRIVATE my_obj)
   ```

### 3.3 `add_library ALIAS`：定义别名目标
给库目标添加命名空间别名，与 `find_package` 导出的目标格式统一，避免名称冲突，Modern CMake 强烈推荐使用。

**语法与示例**：
```cmake
add_library(MyDemo::Core ALIAS my_static)
add_library(MyDemo::HeaderOnly ALIAS my_header_only)

# 下游使用时与导入目标完全一致
target_link_libraries(my_app PRIVATE MyDemo::Core)
```

## 四、目标属性配置核心命令（target_* 系列）
这是 Modern CMake 的核心，所有配置均作用于指定目标，严格遵循作用域与传递性规则，**禁止使用全局替代命令**。

### 4.1 `target_include_directories`：指定头文件搜索路径
替代传统 `include_directories()`，指定目标的头文件搜索路径，支持构建/安装路径分离。

**完整语法**：
```cmake
target_include_directories(<target>
  <PRIVATE|PUBLIC|INTERFACE> <item>...
  [<PRIVATE|PUBLIC|INTERFACE> <item>... ...]
  [SYSTEM] [AFTER|BEFORE])
```

**关键参数与最佳实践示例**：
```cmake
target_include_directories(my_lib
  PUBLIC
    # 构建时使用源码路径，安装后使用安装路径，核心必写
    $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include>
  PRIVATE
    # 仅库内部使用的头文件路径，不传递给下游
    ${PROJECT_SOURCE_DIR}/src/internal
)
```
- `SYSTEM`：标记为系统头文件，抑制编译器警告，常用于第三方库头文件
- `AFTER/BEFORE`：指定路径追加/前置到搜索列表，默认 `AFTER`
- **核心避坑**：必须使用 `BUILD_INTERFACE`/`INSTALL_INTERFACE` 分离构建与安装路径，否则安装后下游项目无法找到头文件

### 4.2 `target_link_libraries`：指定链接依赖
替代传统 `link_libraries()`，是依赖传递的核心命令，**必须显式指定传递性关键字**。

**完整语法**：
```cmake
target_link_libraries(<target>
  <PRIVATE|PUBLIC|INTERFACE> <item>...
  [<PRIVATE|PUBLIC|INTERFACE> <item>... ...])
```

**关键说明与示例**：
`<item>` 支持以下类型，优先级从高到低：
1. **CMake 目标（强烈推荐）**：别名目标/导入目标，自动传递所有 PUBLIC/INTERFACE 属性（头文件路径、编译选项、依赖链）
   ```cmake
   # 链接本地库目标
   target_link_libraries(my_app PRIVATE MyDemo::Core)
   # 链接 find_package 导入的第三方目标
   find_package(Boost 1.74 REQUIRED COMPONENTS system thread)
   target_link_libraries(my_lib PUBLIC Boost::system Boost::thread)
   ```
2. 库文件完整路径（`find_library` 查找结果）
3. 系统库名（如 `pthread`、`m`）
4. 链接器标志

**核心规则**：
- 库内部实现使用的依赖用 `PRIVATE`，API 头文件中用到的依赖用 `PUBLIC`
- 禁止省略传递性关键字，否则会导致依赖传递混乱，出现「头文件找不到」「符号未定义」等疑难问题

### 4.3 `target_compile_definitions`：指定编译宏定义
替代传统 `add_definitions()`，指定目标的编译宏，无需加 `-D` 前缀，CMake 自动处理。

**语法与示例**：
```cmake
target_compile_definitions(my_lib
  PUBLIC
    # 下游也会继承该宏定义
    MY_LIB_VERSION=${PROJECT_VERSION}
    MY_LIB_ENABLE_FEATURE_A
  PRIVATE
    # 仅库内部使用的宏定义
    MY_LIB_BUILD
    $<$<CONFIG:Debug>:MY_LIB_DEBUG> # Debug 模式才启用的宏
)
```

### 4.4 `target_compile_options`：指定编译选项
替代手动设置 `CMAKE_CXX_FLAGS`，指定目标专属编译选项，配合生成器表达式实现跨编译器兼容。

**语法与跨平台示例**：
```cmake
target_compile_options(my_lib
  PRIVATE
    # GCC/Clang 编译器启用严格警告
    $<$<CXX_COMPILER_ID:GNU,Clang,AppleClang>:-Wall -Wextra -Wpedantic -Werror>
    # MSVC 编译器启用严格警告，禁用不安全函数警告
    $<$<CXX_COMPILER_ID:MSVC>:/W4 /WX /wd4996>
)
```

### 4.5 `target_compile_features`：指定 C/C++ 标准特性
替代全局标准设置，针对目标指定 C++ 标准，自动传递给下游目标，保证依赖链标准一致，跨编译器兼容性最佳。

**语法与示例**：
```cmake
# 强制目标及所有下游目标使用 C++17
target_compile_features(my_lib PUBLIC cxx_std_17)

# 指定具体语言特性，CMake 自动推导所需最低标准
target_compile_features(my_lib PRIVATE cxx_lambda_init_captures cxx_constexpr)
```

### 4.6 其他常用 target_* 命令
| 命令 | 最低版本 | 核心作用 | 示例 |
|------|----------|----------|------|
| `target_sources` | 3.13+ | 给目标追加源文件，支持子目录中给父目标添加文件 | `target_sources(my_lib PRIVATE submodule.cpp)` |
| `target_link_directories` | 3.13+ | 指定库搜索路径，**强烈不推荐**，优先使用导入目标 | `target_link_directories(my_lib PRIVATE /opt/xxx/lib)` |
| `target_link_options` | 3.13+ | 指定目标专属链接选项 | `target_link_options(my_app PRIVATE -Wl,--gc-sections)` |
| `target_precompile_headers` | 3.16+ | 预编译头文件，大幅提升编译速度 | `target_precompile_headers(my_lib PRIVATE <vector> <string> <memory>)` |
| `set_target_properties` | 3.0+ | 批量设置目标属性 | 见下文 |

### 4.7 `set_target_properties`：设置目标通用属性
批量设置一个或多个目标的属性，是目标属性配置的兜底方案。

**语法与常用示例**：
```cmake
set_target_properties(my_lib PROPERTIES
  # 库版本号（Linux 下 soname）
  VERSION ${PROJECT_VERSION}
  SOVERSION ${PROJECT_VERSION_MAJOR}
  # 输出文件名
  OUTPUT_NAME "mydemo_core"
  # 安装后的运行时库搜索路径
  INSTALL_RPATH "$ORIGIN/../lib"
  # macOS 下的安装名
  INSTALL_NAME_DIR "@rpath"
  # 禁止输出前缀（如 libxxx.so → xxx.so）
  PREFIX ""
)
```

**常用目标属性**：
| 属性名 | 作用 |
|--------|------|
| `CXX_STANDARD` | 目标专属 C++ 标准版本 |
| `CXX_STANDARD_REQUIRED` | 是否强制要求该标准 |
| `CXX_EXTENSIONS` | 是否启用编译器扩展 |
| `POSITION_INDEPENDENT_CODE` | 是否生成 PIC 位置无关代码 |
| `WIN32_EXECUTABLE` | 同 `add_executable` 的 `WIN32` 参数 |
| `MACOSX_BUNDLE` | 同 `add_executable` 的 `MACOSX_BUNDLE` 参数 |
| `EXCLUDE_FROM_ALL` | 是否排除在默认构建之外 |

## 五、变量、作用域与生成器表达式
### 5.1 变量类型与语法
CMake 变量均为字符串类型，区分大小写，引用格式为 `${变量名}`，`if()` 语句中可直接使用变量名无需 `${}`。

| 变量类型 | 定义语法 | 作用域与特性 |
|----------|----------|--------------|
| 普通变量 | `set(<var> <value>... [PARENT_SCOPE])` | 作用域为当前 CMakeLists.txt 及子目录，`PARENT_SCOPE` 可向上传递到父作用域 |
| 缓存变量 | `set(<var> <value>... CACHE <type> <doc> [FORCE])` | 全局生效，缓存到 `CMakeCache.txt`，可通过 `cmake -Dxxx=yyy` 命令行修改 |
| 环境变量 | `set(ENV{<var>} <value>)` | 仅当前 CMake 进程有效，读取格式为 `$ENV{<var>}` |
| 内置变量 | CMake 预定义 | 如 `CMAKE_SOURCE_DIR`（顶层源码目录）、`CMAKE_BINARY_DIR`（顶层构建目录）、`CMAKE_BUILD_TYPE`（构建类型） |

**常用示例**：
```cmake
# 普通变量
set(MY_SRC_FILES main.cpp utils.cpp)

# 缓存选项（用户可配置）
option(MY_DEMO_BUILD_TESTS "Build unit tests" ON)
set(MY_DEMO_MAX_THREADS 8 CACHE INT "Max thread count")

# 内置变量使用
if(CMAKE_BUILD_TYPE STREQUAL "Debug")
  message(STATUS "Debug build enabled")
endif()
```

### 5.2 生成器表达式（Generator Expressions）
Modern CMake 跨平台、跨配置适配的核心工具，**在构建系统生成阶段（而非 CMake 配置阶段）求值**，可精准区分编译器、构建类型、目标平台、编译语言等场景。

**核心格式**：
- 条件表达式：`$<condition:true_value>` / `$<condition:true_value:false_value>`
- 信息表达式：`$<KEYWORD>`，返回对应信息

**最常用生成器表达式**：
| 表达式 | 作用 |
|--------|------|
| `$<BUILD_INTERFACE:path>` | 构建阶段生效，安装阶段失效 |
| `$<INSTALL_INTERFACE:path>` | 安装阶段生效，构建阶段失效 |
| `$<CXX_COMPILER_ID:id>` | 编译器匹配时返回 1，如 `GNU`/`Clang`/`MSVC` |
| `$<CONFIG:cfg>` | 构建类型匹配时返回 1，如 `Debug`/`Release` |
| `$<PLATFORM_ID:id>` | 平台匹配时返回 1，如 `Linux`/`Windows`/`Darwin` |
| `$<COMPILE_LANGUAGE:lang>` | 编译语言匹配时返回 1，如 `CXX`/`C` |
| `$<AND:a,b>` | 逻辑与，a 和 b 均为 1 时返回 1 |
| `$<OR:a,b>` | 逻辑或，a 或 b 为 1 时返回 1 |
| `$<NOT:a>` | 逻辑非，a 为 0 时返回 1 |

**综合示例**：
```cmake
# 仅 Debug 模式启用 DEBUG 宏
target_compile_definitions(my_lib PRIVATE $<$<CONFIG:Debug>:DEBUG>)

# 仅对 C++ 代码启用旧类型转换警告
target_compile_options(my_lib PRIVATE $<$<COMPILE_LANGUAGE:CXX>:-Wold-style-cast>)

# 仅 Linux 平台链接 pthread 库
target_link_libraries(my_lib PRIVATE $<$<PLATFORM_ID:Linux>:pthread>)
```

## 六、流程控制语法
### 6.1 条件判断：`if()/elseif()/else()/endif()`
**核心语法规则**：
- 变量引用无需 `${}`，直接写变量名即可
- 字符串比较需用双引号包裹
- 支持版本比较专用关键字：`VERSION_LESS`/`VERSION_GREATER`/`VERSION_GREATER_EQUAL`

**常用示例**：
```cmake
# 判断目标是否存在
if(TARGET MyDemo::Core)
  message(STATUS "MyDemo::Core already exists")
endif()

# 编译器版本判断
if(CMAKE_CXX_COMPILER_ID STREQUAL "GNU" AND CMAKE_CXX_COMPILER_VERSION VERSION_GREATER_EQUAL 9.0)
  target_compile_options(my_lib PRIVATE -fconcepts)
endif()

# 平台判断
if(WIN32)
  target_compile_definitions(my_lib PRIVATE MY_LIB_WINDOWS)
elseif(APPLE)
  target_compile_definitions(my_lib PRIVATE MY_LIB_MACOS)
elseif(UNIX)
  target_compile_definitions(my_lib PRIVATE MY_LIB_LINUX)
endif()
```

### 6.2 循环：`foreach()/endforeach()`
**常用语法与示例**：
```cmake
# 基础列表循环
foreach(file main.cpp utils.cpp module.cpp)
  message(STATUS "Processing file: ${file}")
endforeach()

# 范围循环
foreach(i RANGE 1 10 2)
  message(STATUS "Number: ${i}")
endforeach()

# 遍历子目录
foreach(subdir core net utils)
  add_subdirectory(${subdir})
endforeach()
```

### 6.3 函数与宏：`function()/macro()`
Modern CMake **强烈推荐使用 function，禁止滥用 macro**，function 有独立局部作用域，macro 是文本替换，无作用域隔离，极易引发副作用。

**function 语法与示例**：
```cmake
# 定义函数
function(add_my_executable name)
  # ARGN：超出命名参数的所有入参
  add_executable(${name} ${ARGN})
  # 统一配置
  target_compile_features(${name} PRIVATE cxx_std_17)
  target_compile_options(${name} PRIVATE -Wall -Wextra)
  target_link_libraries(${name} PRIVATE MyDemo::Core)
endfunction()

# 使用函数
add_my_executable(app1 main1.cpp)
add_my_executable(app2 main2.cpp module.cpp)
```

**function 与 macro 核心区别**：
| 特性 | function | macro |
|------|----------|-------|
| 作用域 | 有独立局部作用域，变量默认局部 | 无作用域，变量直接修改父作用域 |
| 参数处理 | 实参传递，支持参数校验 | 文本替换，参数无类型校验 |
| 返回值 | 通过 `PARENT_SCOPE` 设置父作用域变量 | 直接修改父作用域变量 |

## 七、依赖管理
Modern CMake 提供两套标准化依赖管理方案，彻底告别手动找头文件、手动链接库的原始写法。

### 7.1 `find_package`：查找已安装依赖
CMake 最主流的依赖查找方案，分为 **Module 模式** 和 **Config 模式**，Modern CMake 优先使用 Config 模式。

**完整语法**：
```cmake
find_package(<PackageName> [version] [EXACT] [QUIET] [MODULE]
             [REQUIRED] [[COMPONENTS] [components...]]
             [OPTIONAL_COMPONENTS components...])
```

**关键参数说明**：
- `REQUIRED`：必须找到该依赖，找不到直接终止配置
- `COMPONENTS`：指定需要加载的组件
- `EXACT`：必须匹配精确的版本号
- `QUIET`：静默模式，不输出查找日志
- `MODULE`：强制使用 Module 模式，默认优先 Config 模式

**Config 模式（推荐）**：
依赖库的作者提供 `<PackageName>Config.cmake` 配置文件，里面导出了完整的 CMake 目标，包含头文件路径、链接库、编译选项等所有信息，直接链接目标即可，零配置成本。

**示例**：
```cmake
# 查找 Qt6
find_package(Qt6 6.5 REQUIRED COMPONENTS Core Widgets)
# 直接链接导入目标，自动处理所有依赖
target_link_libraries(my_app PRIVATE Qt6::Core Qt6::Widgets)

# 查找 OpenCV
find_package(OpenCV 4.0 REQUIRED)
target_link_libraries(my_app PRIVATE ${OpenCV_LIBS}) # 兼容旧写法，优先用导入目标
```

### 7.2 `FetchContent`：源码级依赖管理
3.11 引入，3.14 完善，支持**配置阶段下载依赖源码**，直接加入当前项目一起编译，完美支持 Modern CMake 目标传递，替代传统的 `ExternalProject`（构建时下载，不支持目标传递）。

**完整流程与示例**：
```cmake
# 引入内置模块
include(FetchContent)

# 声明依赖：指定名称、源码地址、版本/提交号
FetchContent_Declare(
  spdlog
  GIT_REPOSITORY https://github.com/gabime/spdlog.git
  GIT_TAG        v1.14.1
  GIT_SHALLOW    TRUE # 浅克隆，加速下载
)

# 下载、配置、添加到当前项目
FetchContent_MakeAvailable(spdlog)

# 直接链接目标，与本地库完全一致
target_link_libraries(my_app PRIVATE spdlog::spdlog)
```

**支持的源码类型**：
- Git 仓库（`GIT_REPOSITORY`）
- 压缩包 URL（`URL`），支持 `URL_HASH` 校验完整性
- 本地源码目录（`SOURCE_DIR`）

### 7.3 `add_subdirectory`：添加子目录
用于项目内模块划分，或本地存放的第三方依赖源码，子目录需包含独立的 CMakeLists.txt。

**语法与示例**：
```cmake
# 添加子模块
add_subdirectory(core)
add_subdirectory(net)

# 添加第三方源码，EXCLUDE_FROM_ALL 避免其测试/示例被默认构建
add_subdirectory(thirdparty/fmt EXCLUDE_FROM_ALL)
```

## 八、安装与导出
Modern CMake 提供标准化的安装与导出流程，让你的库可以被其他项目通过 `find_package` 无缝使用，与第三方库完全一致。

### 8.1 核心前置模块
```cmake
# 提供标准安装目录变量，避免硬编码
include(GNUInstallDirs)
# 生成 Config.cmake 和 ConfigVersion.cmake 文件
include(CMakePackageConfigHelpers)
```

### 8.2 `install`：安装目标与文件
#### 8.2.1 安装目标（核心）
```cmake
install(TARGETS my_lib my_app
  # 导出目标到 my_demo_targets 集合，后续用于生成配置文件
  EXPORT my_demo_targets
  # 可执行文件/Windows DLL 安装路径
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
  # Linux so/macOS dylib 安装路径
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
  # 静态库/Windows 导入库安装路径
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
  # 头文件安装路径，自动添加到目标的 INTERFACE_INCLUDE_DIRECTORIES
  INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)
```

#### 8.2.2 安装头文件
```cmake
# 安装 include 目录下所有头文件，保留目录结构
install(DIRECTORY ${PROJECT_SOURCE_DIR}/include/
  DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
  # 仅安装 .h/.hpp 文件
  FILES_MATCHING PATTERN "*.h" PATTERN "*.hpp"
)
```

#### 8.2.3 导出目标配置文件
```cmake
# 安装导出的目标，生成 my_demoTargets.cmake 文件
install(EXPORT my_demo_targets
  FILE my_demoTargets.cmake
  # 添加命名空间，与别名目标一致
  NAMESPACE MyDemo::
  # 安装路径，标准 cmake 配置文件目录
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/my_demo
)
```

### 8.3 生成包配置文件
#### 8.3.1 生成版本配置文件
```cmake
write_basic_package_version_file(
  ${PROJECT_BINARY_DIR}/my_demoConfigVersion.cmake
  VERSION ${PROJECT_VERSION}
  # 版本兼容性：同主版本号兼容
  COMPATIBILITY SameMajorVersion
)
```

#### 8.3.2 生成主配置文件
1. 新建模板文件 `cmake/my_demoConfig.cmake.in`：
   ```cmake
   @PACKAGE_INIT@
   # 包含目标导出文件
   include("${CMAKE_CURRENT_LIST_DIR}/my_demoTargets.cmake")
   ```

2. 配置并安装配置文件：
   ```cmake
   configure_package_config_file(
     ${PROJECT_SOURCE_DIR}/cmake/my_demoConfig.cmake.in
     ${PROJECT_BINARY_DIR}/my_demoConfig.cmake
     INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/my_demo
   )

   # 安装配置文件
   install(FILES
     ${PROJECT_BINARY_DIR}/my_demoConfig.cmake
     ${PROJECT_BINARY_DIR}/my_demoConfigVersion.cmake
     DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/my_demo
   )
   ```

完成以上配置后，安装的库可被其他项目通过 `find_package(MyDemo REQUIRED)` 直接找到，链接 `MyDemo::my_lib` 目标即可使用。

## 九、最佳实践与避坑指南
### 9.1 核心最佳实践
1. **始终基于目标编程**：所有配置使用 `target_*` 命令，绝对禁止使用全局的 `include_directories`/`link_directories`/`add_definitions` 等命令
2. **显式指定传递性**：所有 `target_*` 命令必须显式指定 `PRIVATE/PUBLIC/INTERFACE`，禁止省略
3. **优先使用导入目标**：第三方依赖始终链接 `find_package`/`FetchContent` 提供的导入目标，不手动处理头文件和库路径
4. **命名空间别名**：所有库目标必须添加 `xxx::yyy` 格式的别名，与导出目标格式统一
5. **显式列出源文件**：禁止用 `file(GLOB)` 收集源文件，除非使用 `CONFIGURE_DEPENDS`，避免新增文件后构建失效
6. **跨平台适配用生成器表达式**：替代大量 `if-else` 平台判断，代码更简洁、可维护性更强
7. **禁用编译器扩展**：设置 `CMAKE_CXX_EXTENSIONS OFF`，保证跨编译器兼容性
8. **最小作用域原则**：能用 `PRIVATE` 就不用 `PUBLIC`，能用局部变量就不用全局变量/缓存变量

### 9.2 高频避坑点
1. **省略传递性关键字**：导致依赖传递混乱，出现「头文件找不到」「符号未定义」等问题
2. **混用构建/安装路径**：未使用 `BUILD_INTERFACE`/`INSTALL_INTERFACE`，导致安装后下游项目无法找到头文件
3. **全局配置污染**：使用全局命令修改所有目标的配置，引发冲突和难以调试的编译问题
4. **静态库未开启 PIC**：静态库被动态库链接时，出现「重定位错误」，需设置 `POSITION_INDEPENDENT_CODE ON`
5. **`CMAKE_BUILD_TYPE` 失效**：该变量仅对单配置生成器（Makefile/Ninja）生效，多配置生成器（Visual Studio/Xcode）使用 `CMAKE_CONFIGURATION_TYPES`
6. **滥用 `file(GLOB)`**：新增源文件后 CMake 无法自动感知，必须手动重新配置，仅推荐使用 `file(GLOB CONFIGURE_DEPENDS)` 用于头文件-only 库
7. **宏替代函数**：使用 `macro` 导致变量污染父作用域，引发难以排查的副作用
8. **硬编码路径**：使用绝对路径导致项目无法迁移，始终使用 `${PROJECT_SOURCE_DIR}`/`${CMAKE_CURRENT_SOURCE_DIR}` 等相对路径变量


