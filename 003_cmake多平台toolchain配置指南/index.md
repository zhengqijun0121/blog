# CMake 多平台 Toolchain 配置指南


# CMake 多平台 Toolchain 编写指南
CMake Toolchain 的核心作用是**隔离平台/架构的编译器、链接器、系统配置差异**，配合通用构建逻辑实现一套代码多平台编译。分为两大核心场景：**原生多平台适配**（在目标平台本地编译，自动适配环境）、**交叉编译**（在主机编译其他平台的目标程序），以下是完整的实现方案、模板和最佳实践。

---

## 一、核心基础概念与关键变量
### 1.1 Toolchain 执行原理
Toolchain 文件通过 `-DCMAKE_TOOLCHAIN_FILE=xxx.cmake` 参数传入，会在 CMake 系统检测流程**之前**执行，用于覆盖默认的编译器、目标系统属性、编译/链接规则，是交叉编译的核心，也可用于原生平台的编译配置统一管理。

### 1.2 核心必知变量
| 变量分类 | 关键变量 | 作用说明 |
| :--- | :--- | :--- |
| 目标系统配置 | `CMAKE_SYSTEM_NAME` | 目标操作系统名（如`Linux`/`Windows`/`Darwin`/`Android`/`Generic`），**设置后CMake自动进入交叉编译模式** |
| | `CMAKE_SYSTEM_PROCESSOR` | 目标CPU架构（如`x86_64`/`aarch64`/`armv7-a`/`riscv64`/`AMD64`） |
| | `CMAKE_SYSROOT` | 交叉编译的系统根目录，对应目标平台的系统头文件/库根路径 |
| 工具链配置 | `CMAKE_C_COMPILER` | C编译器路径/名称（如`gcc`/`clang`/`cl`/`aarch64-linux-gnu-gcc`） |
| | `CMAKE_CXX_COMPILER` | C++编译器路径/名称 |
| | `CMAKE_AR`/`CMAKE_RANLIB`/`CMAKE_STRIP` | 静态库归档、符号表处理、裁剪工具路径 |
| | `CMAKE_LINKER` | 链接器路径 |
| 编译链接选项 | `CMAKE_C_FLAGS`/`CMAKE_CXX_FLAGS` | 全局C/C++编译选项，分构建类型可加`_DEBUG`/`_RELEASE`后缀 |
| | `CMAKE_EXE_LINKER_FLAGS` | 可执行文件全局链接选项 |
| | `CMAKE_SHARED_LINKER_FLAGS` | 共享库全局链接选项 |
| | `CMAKE_POSITION_INDEPENDENT_CODE` | 生成位置无关代码，Linux/macOS共享库必须开启（设为`ON`） |
| 交叉编译查找规则 | `CMAKE_FIND_ROOT_PATH` | 交叉编译时，查找头文件/库/程序的根目录前缀 |
| | `CMAKE_FIND_ROOT_PATH_MODE_PROGRAM` | 程序查找模式，交叉编译设为`NEVER`（仅找主机程序） |
| | `CMAKE_FIND_ROOT_PATH_MODE_LIBRARY` | 库查找模式，交叉编译设为`ONLY`（仅找目标平台库） |
| | `CMAKE_FIND_ROOT_PATH_MODE_INCLUDE` | 头文件查找模式，交叉编译设为`ONLY` |
| | `CMAKE_FIND_ROOT_PATH_MODE_PACKAGE` | 包查找模式，交叉编译设为`ONLY` |

---

## 二、多平台 Toolchain 通用架构设计
最佳实践是**拆分通用配置与平台专属配置**，避免重复代码，便于维护和扩展，架构如下：
```
cmake/
├── toolchain/
│   ├── common.cmake          # 全平台通用基础配置（C++标准、警告规则、构建默认值）
│   ├── linux-x86_64.cmake    # Linux x86_64 平台配置
│   ├── linux-aarch64.cmake   # Linux ARM64 交叉编译配置
│   ├── windows-mingw.cmake   # Windows MinGW 交叉编译配置
│   ├── windows-msvc.cmake    # Windows MSVC 原生配置
│   ├── macos-universal.cmake # macOS 通用二进制配置
│   └── android.cmake         # Android 平台配置
└── build.sh                  # 多平台统一构建脚本
```

---

## 三、全平台通用配置模板（common.cmake）
所有平台的toolchain都先引入该文件，统一编译规则，避免重复配置，可直接复用：
```cmake
# cmake/toolchain/common.cmake
# 防止重复引入
if(COMMON_TOOLCHAIN_INCLUDED)
    return()
endif()
set(COMMON_TOOLCHAIN_INCLUDED ON)

# 1. 统一C/C++标准
set(CMAKE_C_STANDARD 11 CACHE STRING "C standard")
set(CMAKE_C_STANDARD_REQUIRED ON CACHE BOOL "C standard required")
set(CMAKE_C_EXTENSIONS OFF CACHE BOOL "Disable GNU C extensions")

set(CMAKE_CXX_STANDARD 17 CACHE STRING "C++ standard")
set(CMAKE_CXX_STANDARD_REQUIRED ON CACHE BOOL "C++ standard required")
set(CMAKE_CXX_EXTENSIONS OFF CACHE BOOL "Disable GNU C++ extensions")

# 2. 默认构建类型（无指定时用Release）
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
    set(CMAKE_BUILD_TYPE Release CACHE STRING "Build type" FORCE)
    set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "Debug" "Release" "RelWithDebInfo" "MinSizeRel")
endif()

# 3. 统一警告规则（分编译器适配）
if(CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang|AppleClang")
    # GCC/Clang 通用警告选项
    add_compile_options(
        -Wall -Wextra -Wpedantic
        -Wunused -Wshadow -Wconversion -Wsign-conversion
        -Wformat=2 -Wnull-dereference
    )
    # Release 优化配置
    add_compile_options($<$<CONFIG:Release>:-O3 -ffunction-sections -fdata-sections>)
    # Debug 调试配置
    add_compile_options($<$<CONFIG:Debug>:-O0 -g3 -ggdb>)
elseif(CMAKE_CXX_COMPILER_ID STREQUAL "MSVC")
    # MSVC 警告与编码配置
    add_compile_options(/W4 /utf-8 /MP)
    add_compile_options(/wd4996) # 禁用不安全函数警告
    # Release 优化配置
    add_compile_options($<$<CONFIG:Release>:/O2 /GL>)
    add_link_options($<$<CONFIG:Release>:/LTCG>)
    # Debug 调试配置
    add_compile_options($<$<CONFIG:Debug>:/Od /Zi>)
endif()

# 4. 全局链接优化（GCC/Clang 垃圾回收无用段）
if(CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang|AppleClang")
    add_link_options($<$<CONFIG:Release>:-Wl,--gc-sections>)
endif()

# 5. 输出目录统一规划（按平台+架构隔离，避免文件覆盖）
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
```

---

## 四、主流平台 Toolchain 完整示例
### 4.1 Linux x86_64 原生/交叉编译
```cmake
# cmake/toolchain/linux-x86_64.cmake
# 引入通用配置
include(${CMAKE_CURRENT_LIST_DIR}/common.cmake)

# 目标系统配置
set(CMAKE_SYSTEM_NAME Linux CACHE STRING "Target system")
set(CMAKE_SYSTEM_PROCESSOR x86_64 CACHE STRING "Target processor")

# 编译器配置（原生编译可直接写gcc/g++，交叉编译填绝对路径）
# 可通过环境变量 TOOLCHAIN_PREFIX 传入编译器前缀，如 export TOOLCHAIN_PREFIX=/usr/bin/
if(DEFINED ENV{TOOLCHAIN_PREFIX})
    set(TOOLCHAIN_PREFIX $ENV{TOOLCHAIN_PREFIX} CACHE STRING "Toolchain prefix")
else()
    set(TOOLCHAIN_PREFIX "" CACHE STRING "Toolchain prefix")
endif()

set(CMAKE_C_COMPILER ${TOOLCHAIN_PREFIX}gcc CACHE FILEPATH "C compiler")
set(CMAKE_CXX_COMPILER ${TOOLCHAIN_PREFIX}g++ CACHE FILEPATH "C++ compiler")
set(CMAKE_AR ${TOOLCHAIN_PREFIX}ar CACHE FILEPATH "AR tool")
set(CMAKE_RANLIB ${TOOLCHAIN_PREFIX}ranlib CACHE FILEPATH "RANLIB tool")
set(CMAKE_STRIP ${TOOLCHAIN_PREFIX}strip CACHE FILEPATH "STRIP tool")

# 交叉编译查找规则（原生编译可注释，交叉编译必须开启）
if(CMAKE_CROSSCOMPILING)
    set(CMAKE_FIND_ROOT_PATH ${TOOLCHAIN_PREFIX}../${CMAKE_SYSTEM_PROCESSOR}-linux-gnu)
    set(CMAKE_FIND_ROOT_PATH_MODE_PROGRAM NEVER)
    set(CMAKE_FIND_ROOT_PATH_MODE_LIBRARY ONLY)
    set(CMAKE_FIND_ROOT_PATH_MODE_INCLUDE ONLY)
    set(CMAKE_FIND_ROOT_PATH_MODE_PACKAGE ONLY)
endif()

# Linux 专属配置
add_definitions(-D_LINUX_PLATFORM_)
# 开启PIC
set(CMAKE_POSITION_INDEPENDENT_CODE ON CACHE BOOL "Enable PIC")
# Linux 必须链接pthread库
link_libraries(pthread)
```

### 4.2 Linux ARM64 (aarch64) 交叉编译
```cmake
# cmake/toolchain/linux-aarch64.cmake
include(${CMAKE_CURRENT_LIST_DIR}/common.cmake)

# 目标系统配置
set(CMAKE_SYSTEM_NAME Linux CACHE STRING "Target system")
set(CMAKE_SYSTEM_PROCESSOR aarch64 CACHE STRING "Target processor")

# 交叉编译器前缀（需提前安装 aarch64-linux-gnu-gcc 工具链）
set(TOOLCHAIN_PREFIX aarch64-linux-gnu- CACHE STRING "Toolchain prefix")
if(DEFINED ENV{AARCH64_TOOLCHAIN_PATH})
    set(TOOLCHAIN_PATH $ENV{AARCH64_TOOLCHAIN_PATH}/bin/)
endif()

set(CMAKE_C_COMPILER ${TOOLCHAIN_PATH}${TOOLCHAIN_PREFIX}gcc CACHE FILEPATH "C compiler")
set(CMAKE_CXX_COMPILER ${TOOLCHAIN_PATH}${TOOLCHAIN_PREFIX}g++ CACHE FILEPATH "C++ compiler")
set(CMAKE_AR ${TOOLCHAIN_PATH}${TOOLCHAIN_PREFIX}ar CACHE FILEPATH "AR tool")
set(CMAKE_RANLIB ${TOOLCHAIN_PATH}${TOOLCHAIN_PREFIX}ranlib CACHE FILEPATH "RANLIB tool")
set(CMAKE_STRIP ${TOOLCHAIN_PATH}${TOOLCHAIN_PREFIX}strip CACHE FILEPATH "STRIP tool")

# 交叉编译查找规则
set(CMAKE_FIND_ROOT_PATH_MODE_PROGRAM NEVER)
set(CMAKE_FIND_ROOT_PATH_MODE_LIBRARY ONLY)
set(CMAKE_FIND_ROOT_PATH_MODE_INCLUDE ONLY)
set(CMAKE_FIND_ROOT_PATH_MODE_PACKAGE ONLY)

# ARM64 专属配置
set(CMAKE_POSITION_INDEPENDENT_CODE ON CACHE BOOL "Enable PIC")
add_definitions(-D_LINUX_ARM64_PLATFORM_)
link_libraries(pthread)
```

### 4.3 Windows MinGW 交叉编译（Linux主机编译Windows程序）
```cmake
# cmake/toolchain/windows-mingw.cmake
include(${CMAKE_CURRENT_LIST_DIR}/common.cmake)

# 目标系统配置
set(CMAKE_SYSTEM_NAME Windows CACHE STRING "Target system")
set(CMAKE_SYSTEM_PROCESSOR AMD64 CACHE STRING "Target processor")

# MinGW 交叉编译器（需提前安装 mingw-w64 工具链）
set(TOOLCHAIN_PREFIX x86_64-w64-mingw32- CACHE STRING "Toolchain prefix")
if(DEFINED ENV{MINGW_TOOLCHAIN_PATH})
    set(TOOLCHAIN_PATH $ENV{MINGW_TOOLCHAIN_PATH}/bin/)
endif()

set(CMAKE_C_COMPILER ${TOOLCHAIN_PATH}${TOOLCHAIN_PREFIX}gcc CACHE FILEPATH "C compiler")
set(CMAKE_CXX_COMPILER ${TOOLCHAIN_PATH}${TOOLCHAIN_PREFIX}g++ CACHE FILEPATH "C++ compiler")
set(CMAKE_RC_COMPILER ${TOOLCHAIN_PATH}${TOOLCHAIN_PREFIX}windres CACHE FILEPATH "RC compiler")
set(CMAKE_AR ${TOOLCHAIN_PATH}${TOOLCHAIN_PREFIX}ar CACHE FILEPATH "AR tool")
set(CMAKE_RANLIB ${TOOLCHAIN_PATH}${TOOLCHAIN_PREFIX}ranlib CACHE FILEPATH "RANLIB tool")
set(CMAKE_STRIP ${TOOLCHAIN_PATH}${TOOLCHAIN_PREFIX}strip CACHE FILEPATH "STRIP tool")

# 交叉编译查找规则
set(CMAKE_FIND_ROOT_PATH_MODE_PROGRAM NEVER)
set(CMAKE_FIND_ROOT_PATH_MODE_LIBRARY ONLY)
set(CMAKE_FIND_ROOT_PATH_MODE_INCLUDE ONLY)
set(CMAKE_FIND_ROOT_PATH_MODE_PACKAGE ONLY)

# Windows 专属配置
set(CMAKE_EXECUTABLE_SUFFIX .exe)
set(CMAKE_SHARED_LIBRARY_PREFIX "")
set(CMAKE_SHARED_LIBRARY_SUFFIX .dll)
set(CMAKE_STATIC_LIBRARY_PREFIX "")
set(CMAKE_STATIC_LIBRARY_SUFFIX .lib)

# MinGW 静态链接标准库，避免依赖dll
add_link_options(-static-libgcc -static-libstdc++ -Wl,--subsystem,console)
add_definitions(-D_WINDOWS_PLATFORM_ -DWIN32_LEAN_AND_MEAN -DNOMINMAX)
```

### 4.4 Windows MSVC 原生适配
MSVC 通常无需单独的toolchain文件，通过VS开发人员命令提示符执行CMake即可自动适配，如需统一配置，可使用以下模板：
```cmake
# cmake/toolchain/windows-msvc.cmake
include(${CMAKE_CURRENT_LIST_DIR}/common.cmake)

# 目标系统配置
set(CMAKE_SYSTEM_NAME Windows CACHE STRING "Target system")
set(CMAKE_SYSTEM_PROCESSOR AMD64 CACHE STRING "Target processor")

# MSVC 工具集版本（可选，默认使用最新）
# set(CMAKE_GENERATOR_TOOLSET v143 CACHE STRING "MSVC toolset")
# set(CMAKE_VS_PLATFORM_TOOLSET_HOST_ARCHITECTURE x64 CACHE STRING "Host arch")

# Windows 专属配置
add_definitions(-D_WINDOWS_PLATFORM_ -DWIN32_LEAN_AND_MEAN -DNOMINMAX -D_CRT_SECURE_NO_WARNINGS)
# 运行时库配置（MT/MD，根据需求调整）
set(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>DLL" CACHE STRING "MSVC runtime")
```

### 4.5 macOS 通用二进制（x86_64+arm64）
```cmake
# cmake/toolchain/macos-universal.cmake
include(${CMAKE_CURRENT_LIST_DIR}/common.cmake)

# 目标系统配置
set(CMAKE_SYSTEM_NAME Darwin CACHE STRING "Target system")
set(CMAKE_SYSTEM_PROCESSOR arm64 CACHE STRING "Target processor")

# 编译器配置（macOS 原生 clang）
set(CMAKE_C_COMPILER clang CACHE FILEPATH "C compiler")
set(CMAKE_CXX_COMPILER clang++ CACHE FILEPATH "C++ compiler")

# 通用二进制配置（同时支持x86_64和arm64）
set(CMAKE_OSX_ARCHITECTURES "x86_64;arm64" CACHE STRING "macOS architectures")
# 最低支持的macOS版本
set(CMAKE_OSX_DEPLOYMENT_TARGET 10.15 CACHE STRING "macOS deployment target")
# SDK路径（自动检测，也可手动指定）
execute_process(COMMAND xcrun --sdk macosx --show-sdk-path OUTPUT_VARIABLE MACOS_SDK_PATH OUTPUT_STRIP_TRAILING_WHITESPACE)
set(CMAKE_OSX_SYSROOT ${MACOS_SDK_PATH} CACHE PATH "macOS SDK path")

# macOS 专属配置
set(CMAKE_POSITION_INDEPENDENT_CODE ON CACHE BOOL "Enable PIC")
set(CMAKE_MACOSX_RPATH ON CACHE BOOL "Enable RPATH")
set(CMAKE_INSTALL_RPATH "@executable_path/../lib" CACHE STRING "Install RPATH")
add_definitions(-D_DARWIN_PLATFORM_ -D_MACOS_PLATFORM_)
```

### 4.6 Android 平台适配
Android NDK 已内置官方toolchain文件，无需从零编写，直接调用即可，示例如下：
```bash
# 直接通过命令行调用NDK内置toolchain，无需单独写cmake文件
cmake -DCMAKE_TOOLCHAIN_FILE=${NDK_PATH}/build/cmake/android.toolchain.cmake \
      -DANDROID_ABI=arm64-v8a \
      -DANDROID_PLATFORM=android-24 \
      -DANDROID_STL=c++_shared \
      ..
```
如需自定义扩展，可在自己的toolchain中引入NDK的官方toolchain：
```cmake
# cmake/toolchain/android.cmake
set(ANDROID_ABI arm64-v8a CACHE STRING "Android ABI")
set(ANDROID_PLATFORM android-24 CACHE STRING "Android min platform")
set(ANDROID_STL c++_shared CACHE STRING "Android STL")

# 引入NDK官方toolchain
include($ENV{NDK_PATH}/build/cmake/android.toolchain.cmake)
# 引入通用配置
include(${CMAKE_CURRENT_LIST_DIR}/common.cmake)

# 自定义Android专属配置
add_definitions(-D_ANDROID_PLATFORM_)
```

---

## 五、多平台统一构建脚本
编写一键构建脚本，通过参数指定目标平台，自动调用对应toolchain，示例（bash脚本，Linux/macOS可用）：
```bash
#!/bin/bash
# build.sh
set -e

# 支持的平台列表
SUPPORTED_PLATFORMS="linux-x86_64 linux-aarch64 windows-mingw macos-universal android"

# 入参检查
if [ $# -lt 1 ]; then
    echo "Usage: $0 <platform> [build_type]"
    echo "Supported platforms: ${SUPPORTED_PLATFORMS}"
    echo "Default build_type: Release"
    exit 1
fi

PLATFORM=$1
BUILD_TYPE=${2:-Release}
TOOLCHAIN_FILE="$(pwd)/cmake/toolchain/${PLATFORM}.cmake"
BUILD_DIR="$(pwd)/build/${PLATFORM}"

# 检查toolchain文件是否存在
if [ ! -f "${TOOLCHAIN_FILE}" ]; then
    echo "Error: Toolchain file ${TOOLCHAIN_FILE} not found!"
    exit 1
fi

# 创建构建目录
mkdir -p "${BUILD_DIR}"
cd "${BUILD_DIR}"

# 执行CMake构建
echo "Building ${PLATFORM} ${BUILD_TYPE}..."
cmake \
    -DCMAKE_TOOLCHAIN_FILE="${TOOLCHAIN_FILE}" \
    -DCMAKE_BUILD_TYPE="${BUILD_TYPE}" \
    ../..

cmake --build . -j$(nproc)

echo "Build completed! Output: ${BUILD_DIR}/bin"
```

使用方法：
```bash
# 构建Linux x86_64 Release版本
./build.sh linux-x86_64 Release

# 构建Windows MinGW Debug版本
./build.sh windows-mingw Debug

# 构建Linux ARM64版本
./build.sh linux-aarch64
```

---

## 六、最佳实践与避坑指南
### 6.1 核心最佳实践
1. **严格隔离平台差异**：平台专属的编译选项、链接库、宏定义全部放在toolchain文件中，主`CMakeLists.txt`只写通用构建逻辑，避免大量`if(WIN32)`/`if(UNIX)`分支。
2. **避免硬编码路径**：编译器路径、SDK路径通过环境变量传入，不要写死在cmake文件中，适配不同开发环境。
3. **输出目录按平台隔离**：不同平台的构建产物放在独立的build目录下，避免文件覆盖和增量编译异常。
4. **统一编译标准与警告规则**：通过common.cmake统一C/C++标准和警告等级，避免不同平台出现编译行为不一致。
5. **交叉编译严格控制查找规则**：必须设置`CMAKE_FIND_ROOT_PATH_MODE_*`变量，防止`find_library`/`find_package`找到主机的库，导致链接失败。
6. **优先使用官方toolchain**：Android、iOS、ROS等平台已提供官方维护的toolchain文件，直接复用即可，无需从零编写，减少兼容性问题。

### 6.2 常见坑与解决方案
| 问题现象 | 根因 | 解决方案 |
| :--- | :--- | :--- |
| 交叉编译时，CMake提示“编译器无法编译简单测试程序” | 未设置`CMAKE_SYSTEM_NAME`，CMake未进入交叉编译模式，用主机规则校验目标编译器 | 必须在toolchain开头设置`CMAKE_SYSTEM_NAME` |
| 交叉编译链接时，找到主机的so库，出现架构不匹配错误 | 未设置`FIND_ROOT_PATH`相关变量，CMake优先查找主机系统库 | 设置`CMAKE_FIND_ROOT_PATH_MODE_LIBRARY/INCLUDE`为`ONLY` |
| Linux共享库编译报错“relocation R_X86_64_32 against `.rodata' can not be used when making a shared object” | 未开启PIC（位置无关代码） | 设置`CMAKE_POSITION_INDEPENDENT_CODE ON` |
| MinGW编译的程序运行时缺少libgcc_s_seh-1.dll等依赖 | 标准库动态链接，未静态链接 | 添加链接选项`-static-libgcc -static-libstdc++` |
| MSVC编译出现中文乱码 | 未指定源码编码为UTF-8 | 添加编译选项`/utf-8` |
| macOS编译提示“library not found for -lxxx” | RPATH未配置，运行时找不到动态库 | 开启`CMAKE_MACOSX_RPATH`，设置`CMAKE_INSTALL_RPATH` |


