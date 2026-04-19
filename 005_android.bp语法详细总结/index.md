# Android.bp 语法详细总结


# Android.bp 语法详细总结
Android.bp 是 Android **Soong 构建系统**的核心配置文件，基于 Blueprint 语法设计，用于替代传统的 Android.mk（Make 构建体系），采用纯声明式语法，风格接近 Go/Bazel，无原生的分支/循环控制流，复杂逻辑由 Go 语言编写的构建底层处理，具备语法简洁、构建高效、可读性强的核心优势。

## 一、基础语法核心规则
### 1. 基础语法格式
Android.bp 的核心是**模块定义**，一个模块以「模块类型」开头，后跟花括号包裹的键值对属性集，格式如下：
```bp
模块类型 {
    属性名1: 属性值1,
    属性名2: 属性值2,
    // 更多属性
}
```
- 键值对必须以 `,` 结尾，最后一个属性末尾也可保留逗号（语法兼容）
- 缩进无强制语法要求，推荐使用 4 空格缩进保证可读性
- 语法大小写敏感，所有关键字、属性名、模块名均严格区分大小写

### 2. 支持的数据类型
Android.bp 是强类型语法，变量类型由首次赋值动态推断，模块属性类型由模块类型静态定义，支持的核心类型如下：

| 数据类型 | 说明 | 示例 |
|----------|------|------|
| 布尔值 | 仅支持 `true`/`false` | `enabled: true` |
| 整数 | 整型数值，支持加减运算 | `version: 1` |
| 字符串 | 双引号包裹的文本，不支持单引号 | `name: "my_demo"` |
| 字符串列表 | 方括号包裹的字符串数组，最常用类型 | `srcs: ["main.cpp", "utils.cpp"]` |
| Map/字典 | 花括号包裹的嵌套键值对，支持多层嵌套 | `product_variables: { eng: { cflags: ["-DDEBUG"] } }` |

### 3. 注释语法
与 Go 语言完全兼容，支持两种注释格式：
- 单行注释：`// 注释内容`，最常用
- 块注释：`/* 多行注释内容 */`，适合大段说明
```bp
// 这是一个C++可执行文件模块
cc_binary {
    name: "demo",
    /*
    这里是多行注释
    用于说明源文件配置
    */
    srcs: ["main.cpp"],
}
```

### 4. 变量与操作符
#### 4.1 变量定义与作用域
- 仅支持文件顶层定义变量，作用域为当前文件剩余部分及子目录的 Blueprint 文件
- 变量默认**不可变**，仅可在被引用前通过 `+=` 追加内容
```bp
// 顶层变量定义
common_srcs = ["base.cpp", "utils.cpp"]
common_cflags = ["-Wall", "-Wextra"]

// 变量追加（仅可在引用前执行）
common_cflags += ["-O2"]

cc_binary {
    name: "demo",
    srcs: common_srcs + ["main.cpp"], // 变量直接引用
    cflags: common_cflags,
}
```

#### 4.2 支持的操作符
Android.bp 仅支持极简操作符，无复杂逻辑运算，核心操作符如下：
- 赋值：`=` 基础赋值，`+=` 追加赋值
- 加法：`+`，支持字符串拼接、列表合并、Map 并集、整数相加
  - 字符串：`"lib" + "demo" = "libdemo"`
  - 列表：`["a.cpp"] + ["b.cpp"] = ["a.cpp", "b.cpp"]`
  - Map：合并两个字典，重复键的值会被覆盖

## 二、模块核心定义与通用规则
模块是 Android.bp 的最小构建单元，每个模块对应一个构建目标（可执行文件、库、应用等）。

### 1. 强制必填属性：name
**所有模块必须包含 `name` 属性**，其值为模块的唯一标识符，规则如下：
- 模块名在全源码所有 Android.bp 文件中必须唯一（命名空间、预编译模块除外）
- 其他模块通过该名称引用、依赖当前模块
- 命名需符合构建系统规范，禁止特殊字符，推荐使用下划线、字母、数字组合
```bp
cc_library {
    name: "libdemo", // 全局唯一模块名
    srcs: ["demo.cpp"],
}
```

### 2. 源文件配置：srcs 与 exclude_srcs
#### 2.1 srcs 核心用法
`srcs` 用于指定模块的源文件，支持 3 种核心用法：
1.  直接指定文件路径：相对当前 Android.bp 所在目录的路径
2.  通配符匹配：支持 Unix 标准通配符，`*` 匹配任意字符，`**` 匹配多级目录
3.  模块引用：通过 `:模块名` 引用其他生成源文件的模块（如 `filegroup`、`genrule`）
```bp
cc_binary {
    name: "demo",
    // 1. 直接指定文件
    // 2. 通配符匹配：src目录下所有.cpp文件
    // 3. 引用filegroup模块的源文件
    srcs: [
        "main.cpp",
        "src/**/*.cpp",
        ":common_filegroup",
    ],
}

// 源文件分组模块
filegroup {
    name: "common_filegroup",
    srcs: ["base/utils.cpp", "base/log.cpp"],
}
```

#### 2.2 exclude_srcs 排除文件
用于从 `srcs` 的匹配结果中排除指定文件，通常配合通配符使用：
```bp
cc_binary {
    name: "demo",
    srcs: ["src/**/*.cpp"],
    exclude_srcs: [
        "src/test.cpp", // 排除单文件
        "src/unused/**/*.cpp", // 排除整个目录
    ],
}
```

## 三、高频常用模块类型详解
Android.bp 支持上百种模块类型，覆盖 Android 系统构建全场景，以下是开发中最常用的模块类型，按场景分类说明。

### 1. C/C++ 原生开发模块
| 模块类型 | 核心作用 | 最简示例 |
|----------|----------|----------|
| `cc_binary` | 编译生成 C/C++ 可执行文件 | ```bp cc_binary { name: "my_daemon", srcs: ["main.cpp"], shared_libs: ["liblog"], } ``` |
| `cc_library` | 编译生成 C/C++ 库，可同时输出静态库+动态库 | ```bp cc_library { name: "libdemo", srcs: ["demo.cpp"], export_include_dirs: ["include"], } ``` |
| `cc_library_static` | 仅编译生成 C/C++ 静态库（.a） | ```bp cc_library_static { name: "libdemo_static", srcs: ["demo.cpp"], } ``` |
| `cc_library_shared` | 仅编译生成 C/C++ 动态库（.so） | ```bp cc_library_shared { name: "libdemo_shared", srcs: ["demo.cpp"], } ``` |
| `cc_defaults` | C/C++ 模块的默认配置模板，用于属性复用 | ```bp cc_defaults { name: "common_cc_config", cflags: ["-Wall", "-Werror"], shared_libs: ["libbase"], } ``` |
| `cc_benchmark` | C/C++ 性能基准测试模块 | ```bp cc_benchmark { name: "demo_bench", srcs: ["benchmark.cpp"], } ``` |
| `cc_test` | C/C++ 单元测试模块 | ```bp cc_test { name: "demo_test", srcs: ["test.cpp"], } ``` |

### 2. Java/Kotlin 开发模块
| 模块类型 | 核心作用 | 最简示例 |
|----------|----------|----------|
| `java_library` | 编译生成 Java 库（.jar），支持 Kotlin | ```bp java_library { name: "demo_java_lib", srcs: ["src/**/*.java"], sdk_version: "current", } ``` |
| `java_library_static` | 编译生成 Java 静态库，打包进目标模块 | ```bp java_library_static { name: "demo_java_static", srcs: ["src/**/*.java"], } ``` |
| `java_defaults` | Java 模块的默认配置模板，属性复用 | ```bp java_defaults { name: "common_java_config", javacflags: ["-Xlint:all"], sdk_version: "current", } ``` |

### 3. Android 应用开发模块
| 模块类型 | 核心作用 | 最简示例 |
|----------|----------|----------|
| `android_app` | 编译生成 Android APK 安装包 | ```bp android_app { name: "MyApp", srcs: ["src/**/*.java"], manifest: "AndroidManifest.xml", resource_dirs: ["res"], sdk_version: "current", certificate: "platform", } ``` |
| `android_app_certificate` | 定义 APK 签名证书 | ```bp android_app_certificate { name: "my_app_cert", key: "app.key", certificate: "app.x509.pem", } ``` |
| `android_library` | 编译生成 Android AAR 库 | ```bp android_library { name: "my_aar", srcs: ["src/**/*.java"], resource_dirs: ["res"], manifest: "AndroidManifest.xml", } ``` |

### 4. 预编译与辅助模块
| 模块类型 | 核心作用 | 最简示例 |
|----------|----------|----------|
| `filegroup` | 源文件分组，供其他模块引用，不参与编译 | ```bp filegroup { name: "common_res", srcs: ["res/**/*.xml"], } ``` |
| `java_import` | 导入预编译的 JAR 包 | ```bp java_import { name: "third_party_jar", jars: ["libs/third.jar"], } ``` |
| `prebuilt_etc` | 预编译配置文件，拷贝到 system/etc 分区 | ```bp prebuilt_etc { name: "demo.conf", src: "demo.conf", } ``` |
| `prebuilt_libs` | 导入预编译的 so 动态库 | ```bp prebuilt_libs { name: "libprebuilt", srcs: ["libs/arm64-v8a/libprebuilt.so"], } ``` |
| `genrule` | 自定义生成规则，执行脚本/命令生成文件 | ```bp genrule { name: "gen_header", srcs: ["config.json"], out: ["config.h"], cmd: "python $(location :gen_script.py) $< -o $@", } ``` |

## 四、核心属性详解
### 1. 依赖管理属性
依赖是模块间关联的核心，Android.bp 按语言、链接类型拆分了精细化的依赖属性，核心如下：

| 属性名 | 适用模块 | 核心作用 | 示例 |
|--------|----------|----------|------|
| `shared_libs` | C/C++ 模块 | 依赖动态链接库（.so），会传递链接关系 | `shared_libs: ["liblog", "libutils"]` |
| `static_libs` | C/C++/Java 模块 | 依赖静态库，编译时打包进当前模块 | `static_libs: ["libbase", "libdemo_static"]` |
| `libs` | Java/Android 模块 | 依赖 Java 库/JAR 包 | `libs: ["framework", "third_party_jar"]` |
| `deps` | 全模块通用 | 通用依赖，用于非链接型依赖（如生成源文件的模块） | `deps: [":gen_header", ":common_filegroup"]` |
| `header_libs` | C/C++ 模块 | 依赖头文件-only 库，仅传递头文件路径，不参与链接 | `header_libs: ["libbase_headers"]` |
| `export_include_dirs` | C/C++ 库模块 | 导出头文件目录，依赖该库的模块会自动添加该 include 路径 | `export_include_dirs: ["include"]` |

### 2. 编译选项属性
| 属性名 | 适用模块 | 核心作用 | 示例 |
|--------|----------|----------|------|
| `cflags` | C/C++ 模块 | C/C++ 通用编译选项，支持宏定义、警告配置等 | `cflags: ["-DDEBUG", "-Wall", "-O2"]` |
| `conlyflags` | C/C++ 模块 | 仅对 C 语言生效的编译选项 | `conlyflags: ["-std=c99"]` |
| `cppflags` | C/C++ 模块 | 仅对 C++ 语言生效的编译选项 | `cppflags: ["-std=c++17", "-fno-exceptions"]` |
| `ldflags` | C/C++ 模块 | 链接选项，用于配置链接器参数 | `ldflags: ["-Wl,--as-needed"]` |
| `javacflags` | Java 模块 | Java 编译选项 | `javacflags: ["-Xlint:unchecked"]` |
| `kotlincflags` | Kotlin 模块 | Kotlin 编译选项 | `kotlincflags: ["-Xjvm-default=all"]` |
| `stl` | C/C++ 模块 | 指定 C++ 标准库实现 | `stl: "libc++_static"` / `stl: "none"` |

### 3. 分区与安装属性
用于控制模块编译后的安装分区、是否安装、权限等，核心如下：

| 属性名 | 核心作用 | 可选值/示例 |
|--------|----------|-------------|
| `vendor: true` | 标记模块安装到 vendor 分区（厂商分区） | `true`/`false` |
| `product_specific: true` | 标记模块安装到 product 分区（产品定制分区） | `true`/`false` |
| `odm: true` | 标记模块安装到 odm 分区（设备定制分区） | `true`/`false` |
| `system_ext_specific: true` | 标记模块安装到 system_ext 分区 | `true`/`false` |
| `installable` | 控制模块是否安装到镜像，默认 true | `installable: false`（仅编译不安装，如测试库） |
| `required` | 声明模块运行依赖的其他模块，系统打包时会一同包含 | `required: ["demo_config.conf"]` |

### 4. SDK 与平台 API 属性
| 属性名 | 核心作用 | 示例 |
|--------|----------|------|
| `sdk_version` | 指定模块编译依赖的 Android SDK 版本 | `sdk_version: "34"` / `sdk_version: "current"` |
| `min_sdk_version` | 指定模块支持的最低 Android 版本 | `min_sdk_version: "29"` |
| `platform_apis` | 允许使用系统隐藏 API（平台私有API），仅系统级模块可用 | `platform_apis: true` |
| `privileged` | 标记应用为特权应用，安装到 priv-app 目录，可申请系统级权限 | `privileged: true` |
| `certificate` | 指定 APK 签名方式 | `certificate: "platform"` / `certificate: "testkey"` / 自定义证书模块 |

## 五、代码复用：defaults 模板继承
Android.bp 无类继承机制，通过 `defaults` 模块实现属性复用，解决多模块重复配置的问题，核心规则如下：

1.  先通过 `xx_defaults` 模块定义通用配置模板（如 `cc_defaults`、`java_defaults`）
2.  业务模块通过 `defaults: ["模板名"]` 继承模板的所有属性
3.  业务模块可重写模板中的属性，重写后以模块自身配置为准
4.  支持同时继承多个 defaults 模板，按数组顺序合并属性

```bp
// 1. 定义C++通用配置模板
cc_defaults {
    name: "common_cc_config",
    cflags: ["-Wall", "-Werror", "-O2"],
    shared_libs: ["liblog", "libbase", "libutils"],
    stl: "libc++_shared",
}

// 2. 定义调试版本专属配置
cc_defaults {
    name: "debug_cc_config",
    cflags: ["-DDEBUG", "-g"],
    optimized: false,
}

// 3. 业务模块继承模板，无需重复配置
cc_binary {
    name: "demo",
    // 继承多个defaults模板
    defaults: ["common_cc_config", "debug_cc_config"],
    // 自身独有配置
    srcs: ["main.cpp", "demo.cpp"],
    // 重写模板中的cflags，追加额外配置
    cflags: ["-DDEMO_VERSION=1.0"],
}
```

### 优先级规则
模块最终属性的优先级：`模块自身配置 > defaults模板配置 > 系统默认值`

## 六、条件控制语法
Android.bp 设计上无原生 `if-else` 分支语句，所有条件逻辑通过内置的条件块实现，核心分为 3 种场景。

### 1. 产品变量条件：product_variables
用于根据产品构建类型、产品特性开关控制模块属性，是最常用的条件控制方式。内置支持 `eng`/`userdebug`/`user` 构建类型、`debuggable` 等系统变量，也支持自定义产品变量。

#### 基础语法示例
```bp
cc_binary {
    name: "demo",
    srcs: ["main.cpp"],
    // 产品变量条件块
    product_variables: {
        // 1. 根据构建类型区分
        eng: {
            // eng版本开启debug日志、关闭优化
            cflags: ["-DDEBUG", "-DENG_BUILD"],
            optimized: false,
            srcs: ["debug_helper.cpp"],
        },
        userdebug: {
            cflags: ["-DDEBUG"],
        },
        user: {
            // user版本排除调试代码
            exclude_srcs: ["debug_helper.cpp"],
            cflags: ["-DRELEASE"],
        },
        // 2. 根据自定义产品变量控制
        nfc_support: {
            cflags: ["-DNFC_SUPPORT"],
            srcs: ["nfc_adapter.cpp"],
        },
    },
}
```

#### 优先级规则
`product_variables` 内的属性会覆盖模块顶层的同名属性，优先级：`product_variables > 模块顶层属性 > defaults模板`

### 2. 架构条件控制：arch
用于根据目标 CPU 架构（如 arm64、x86_64）配置差异化属性，语法如下：
```bp
cc_library {
    name: "libdemo",
    srcs: ["common.cpp"],
    // 按CPU架构区分配置
    arch: {
        arm64: {
            srcs: ["arm64/asm_helper.S"],
            cflags: ["-DARM64_ARCH"],
        },
        arm: {
            srcs: ["arm/asm_helper.S"],
            cflags: ["-DARM_ARCH"],
        },
        x86_64: {
            srcs: ["x86_64/asm_helper.S"],
            cflags: ["-DX86_64_ARCH"],
        },
        x86: {
            srcs: ["x86/asm_helper.S"],
            cflags: ["-DX86_ARCH"],
        },
    },
}
```

### 3. 自定义配置：soong_config
用于自定义全局配置开关，适合跨模块的统一条件控制，需要先定义配置命名空间，再在模块中使用，适合大型项目的定制化构建。

## 七、高级特性
### 1. 可见性控制：visibility
用于限制模块可被哪些其他模块引用，避免模块被意外依赖，默认情况下模块对所有其他模块可见。
```bp
cc_library {
    name: "libdemo_internal",
    srcs: ["internal.cpp"],
    // 仅允许同目录及子目录的模块引用
    visibility: [":__subpackages__"],
    // 也可指定具体模块/目录
    // visibility: ["//vendor/demo:__subpackages__", "//system/core/libdemo"]
}
```

### 2. 命名空间：namespace
用于解决模块名冲突问题，不同命名空间内的模块名可重复，通常用于厂商定制、第三方代码隔离。
```bp
// 定义命名空间
namespace {
    name: "vendor_demo",
    // 声明可见的其他命名空间
    imports: ["*"],
}

// 该模块属于vendor_demo命名空间，模块名可与全局空间重复
cc_binary {
    name: "demo",
    namespace: "vendor_demo",
    srcs: ["main.cpp"],
}
```

### 3. 模块启用/禁用：enabled
通过 `enabled` 属性控制模块是否参与编译，配合条件块可实现灵活的编译开关。
```bp
cc_binary {
    name: "demo_tool",
    srcs: ["tool.cpp"],
    // 仅在eng版本编译该模块
    product_variables: {
        eng: {
            enabled: true,
        },
        user: {
            enabled: false,
        },
    },
}
```

## 八、配套工具
### 1. bpfmt 格式化工具
Android 源码内置的 Android.bp 格式化工具，统一代码风格，用法：
```bash
# 格式化单个文件
bpfmt -w Android.bp
# 递归格式化目录下所有bp文件
bpfmt -w ./
```

### 2. androidmk 转换工具
用于将传统的 Android.mk 文件自动转换为 Android.bp 文件，大幅降低迁移成本。
```bash
# 转换Android.mk为Android.bp
androidmk Android.mk > Android.bp
```
> 注意：复杂的条件逻辑、自定义 Make 函数无法自动转换，需要手动适配为 Soong 支持的语法。

### 3. soong_docs 文档生成工具
用于生成当前 Soong 版本支持的所有模块类型、属性的参考文档，解决语法查询问题。
```bash
# 在AOSP根目录执行，生成模块参考文档
m soong_docs
# 生成的文档路径：out/soong/docs/soong_build.html
```


