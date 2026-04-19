# CMake 语法详细总结


# CMake 语法详细总结
CMake 是跨平台的构建系统生成器，核心是**基于目标（Target）**的声明式语法，本文从基础到进阶全面梳理，重点突出现代CMake（3.15+ 推荐）的最佳实践，同时覆盖老项目常用语法与避坑指南。

## 一、基础语法核心
### 1.1 基本规则
- **命令不区分大小写**，但**变量名、参数、路径、目标名严格区分大小写**（Linux/macOS 路径大小写敏感，Windows 不敏感，建议统一规范）。
- 命令格式：`命令名(参数1 参数2 ...)`，参数用空格/分号分隔，带空格的参数必须用双引号包裹。
- 脚本入口：每个项目的顶层必须有 `CMakeLists.txt` 文件，子目录可通过 `add_subdirectory()` 引入。

### 1.2 注释
```cmake
# 单行注释，# 后到行尾均为注释

#[[ 块注释
支持多行、嵌套，3.0+ 版本支持
#[[ 嵌套块注释 ]]
]]
```

### 1.3 变量体系
CMake 变量分为三类：普通变量、缓存变量、环境变量，本质均为字符串，多值变量以分号分隔的列表存储。

#### 1.3.1 普通变量
- 定义/取消：
  ```cmake
  # 定义：set(变量名 值1 值2 ...)
  set(MY_VAR "hello world")
  set(MY_LIST a b c d) # 等价于 set(MY_LIST "a;b;c;d")
  # 取消定义
  unset(MY_VAR)
  ```
- 引用：`${变量名}`，在双引号内会自动展开，单引号不支持转义和展开。
- 作用域：
  - 函数、`add_subdirectory()` 会创建独立的子作用域。
  - 子作用域可读取父作用域变量，修改父作用域变量需加 `PARENT_SCOPE`：
    ```cmake
    set(PARENT_VAR "origin" PARENT_SCOPE) # 修改父作用域变量
    ```

#### 1.3.2 缓存变量（Cache）
缓存变量存储在 `CMakeCache.txt` 中，全局生效，cmake-gui/ccmake 可可视化修改，持久化跨配置运行。
```cmake
# 定义：set(变量名 值 CACHE 类型 "描述" [FORCE])
set(MY_CACHE_VAR "default" CACHE STRING "这是一个缓存变量" FORCE)

# 简化版：BOOL 类型缓存变量，用于开关
option(ENABLE_DEBUG "开启调试模式" OFF)
```
- 类型说明：`BOOL`（布尔）、`STRING`（字符串）、`FILEPATH`（文件路径）、`PATH`（目录路径）、`INTERNAL`（内部变量，不显示在GUI）。
- 引用：普通变量会覆盖缓存变量的同名引用，强制引用缓存变量用 `$CACHE{变量名}`。

#### 1.3.3 环境变量
仅在当前 CMake 进程内生效，不修改系统环境变量。
```cmake
# 引用
$ENV{PATH}
# 设置
set(ENV{MY_ENV_VAR} "value")
```

### 1.4 列表操作
CMake 多值变量本质是列表，通过 `list()` 命令操作，常用子命令：
```cmake
# 追加元素
list(APPEND MY_LIST e f)
# 移除元素
list(REMOVE_ITEM MY_LIST a)
list(REMOVE_AT MY_LIST 0)
# 列表长度
list(LENGTH MY_LIST LIST_LEN)
# 查找元素索引（找不到返回-1）
list(FIND MY_LIST b INDEX)
# 截取/获取元素
list(GET MY_LIST 1 2 ELEMENTS)
# 排序
list(SORT MY_LIST)
# 去重
list(REMOVE_DUPLICATES MY_LIST)
```

## 二、流程控制
### 2.1 条件判断 `if()/elseif()/else()/endif()`
#### 核心规则
- 基本语法：`if(表达式)`，表达式内变量可省略 `${}`，但字符串比较必须加双引号避免解析歧义。
- 布尔真值：`ON/TRUE/YES/1/非空字符串`；假值：`OFF/FALSE/NO/0/空字符串/以-NOTFOUND结尾的字符串`，不区分大小写。

#### 常用运算符
| 分类 | 运算符 | 说明 |
|------|--------|------|
| 逻辑运算 | `AND/OR/NOT` | 逻辑与、或、非 |
| 数值比较 | `EQUAL/LESS/GREATER/LESS_EQUAL/GREATER_EQUAL` | 数字相等、小于、大于等 |
| 字符串比较 | `STREQUAL/STRLESS/STRGREATER` | 字符串相等、字典序小于/大于 |
| 版本比较 | `VERSION_LESS/VERSION_GREATER/VERSION_EQUAL` | 语义化版本号比较 |
| 文件系统 | `EXISTS/IS_DIRECTORY/IS_SYMLINK/IS_ABSOLUTE` | 路径存在/是目录/是软链接/是绝对路径 |
| 存在判断 | `DEFINED 变量名` | 变量是否定义 |
| 命令判断 | `COMMAND 命令名` | 命令/函数/宏是否存在 |

#### 示例与避坑
```cmake
# 正确：字符串比较加双引号，避免变量值为常量时解析错误
if("${MY_VAR}" STREQUAL "debug")
  message(STATUS "调试模式")
elseif(${VERSION} VERSION_GREATER "1.0.0")
  message(STATUS "版本大于1.0.0")
else()
  message(STATUS "默认模式")
endif()

# 避坑：if 内不要用生成器表达式（配置阶段执行，生成器表达式构建阶段才求值）
```

### 2.2 循环 `foreach()/endforeach()`、`while()/endwhile()`
#### foreach 循环（最常用）
```cmake
# 1. 遍历列表（推荐写法，避免空元素解析问题）
foreach(item IN LISTS MY_LIST)
  message(STATUS "元素: ${item}")
endforeach()

# 2. 范围遍历
foreach(i RANGE 1 10 2) # 1到10，步长2
  message(STATUS "数字: ${i}")
endforeach()

# 3. 直接遍历元素
foreach(item IN ITEMS a b c d)
  message(STATUS "元素: ${item}")
endforeach()
```

#### while 循环
```cmake
set(COUNT 0)
while(${COUNT} LESS 5)
  message(STATUS "计数: ${COUNT}")
  math(EXPR COUNT "${COUNT} + 1") # 数学运算
endwhile()
```

#### 循环控制
- `break()`：跳出整个循环
- `continue()`：跳过当前迭代，进入下一次循环

### 2.3 函数与宏
#### 核心区别
| 特性 | 函数 `function()` | 宏 `macro()` |
|------|-------------------|--------------|
| 作用域 | 有独立作用域，参数为局部变量 | 无独立作用域，纯文本替换，在调用处展开 |
| 变量修改 | 默认仅函数内生效，修改父作用域需 `PARENT_SCOPE` | 直接修改调用者作用域的变量 |
| 执行时机 | 调用时执行，有函数调用栈 | 预编译期文本替换，无调用栈 |
| 推荐场景 | 绝大多数场景，无副作用 | 仅简单文本替换，谨慎使用 |

#### 函数语法
```cmake
# 定义
function(my_func param1 param2)
  # ARGC: 参数总数；ARGV: 所有参数列表；ARGN: 超出定义的额外参数
  message(STATUS "参数1: ${param1}, 参数2: ${param2}")
  message(STATUS "额外参数: ${ARGN}")
  # 修改父作用域变量
  set(RETURN_VAL "func_result" PARENT_SCOPE)
endfunction()

# 调用
my_func(a b c d)
message(STATUS "函数返回: ${RETURN_VAL}")
```

#### 宏语法
```cmake
# 定义
macro(my_macro param1 param2)
  message(STATUS "宏参数1: ${param1}")
  # 直接修改调用者作用域的变量
  set(MACRO_VAR "macro_result")
endmacro()

# 调用
my_macro(x y)
message(STATUS "宏变量: ${MACRO_VAR}")
```

## 三、核心：项目与目标管理（现代CMake核心）
现代CMake 全程围绕**目标**编程，所有编译、链接、依赖规则都绑定到目标，精准控制传播范围，避免全局污染。

### 3.1 入口必备命令
#### 3.1.1 `cmake_minimum_required()`
必须放在 CMakeLists.txt 的**第一行**，指定最低CMake版本，低于版本直接报错。
```cmake
cmake_minimum_required(VERSION 3.15 FATAL_ERROR)
# 推荐3.15+，兼容绝大多数现代特性；新项目可直接用3.20+
```

#### 3.1.2 `project()`
定义项目名称、版本、语言等信息，初始化编译环境，自动生成一批项目相关内置变量。
```cmake
project(
  MyProject
  VERSION 1.0.0 # 主版本.次版本.补丁版本.微调版本
  DESCRIPTION "CMake示例项目"
  HOMEPAGE_URL "https://github.com/xxx/MyProject"
  LANGUAGES CXX C # 启用的语言，默认C/CXX，无需编译可设为NONE
)
```
自动生成的核心变量：`PROJECT_NAME`、`PROJECT_VERSION`、`PROJECT_SOURCE_DIR`（项目源码目录）、`PROJECT_BINARY_DIR`（项目构建目录）。

### 3.2 目标定义
CMake 目标分为三大类：可执行目标、库目标、自定义目标，目标名全局唯一。

#### 3.2.1 可执行目标 `add_executable()`
```cmake
add_executable(
  my_app # 目标名
  [WIN32] # Windows下生成GUI程序，入口为WinMain而非main
  [MACOSX_BUNDLE] # macOS下生成.app应用包
  src/main.cpp
  src/app.cpp
  # 源文件列表
)
```

#### 3.2.2 库目标 `add_library()`
核心库类型说明：
| 类型 | 说明 | 生成文件 |
|------|------|----------|
| `STATIC` | 静态库 | Linux/macOS: `.a`；Windows: `.lib` |
| `SHARED` | 动态/共享库 | Linux: `.so`；macOS: `.dylib`；Windows: `.dll+.lib` |
| `MODULE` | 模块库 | 运行时动态加载（dlopen），Windows下无导入库 |
| `OBJECT` | 对象库 | 仅编译不链接，生成 `.o/.obj`，可被其他目标复用 |
| `INTERFACE` | 纯接口库 | 无源文件，用于头文件-only库，传递编译/链接规则 |
| `IMPORTED` | 导入库 | 引用系统预编译的第三方库，无需重新编译 |
| `ALIAS` | 别名库 | 给已有目标起别名，通常用于添加命名空间 |

```cmake
# 1. 静态库
add_library(my_static STATIC src/lib.cpp)

# 2. 动态库
add_library(my_shared SHARED src/lib.cpp)

# 3. 对象库
add_library(my_obj OBJECT src/utils.cpp)
# 3.12+ 可直接链接对象库，无需 $<TARGET_OBJECTS:my_obj>
target_link_libraries(my_static PRIVATE my_obj)

# 4. 纯接口库（头文件-only）
add_library(my_header_only INTERFACE)
target_include_directories(my_header_only INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}/include)

# 5. 别名库（命名空间规范，现代CMake推荐）
add_library(MyProject::my_static ALIAS my_static)
add_library(MyProject::my_header_only ALIAS my_header_only)

# 6. 导入预编译库
add_library(third_party SHARED IMPORTED)
set_target_properties(third_party PROPERTIES
  IMPORTED_LOCATION ${CMAKE_CURRENT_SOURCE_DIR}/lib/libthird_party.so
  INTERFACE_INCLUDE_DIRECTORIES ${CMAKE_CURRENT_SOURCE_DIR}/include
)
```

### 3.3 目标属性与传播范围（重中之重）
通过 `target_*` 系列命令给目标设置属性，通过 `PRIVATE/PUBLIC/INTERFACE` 三个关键字精准控制属性的传播范围，这是现代CMake的核心。

#### 传播范围关键字说明
| 关键字 | 作用范围 | 典型场景 |
|--------|----------|----------|
| `PRIVATE` | 仅作用于当前目标，不传递给依赖它的其他目标 | 当前目标源文件编译需要的头文件、内部警告选项 |
| `INTERFACE` | 不作用于当前目标，仅传递给依赖它的目标 | 头文件-only库的头文件路径、API依赖的编译宏 |
| `PUBLIC` | 既作用于当前目标，也传递给依赖它的目标 | 头文件中包含的第三方库头文件、C++标准特性 |

#### 核心 target_* 命令
```cmake
# 1. 设置头文件包含路径（替代全局 include_directories）
target_include_directories(my_static
  PUBLIC
    $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/include> # 构建时路径
    $<INSTALL_INTERFACE:include> # 安装后路径
  PRIVATE
    ${PROJECT_SOURCE_DIR}/src # 仅内部使用的头文件路径
)

# 2. 设置编译宏定义（替代全局 add_definitions）
target_compile_definitions(my_static
  PRIVATE MY_LIB_INTERNAL # 内部宏
  PUBLIC MY_LIB_ENABLED # 传递给依赖者的宏
)

# 3. 设置编译选项（替代全局 add_compile_options）
target_compile_options(my_static
  PRIVATE
    # 仅当前目标启用的警告选项，不污染依赖者
    $<$<CXX_COMPILER_ID:GNU,Clang>:-Wall -Wextra -Wpedantic>
    $<$<CXX_COMPILER_ID:MSVC>:/W4>
)

# 4. 设置C/C++标准特性（比全局设置更优，自动传递）
target_compile_features(my_static PUBLIC cxx_std_17)

# 5. 链接依赖库（最常用）
target_link_libraries(my_app
  PRIVATE my_static # 私有依赖，不传递
  PUBLIC Threads::Threads # 公共依赖，传递给依赖my_app的目标
)

# 6. 设置链接选项（替代全局 add_link_options）
target_link_options(my_shared PRIVATE -Wl,--no-undefined)

# 7. 给目标追加源文件（3.13+）
target_sources(my_static PRIVATE src/extra.cpp)
```

#### 通用属性操作
```cmake
# 批量设置目标属性
set_target_properties(my_shared PROPERTIES
  OUTPUT_NAME "my_lib" # 生成的库文件名，默认是目标名
  VERSION ${PROJECT_VERSION} # 库版本号
  SOVERSION 1 # API版本号（soname）
  POSITION_INDEPENDENT_CODE ON # 生成PIC位置无关代码
  CXX_STANDARD 17 # C++标准
  CXX_STANDARD_REQUIRED ON # 强制要求标准
  CXX_EXTENSIONS OFF # 禁用编译器扩展，保证跨平台
)

# 获取目标属性
get_target_property(LIB_OUTPUT_NAME my_shared OUTPUT_NAME)
message(STATUS "库输出名: ${LIB_OUTPUT_NAME}")
```

## 四、依赖与链接管理
### 4.1 `find_package()`：查找第三方库
CMake 最核心的依赖查找命令，支持两种模式，成功后会提供导入目标或头文件/库路径变量。
```cmake
# 基础语法
find_package(
  包名
  [版本号] # 最低版本要求
  [REQUIRED] # 找不到直接报错终止
  [QUIET] # 不输出查找日志
  [COMPONENTS 组件1 组件2] # 必须的组件
  [OPTIONAL_COMPONENTS 组件3] # 可选组件
  [CONFIG/NO_MODULE] # 强制使用Config模式
)
```

#### 两种查找模式
1. **Module模式**：查找 `Find<包名>.cmake` 模块，先在 `CMAKE_MODULE_PATH` 中查找，再查找CMake内置模块。CMake内置了大量常用库的Find模块，如 `FindThreads.cmake`、`FindZLIB.cmake`、`FindOpenGL.cmake` 等。
2. **Config模式**：查找 `<包名>Config.cmake` 或 `<小写包名>-config.cmake` 文件，由第三方库安装时自动生成，包含完整的目标定义和依赖规则，比Module模式更可靠，现代CMake库均提供该文件。

#### 最佳实践
优先使用库提供的**导入目标**，无需手动设置头文件和库路径，自动传递所有依赖：
```cmake
# 示例1：查找ZLIB
find_package(ZLIB REQUIRED)
target_link_libraries(my_app PRIVATE ZLIB::ZLIB)

# 示例2：查找Qt6
find_package(Qt6 REQUIRED COMPONENTS Core Widgets)
target_link_libraries(my_app PRIVATE Qt6::Core Qt6::Widgets)
```

### 4.2 `FetchContent`：配置阶段拉取依赖
3.11+ 版本支持，在CMake配置阶段就拉取第三方库源码，和主项目一起编译，完美兼容 `find_package()`，替代老旧的 `ExternalProject`（构建阶段拉取）。
```cmake
include(FetchContent)
# 声明依赖
FetchContent_Declare(
  spdlog
  GIT_REPOSITORY https://github.com/gabime/spdlog.git
  GIT_TAG v1.14.1 # 指定版本tag/commit
  # 也可使用URL下载压缩包
  # URL https://github.com/gabime/spdlog/archive/refs/tags/v1.14.1.tar.gz
  # URL_HASH SHA256=xxx
)
# 拉取并添加到项目
FetchContent_MakeAvailable(spdlog)

# 直接链接导入目标
target_link_libraries(my_app PRIVATE spdlog::spdlog)
```

### 4.3 其他依赖查找方式
1. **pkg-config 支持**：用于查找Linux系统中pkg-config管理的库
   ```cmake
   include(FindPkgConfig)
   pkg_check_modules(LIBUSB REQUIRED libusb-1.0)
   target_include_directories(my_app PRIVATE ${LIBUSB_INCLUDE_DIRS})
   target_link_libraries(my_app PRIVATE ${LIBUSB_LIBRARIES})
   ```
2. **手动链接库文件**：直接指定库的绝对路径，不推荐跨平台场景使用
   ```cmake
   target_link_libraries(my_app PRIVATE /usr/local/lib/libxxx.a)
   ```

## 五、全局编译与构建配置
> 注意：现代CMake推荐用目标级别的配置替代全局配置，避免全局污染，此处仅说明老项目常用的全局配置。

### 5.1 构建类型配置
单配置生成器（Unix Makefiles、Ninja）通过 `CMAKE_BUILD_TYPE` 设置，多配置生成器（Visual Studio、Xcode）通过 `CMAKE_CONFIGURATION_TYPES` 设置。
```cmake
# 设置构建类型，必须在cmake命令行或project()前设置
# 可选值：Debug/Release/RelWithDebInfo/MinSizeRel
set(CMAKE_BUILD_TYPE Release CACHE STRING "构建类型")
```
| 构建类型 | 说明 | 典型编译选项 |
|----------|------|--------------|
| Debug | 调试版 | 无优化，带完整调试信息 `-O0 -g` |
| Release | 发布版 | 最高优化，无调试信息 `-O3` |
| RelWithDebInfo | 带调试信息的发布版 | 中等优化，带调试信息 `-O2 -g`，生产环境常用 |
| MinSizeRel | 最小体积发布版 | 优化体积 `-Os`，嵌入式场景常用 |

> 避坑：不设置 `CMAKE_BUILD_TYPE` 时，默认是无优化、无调试信息的空配置，既不是Debug也不是Release。

### 5.2 全局语言标准配置
```cmake
# 全局C++标准，目标级配置会覆盖该值
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON) # 不支持该标准直接报错
set(CMAKE_CXX_EXTENSIONS OFF) # 禁用GNU等编译器扩展，保证跨平台

# 对应C语言标准
set(CMAKE_C_STANDARD 11)
set(CMAKE_C_STANDARD_REQUIRED ON)
set(CMAKE_C_EXTENSIONS OFF)
```

### 5.3 输出路径配置
```cmake
# 全局输出目录
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/bin) # 可执行文件+Windows dll
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/lib) # 共享库
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/lib) # 静态库

# 按配置分目录
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY_DEBUG ${PROJECT_BINARY_DIR}/debug/bin)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY_RELEASE ${PROJECT_BINARY_DIR}/release/bin)
```

### 5.4 全局编译/链接选项
```cmake
# 全局添加编译选项，所有目标都会继承
add_compile_options(-Wall -Wextra)

# 全局添加链接选项
add_link_options(-Wl,--as-needed)

# 全局添加编译宏
add_definitions(-DGLOBAL_ENABLE=1)

# 全局添加头文件路径（强烈不推荐，污染所有子目录）
include_directories(${PROJECT_SOURCE_DIR}/include)

# 全局添加库搜索路径（强烈不推荐，极易出现链接错误）
link_directories(${PROJECT_SOURCE_DIR}/lib)
```

### 5.5 编译器相关变量
```cmake
# 指定编译器，必须在project()前设置，或通过命令行 -DCMAKE_CXX_COMPILER=xxx 指定
set(CMAKE_C_COMPILER gcc)
set(CMAKE_CXX_COMPILER g++)

# 全局编译标志，所有构建类型生效
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -pthread")
# 分构建类型的编译标志
set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -O0 -g")
set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -O3 -DNDEBUG")
```

## 六、常用内置命令与模块
### 6.1 文件操作 `file()`
CMake 最强大的命令之一，支持文件读写、复制、下载、匹配等操作。
```cmake
# 1. 匹配文件（避坑：GLOB不会自动检测新增文件，需重新运行cmake）
file(GLOB SRC_FILES src/*.cpp) # 匹配当前目录cpp文件
file(GLOB_RECURSE ALL_SRC src/*.cpp include/*.h) # 递归匹配
# 3.12+ 新增CONFIGURE_DEPENDS，构建阶段自动检测文件变化
file(GLOB SRC_FILES CONFIGURE_DEPENDS src/*.cpp)

# 2. 文件读写
file(WRITE test.txt "hello cmake") # 覆盖写入
file(APPEND test.txt "\nnew line") # 追加写入
file(READ test.txt FILE_CONTENT) # 读取内容到变量

# 3. 文件/目录复制
file(COPY src/ DESTINATION ${PROJECT_BINARY_DIR}/src)
file(MAKE_DIRECTORY ${PROJECT_BINARY_DIR}/temp) # 创建目录

# 4. 删除文件/目录
file(REMOVE test.txt)
file(REMOVE_RECURSE ${PROJECT_BINARY_DIR}/temp)

# 5. 下载文件
file(DOWNLOAD
  https://example.com/file.tar.gz
  ${PROJECT_BINARY_DIR}/file.tar.gz
  SHOW_PROGRESS
  EXPECTED_HASH SHA256=xxx
)
```

### 6.2 配置文件生成 `configure_file()`
将输入文件中的 `@VAR@` 或 `${VAR}` 替换为CMake中的变量值，生成输出文件，常用于生成版本头文件、配置文件。
```cmake
# 语法：configure_file(输入.in 输出文件 [@ONLY] [COPYONLY])
configure_file(
  ${PROJECT_SOURCE_DIR}/version.h.in
  ${PROJECT_BINARY_DIR}/version.h
  @ONLY # 仅替换@VAR@，避免和脚本中的${}冲突，推荐使用
)
```
示例 `version.h.in`：
```c
#define PROJECT_NAME "@PROJECT_NAME@"
#define PROJECT_VERSION "@PROJECT_VERSION@"
#define DEBUG_MODE @ENABLE_DEBUG@
```

### 6.3 安装命令 `install()`
用于 `make install`/`ninja install` 时安装编译产物、头文件、文档等，支持多平台标准路径。
```cmake
# 引入GNU标准安装目录，兼容多平台
include(GNUInstallDirs)

# 1. 安装目标（可执行文件/库）
install(TARGETS my_app my_static my_shared
  EXPORT MyProjectTargets # 导出目标，用于生成Config文件
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR} # 可执行文件
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR} # 共享库
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR} # 静态库
  INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR} # 头文件路径
)

# 2. 安装头文件
install(DIRECTORY ${PROJECT_SOURCE_DIR}/include/
  DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
  FILES_MATCHING PATTERN "*.h" "*.hpp" # 仅安装头文件
  PATTERN "private" EXCLUDE # 排除私有目录
)

# 3. 安装单个文件
install(FILES LICENSE README.md DESTINATION ${CMAKE_INSTALL_DOCDIR})

# 4. 安装时执行脚本/代码
install(CODE "message(STATUS \"安装完成\")")
install(SCRIPT ${PROJECT_SOURCE_DIR}/cmake/install_script.cmake)
```
> 安装前缀：通过 `CMAKE_INSTALL_PREFIX` 设置，默认Unix下为 `/usr/local`，Windows下为 `C:/Program Files/项目名`，可通过命令行 `cmake -DCMAKE_INSTALL_PREFIX=xxx ..` 修改。

### 6.4 测试命令 `CTest`
```cmake
# 启用测试，当前目录和子目录可添加测试用例
enable_testing()
# 或引入完整CTest模块，支持仪表盘、内存检测等
include(CTest)

# 添加测试用例
add_test(
  NAME my_app_test
  COMMAND my_app --test # 测试命令
  WORKING_DIRECTORY ${PROJECT_BINARY_DIR}
)

# 常用：GTest测试框架集成
find_package(GTest REQUIRED)
include(GoogleTest)
add_executable(unit_test test/unit_test.cpp)
target_link_libraries(unit_test PRIVATE GTest::gtest GTest::gtest_main my_static)
gtest_discover_tests(unit_test) # 自动发现测试用例
```

### 6.5 消息输出 `message()`
用于日志打印、调试，支持多种日志级别。
```cmake
# 语法：message([级别] "消息内容")
message(STATUS "项目版本: ${PROJECT_VERSION}") # 最常用，带--前缀
message(WARNING "这是一个警告") # 警告，不终止执行
message(AUTHOR_WARNING "开发者警告")
message(SEND_ERROR "错误，继续执行但生成失败")
message(FATAL_ERROR "致命错误，立即终止CMake")
message(DEBUG "调试信息，需设置--log-level=DEBUG才显示")
```

## 七、进阶特性
### 7.1 生成器表达式（Generator Expressions）
CMake 核心进阶特性，**构建阶段（生成构建文件时）才求值**，而非配置阶段，用于根据编译器、平台、构建配置等动态设置规则，完美解决跨平台、跨配置的条件编译问题。

#### 基础语法
- 条件表达式：`$<条件:真值>`，条件为真时求值为真值，否则为空
- 二选一表达式：`$<条件:真值,假值>`
- 逻辑运算：`$<AND:条件1,条件2>`、`$<OR:条件1,条件2>`、`$<NOT:条件>`

#### 常用生成器表达式
| 表达式 | 说明 |
|--------|------|
| `$<CONFIG:Debug>` | Debug配置时为1，否则为0 |
| `$<CXX_COMPILER_ID:GNU,Clang>` | 编译器为GCC/Clang时为1 |
| `$<PLATFORM_ID:Linux,Windows,Darwin>` | 对应平台时为1 |
| `$<COMPILE_LANGUAGE:CXX>` | 当前编译语言为C++时为1 |
| `$<TARGET_EXISTS:目标名>` | 目标存在时为1 |
| `$<TARGET_FILE:目标名>` | 目标的输出文件绝对路径 |
| `$<TARGET_FILE_DIR:目标名>` | 目标输出文件所在目录 |

#### 实用示例
```cmake
# 1. 仅Debug配置添加调试选项
target_compile_options(my_app PRIVATE $<$<CONFIG:Debug>:-O0 -g -fsanitize=address>)

# 2. 不同编译器设置不同警告选项
target_compile_options(my_app PRIVATE
  $<$<CXX_COMPILER_ID:GNU,Clang,AppleClang>:-Wall -Wextra -Wpedantic -Werror>
  $<$<CXX_COMPILER_ID:MSVC>:/W4 /WX /utf-8>
)

# 3. 编译后复制依赖库到可执行文件目录（跨配置）
add_custom_command(TARGET my_app POST_BUILD
  COMMAND ${CMAKE_COMMAND} -E copy_if_different
  $<TARGET_FILE:my_shared>
  $<TARGET_FILE_DIR:my_app>
  COMMENT "复制依赖库..."
  VERBATIM
)
```

> 避坑：生成器表达式不能用在 `if()` 中，`if()` 是配置阶段执行，此时生成器表达式尚未求值。

### 7.2 自定义命令与自定义目标
#### 7.2.1 `add_custom_command()`：自定义构建命令
两种核心模式：
1. **生成文件模式**：生成源文件，当目标依赖该文件时自动执行命令
   ```cmake
   # 示例：用flex生成词法分析器
   add_custom_command(
     OUTPUT lex.yy.c
     COMMAND flex ${CMAKE_CURRENT_SOURCE_DIR}/lexer.l
     MAIN_DEPENDENCY lexer.l # 主依赖文件
     DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/lexer.h # 额外依赖
     COMMENT "生成词法分析器..."
     WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
     VERBATIM # 保证参数正确转义，跨平台必加
   )
   # 把生成的文件加入目标
   add_executable(my_app src/main.c lex.yy.c)
   ```

2. **构建事件模式**：给目标添加构建前/后事件
   ```cmake
   add_custom_command(
     TARGET my_app
     POST_BUILD # 链接后执行，最常用；可选PRE_BUILD/PRE_LINK
     COMMAND ${CMAKE_COMMAND} -E copy $<TARGET_FILE:my_app> ${PROJECT_BINARY_DIR}/output
     COMMENT "复制可执行文件到输出目录..."
     VERBATIM
   )
   ```

#### 7.2.2 `add_custom_target()`：自定义目标
无输出文件，相当于Makefile的伪目标，默认不执行，需手动构建，常用于格式化、代码检查、文档生成等场景。
```cmake
# 示例：添加代码格式化目标
add_custom_target(
  format
  COMMAND clang-format -i ${SRC_FILES}
  COMMENT "格式化代码..."
  VERBATIM
)
# 执行方式：make format / ninja format / cmake --build . --target format
```

### 7.3 交叉编译
CMake 通过**工具链文件（toolchain file）**实现交叉编译，执行cmake时通过 `-DCMAKE_TOOLCHAIN_FILE=xxx.cmake` 指定。

#### 工具链文件核心配置
```cmake
# 目标系统名，裸机开发设为Generic
set(CMAKE_SYSTEM_NAME Linux)
# 目标处理器架构
set(CMAKE_SYSTEM_PROCESSOR aarch64)

# 交叉编译器
set(CMAKE_C_COMPILER aarch64-linux-gnu-gcc)
set(CMAKE_CXX_COMPILER aarch64-linux-gnu-g++)

# 依赖查找根目录，限制交叉编译时仅在该目录下查找库/头文件
set(CMAKE_FIND_ROOT_PATH /opt/aarch64-sysroot)

# 查找规则：程序用主机的，库/头文件只用目标系统的
set(CMAKE_FIND_ROOT_PATH_MODE_PROGRAM NEVER)
set(CMAKE_FIND_ROOT_PATH_MODE_LIBRARY ONLY)
set(CMAKE_FIND_ROOT_PATH_MODE_INCLUDE ONLY)
set(CMAKE_FIND_ROOT_PATH_MODE_PACKAGE ONLY)
```

## 八、最佳实践与避坑指南
### 8.1 现代CMake 核心最佳实践
1. **最低版本至少3.15+**，推荐3.20+，使用新特性，规避老旧bug。
2. **全程基于目标编程**，使用 `target_*` 系列命令，绝对避免全局的 `include_directories`、`add_definitions`、`link_directories`，防止污染。
3. **精准控制传播范围**，区分 `PRIVATE/PUBLIC/INTERFACE`，不要滥用 `PUBLIC`，警告选项一律用 `PRIVATE`。
4. **优先使用导入目标**，`find_package()` 优先链接库提供的命名空间目标，不要手动设置头文件和库路径。
5. **给库添加命名空间别名**，`add_library(MyProject::my_lib ALIAS my_lib)`，和导入目标保持一致，方便用户使用。
6. **手动列出源文件**，避免 `file(GLOB)` 收集源文件，非要使用需加 `CONFIGURE_DEPENDS`。
7. **用生成器表达式处理跨平台逻辑**，替代大量 `if()` 平台判断，代码更简洁、可维护。
8. **禁用编译器扩展**，`set(CMAKE_CXX_EXTENSIONS OFF)`，保证代码跨平台兼容性。
9. **安装时导出目标**，生成Config文件，让其他项目可通过 `find_package()` 直接使用你的库。
10. **禁止源内构建**，在CMakeLists.txt中添加防护代码，避免污染源码目录：
    ```cmake
    if(PROJECT_SOURCE_DIR STREQUAL PROJECT_BINARY_DIR)
      message(FATAL_ERROR "不允许源内构建，请创建build目录执行cmake")
    endif()
    ```

### 8.2 高频坑点避坑
1. **大小写坑**：命令不区分大小写，但变量、目标名、路径严格区分大小写，Linux下路径大小写敏感。
2. **if()解析坑**：字符串比较必须加双引号，变量值为 `ON/OFF` 等常量时，不加双引号会被直接解析为布尔值，导致判断错误。
3. **GLOB文件坑**：`file(GLOB)` 仅在配置阶段执行，新增/删除文件不会自动触发重新配置，导致漏编译/编译残留。
4. **构建类型坑**：不设置 `CMAKE_BUILD_TYPE` 时，默认是空配置，无优化无调试信息，不是Debug也不是Release。
5. **动态库导出坑**：Windows下dll默认不导出符号，会生成不了导入库，导致链接失败。解决方案：`__declspec(dllexport)` 导出符号，或设置 `set(CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS ON)` 自动导出。
6. **宏函数坑**：宏是文本替换，无作用域，会修改调用者的变量，优先使用函数，避免宏的副作用。
7. **静态库链接坑**：静态库链接静态库，不会合并依赖的静态库，必须把所有依赖的静态库都链接到最终的可执行文件/动态库。
8. **交叉编译查找坑**：交叉编译时 `find_package()` 找到主机的库，而非目标平台的库。解决方案：设置 `CMAKE_FIND_ROOT_PATH_MODE_LIBRARY/INCLUDE` 为 `ONLY`。
9. **生成器表达式坑**：生成器表达式不能用在 `if()` 中，`if()` 是配置阶段执行，生成器表达式构建阶段才求值。
10. **作用域坑**：`set(PARENT_SCOPE)` 只能修改直接父作用域的变量，无法修改祖父作用域，函数嵌套时需逐层向上传递。

## 九、标准现代CMakeLists.txt 模板
```cmake
cmake_minimum_required(VERSION 3.15 FATAL_ERROR)

project(
  MyProject
  VERSION 1.0.0
  DESCRIPTION "现代CMake示例项目"
  LANGUAGES CXX
)

# 禁止源内构建
if(PROJECT_SOURCE_DIR STREQUAL PROJECT_BINARY_DIR)
  message(FATAL_ERROR "不允许源内构建，请创建build目录执行cmake")
endif()

# 全局C++标准兜底，目标级配置优先
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# 输出目录设置
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/bin)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/lib)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/lib)

# 查找依赖
find_package(Threads REQUIRED)
find_package(spdlog REQUIRED)

# 定义库目标
add_library(my_lib STATIC
  src/my_lib.cpp
  src/utils.cpp
)

# 给库添加别名
add_library(MyProject::my_lib ALIAS my_lib)

# 设置库目标属性
target_include_directories(my_lib
  PUBLIC
    $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include>
  PRIVATE
    ${PROJECT_SOURCE_DIR}/src
)

target_compile_features(my_lib PUBLIC cxx_std_17)

# 私有警告选项，不传递给用户
target_compile_options(my_lib PRIVATE
  $<$<CXX_COMPILER_ID:GNU,Clang,AppleClang>:-Wall -Wextra -Wpedantic -Werror>
  $<$<CXX_COMPILER_ID:MSVC>:/W4 /WX /utf-8>
)

# 链接依赖
target_link_libraries(my_lib
  PUBLIC Threads::Threads spdlog::spdlog
)

# 定义可执行目标
add_executable(my_app src/main.cpp)
target_link_libraries(my_app PRIVATE MyProject::my_lib)

# 启用测试
enable_testing()
add_test(NAME my_app_basic_test COMMAND my_app)

# 安装配置
include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

# 安装目标
install(TARGETS my_lib my_app
  EXPORT MyProjectTargets
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
  INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)

# 安装头文件
install(DIRECTORY ${PROJECT_SOURCE_DIR}/include/
  DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
  FILES_MATCHING PATTERN "*.h" "*.hpp"
)

# 生成Config文件
configure_package_config_file(
  ${PROJECT_SOURCE_DIR}/cmake/MyProjectConfig.cmake.in
  ${PROJECT_BINARY_DIR}/MyProjectConfig.cmake
  INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/MyProject
)

# 生成版本文件
write_basic_package_version_file(
  ${PROJECT_BINARY_DIR}/MyProjectConfigVersion.cmake
  VERSION ${PROJECT_VERSION}
  COMPATIBILITY SameMajorVersion
)

# 安装Config文件
install(FILES
  ${PROJECT_BINARY_DIR}/MyProjectConfig.cmake
  ${PROJECT_BINARY_DIR}/MyProjectConfigVersion.cmake
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/MyProject
)

# 安装导出目标
install(EXPORT MyProjectTargets
  FILE MyProjectTargets.cmake
  NAMESPACE MyProject::
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/MyProject
)
```

## 十、常用内置变量速查
| 变量名 | 说明 |
|--------|------|
| `CMAKE_SOURCE_DIR` | 顶层CMakeLists.txt源码根目录 |
| `CMAKE_BINARY_DIR` | 顶层构建目录（build目录） |
| `CMAKE_CURRENT_SOURCE_DIR` | 当前处理的CMakeLists.txt所在目录 |
| `CMAKE_CURRENT_BINARY_DIR` | 当前CMakeLists.txt对应的构建目录 |
| `PROJECT_SOURCE_DIR` | 最近的project()对应的源码目录 |
| `PROJECT_BINARY_DIR` | 最近的project()对应的构建目录 |
| `PROJECT_NAME` | 最近的project()设置的项目名 |
| `PROJECT_VERSION` | 最近的project()设置的版本号 |
| `CMAKE_INSTALL_PREFIX` | 安装路径前缀 |
| `CMAKE_MODULE_PATH` | CMake模块搜索路径 |
| `CMAKE_TOOLCHAIN_FILE` | 交叉编译工具链文件路径 |
| `CMAKE_BUILD_TYPE` | 构建类型（单配置生成器） |
| `CMAKE_CXX_COMPILER` | C++编译器路径 |
| `CMAKE_C_COMPILER` | C编译器路径 |


