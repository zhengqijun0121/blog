# Dockerfile 语法详细总结


# Dockerfile 语法详细总结

Dockerfile 是用于构建 Docker 镜像的文本配置文件，由一系列指令和参数组成，遵循**分层构建**原则（每条指令生成一个独立镜像层），是 Docker 镜像构建的核心标准。

## 一、基础规则与前置约定

1. **指令大小写**：指令本身不区分大小写，行业惯例使用**大写**，便于和参数区分。

2. **执行顺序**：指令从上到下依次执行，Dockerfile 必须以 `FROM` 指令开头（解析器指令、前置 `ARG` 除外）。

3. **注释规则**：行首以 `#` 开头为注释，不支持行内注释（`#` 必须在行首，转义参数场景除外）。

4. **转义符**：默认转义符为 `\`，可通过解析器指令 `# escape=` 修改（常用于 Windows 容器路径适配）。

5. **环境变量引用**：通过 `$VAR` 或 `${VAR}` 引用，支持 Shell 风格的变量替换：

    - `${VAR:-default}`：变量未定义时使用默认值

    - `${VAR:+alt}`：变量已定义时使用替代值，否则为空

6. **.dockerignore 文件**：与 Dockerfile 同级，用于排除构建上下文中的文件（如源码、缓存、敏感信息），减少镜像体积、避免缓存失效、防止信息泄露，语法与 `.gitignore` 一致。

## 二、核心指令全详解

### （一）解析器指令（Parser Directives）

必须放在 Dockerfile 最顶部，仅能出现一次，用于配置 Dockerfile 解析器的行为，换行符、注释之外不能有其他内容。

1. **`escape`** ** 指令**

    - 语法：`# escape=\` 或 `# escape=` ```

    - 作用：修改默认转义符，Windows 容器中常用反引号避免与路径反斜杠冲突。

    - 示例：`# escape=` ```

2. **`syntax`** ** 指令**

    - 语法：`# syntax=docker/dockerfile:1.7-labs`

    - 作用：指定 Dockerfile 语法版本，启用 BuildKit 新特性（如多平台构建、缓存挂载、机密挂载等），推荐始终指定官方稳定版本。

---

### （二）基础镜像与元数据指令

#### 1. `FROM`（必选，核心指令）

指定构建的基础镜像，是 Dockerfile 的第一条有效指令（解析器指令、前置 `ARG` 除外），支持多阶段构建。

- 语法：

    ```Dockerfile

    FROM [--platform=<platform>] <image>[:<tag>] [AS <stage_name>]
    ```

- 参数说明：

    - `--platform`：指定镜像平台，如 `linux/amd64`、`linux/arm64`、`windows/amd64`，用于跨平台构建。

    - `image:tag`：基础镜像名称与标签，标签默认 `latest`，推荐固定版本号保证构建一致性。

    - `AS <stage_name>`：为当前构建阶段命名，多阶段构建中用于后续引用。

- 示例：

    ```Dockerfile

    # 单阶段构建
    FROM ubuntu:22.04

    # 多阶段构建
    FROM golang:1.22 AS builder
    ```

- 注意事项：

    - 一个 Dockerfile 可包含多个 `FROM`，实现多阶段构建，最终镜像仅保留最后一个阶段的内容。

    - 前置 `ARG` 仅在 `FROM` 行中有效，阶段内使用需重新声明。

    - 优先使用官方轻量镜像（alpine、slim 系列），减少镜像体积与攻击面。

#### 2. `ARG`（构建参数）

定义**仅在构建过程中生效**的变量，容器运行时不会保留，可通过构建命令覆盖默认值。

- 语法：

    ```Dockerfile

    ARG <name>[=<default_value>]
    ```

- 作用域：

    - **前置 ARG**（`FROM` 之前定义）：仅在 `FROM` 行中生效，每个 `FROM` 阶段需重新声明才能复用。

    - **阶段内 ARG**（`FROM` 之后定义）：从声明处到当前阶段结束有效，跨阶段需重新声明。

- 示例：

    ```Dockerfile

    # 前置ARG，用于指定基础镜像版本
    ARG UBUNTU_VERSION=22.04
    FROM ubuntu:${UBUNTU_VERSION}

    # 阶段内ARG
    ARG APP_VERSION=1.0.0
    RUN echo "App version: ${APP_VERSION}"
    ```

- 注意事项：

    - 与 `ENV` 核心区别：`ARG` 仅构建时有效，`ENV` 构建+运行时均有效。

    - 敏感信息（密钥、密码）禁止使用 `ARG`，`docker history` 可查看所有构建参数。

    - 可通过 `docker build --build-arg <key>=<value>` 覆盖默认值。

    - Docker 预定义 ARG（如 `HTTP_PROXY`、`TARGETARCH`）无需在 Dockerfile 中声明即可使用。

#### 3. `LABEL`（镜像元数据标签）

为镜像添加键值对形式的元数据，替代已废弃的 `MAINTAINER` 指令。

- 语法：

    ```Dockerfile

    LABEL <key1>=<value1> <key2>=<value2> ...
    ```

- 示例：

    ```Dockerfile

    LABEL maintainer="dev@example.com" \
          version="1.0.0" \
          description="Production image for backend service" \
          org.opencontainers.image.authors="Dev Team"
    ```

- 注意事项：

    - 多个标签可合并为一个 `LABEL` 指令，减少镜像层数。

    - 可通过 `docker inspect <image>` 查看镜像的所有标签。

    - 推荐遵循 OCI 镜像规范定义标准化标签。

---

### （三）内容操作指令

#### 1. `RUN`（构建时执行命令）

在当前镜像层之上执行命令并提交结果，是安装软件、编译代码、修改配置的核心指令。

- 两种语法格式：

    1. **Shell 格式**（默认）：`RUN <command>`

        - 底层通过 `/bin/sh -c` 执行，支持管道、通配符、环境变量等 Shell 特性。

    2. **Exec 格式**（推荐）：`RUN ["executable", "param1", "param2"]`

        - 直接执行可执行文件，不启动 Shell 进程，避免 PID1 进程异常、信号传递问题。

        - 必须使用**双引号**，符合 JSON 数组格式。

- 示例：

    ```Dockerfile

    # Shell格式：合并命令+清理缓存，减少镜像层
    RUN apt-get update && \
        apt-get install -y nginx curl && \
        rm -rf /var/lib/apt/lists/*

    # Exec格式
    RUN ["apt-get", "install", "-y", "nginx"]
    ```

- 高级特性（BuildKit）：`RUN --mount` 挂载增强

    ```Dockerfile

    # 缓存挂载：复用npm缓存，不写入镜像层，提升构建速度
    RUN --mount=type=cache,target=/root/.npm npm install

    # 机密挂载：读取构建时的密钥，不写入镜像历史
    RUN --mount=type=secret,id=npm_token,target=/root/.npmrc npm install

    # SSH挂载：复用宿主机SSH代理，拉取私有Git仓库，无需复制私钥
    RUN --mount=type=ssh git clone git@github.com:private/repo.git
    ```

- 注意事项：

    - 多条关联命令用 `&&` 合并为一个 `RUN`，减少镜像层数，同时避免缓存异常（如 `apt-get update` 与 `install` 必须同一条指令）。

    - 命令执行后必须清理缓存（如 apt、yum、npm 缓存），减小镜像体积。

    - Exec 格式不会进行环境变量替换，如需使用需手动指定 Shell：`RUN ["sh", "-c", "echo $HOME"]`。

#### 2. `COPY`（文件复制）

将构建上下文中的文件/目录，复制到镜像内的目标路径，语义清晰、缓存可控，是文件复制的首选指令。

- 语法：

    ```Dockerfile

    # 常规格式
    COPY [--chown=<user>:<group>] [--chmod=<mode>] [--from=<stage|image>] <src>... <dest>
    # 路径含空格的格式
    COPY [--chown=<user>:<group>] [--chmod=<mode>] [--from=<stage|image>] ["<src>",... "<dest>"]
    ```

- 核心参数：

    - `--chown`：指定复制后文件的所属用户与组，仅 Linux 容器有效。

    - `--chmod`：指定文件权限，如 `0755`、`0644`。

    - `--from`：从指定构建阶段（`AS` 命名的阶段）或外部镜像中复制文件，多阶段构建核心用法。

    - `src`：源路径，相对于构建上下文，支持通配符（`*`、`?`），**禁止使用 ** **`../`** ** 跳出上下文目录**。

    - `dest`：目标路径，镜像内的绝对路径，或相对于 `WORKDIR` 的相对路径。

- 示例：

    ```Dockerfile

    # 复制当前目录所有文件到镜像/app目录
    COPY . /app

    # 带权限配置复制
    COPY --chown=nginx:nginx --chmod=0644 ./nginx.conf /etc/nginx/nginx.conf

    # 多阶段构建：从builder阶段复制编译产物
    COPY --from=builder /app/build /app
    ```

- 注意事项：

    - 源路径为目录时，仅复制目录内的所有内容，而非目录本身。

    - 多个源路径时，目标路径必须以 `/` 结尾。

    - 源文件内容发生变化时，会触发该层及后续所有层缓存失效，因此推荐先复制依赖文件（如 `package.json`），再复制完整源码。

#### 3. `ADD`（高级文件复制）

功能与 `COPY` 基本一致，额外支持两个特性：**本地压缩包自动解压**、**远程 URL 下载**。

- 语法与 `COPY` 完全一致。

- 核心特性：

    1. 本地 tar/gzip/bzip2/xz 压缩包会自动解压到目标路径（远程 URL 下载的压缩包不会解压）。

    2. 支持 HTTP/HTTPS 远程 URL 作为源路径，自动下载到目标路径。

- 示例：

    ```Dockerfile

    # 本地压缩包自动解压
    ADD ./package.tar.gz /app

    # 远程文件下载
    ADD https://example.com/binary /usr/local/bin/
    ```

- 注意事项：

    - **非必要不使用 ** **`ADD`**，常规文件复制优先用 `COPY`，语义更清晰、缓存行为更可控。

    - 不推荐用 `ADD` 下载远程文件，应使用 `RUN curl/wget` 下载+执行+清理，合并为一层减少镜像体积。

    - 仅当需要自动解压本地压缩包时，使用 `ADD` 指令。

---

### （四）环境与运行配置指令

#### 1. `ENV`（环境变量）

设置环境变量，**构建过程中后续指令与容器运行时均有效**，是配置应用运行环境的核心指令。

- 语法：

    ```Dockerfile

    # 单变量格式
    ENV <key> <value>
    # 多变量格式（推荐）
    ENV <key1>=<value1> <key2>=<value2> ...
    ```

- 示例：

    ```Dockerfile

    ENV JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64 \
        PATH=$PATH:$JAVA_HOME/bin \
        APP_ENV=production \
        LOG_LEVEL=info
    ```

- 注意事项：

    - 环境变量会永久保留在镜像中，容器运行时可通过 `docker run -e <key>=<value>` 覆盖。

    - 敏感信息（密钥、密码）禁止使用 `ENV`，会通过 `docker history` 泄露。

    - 多变量合并为一个 `ENV` 指令，减少镜像层数。

#### 2. `WORKDIR`（工作目录）

为后续的 `RUN`/`CMD`/`ENTRYPOINT`/`COPY`/`ADD` 指令设置工作目录，目录不存在时会自动创建。

- 语法：

    ```Dockerfile

    WORKDIR /path/to/workdir
    ```

- 示例：

    ```Dockerfile

    WORKDIR /app
    COPY . .
    RUN npm install

    # 相对路径，基于上一个WORKDIR，最终路径为/app/logs
    WORKDIR logs
    ```

- 注意事项：

    - 禁止使用 `RUN cd xxx && command` 切换目录，`cd` 仅在当前 `RUN` 指令有效，后续指令会恢复默认目录。

    - 推荐使用绝对路径，避免路径混乱。

    - 容器启动后的默认工作目录，为最后一个 `WORKDIR` 指定的路径。

#### 3. `USER`（运行用户）

为后续的 `RUN`/`CMD`/`ENTRYPOINT` 指令指定运行用户与用户组，同时设置容器启动时的默认用户。

- 语法：

    ```Dockerfile

    USER <user>[:<group>]
    USER <UID>[:<GID>]
    ```

- 示例：

    ```Dockerfile

    # 提前创建非root用户
    RUN groupadd -r appgroup && useradd -r -g appgroup appuser
    # 切换用户
    USER appuser
    ```

- 注意事项：

    - 用户/用户组必须提前创建，否则会报错。

    - 生产环境推荐使用非 root 用户运行容器，提升安全性，减少攻击面。

    - 容器运行时可通过 `docker run --user` 覆盖默认用户。

    - 切换用户后，需确保工作目录、文件的权限与用户匹配，可通过 `COPY --chown` 提前设置。

#### 4. `SHELL`（默认Shell配置）

修改 Shell 格式指令（`RUN`/`CMD`/`ENTRYPOINT`）的默认 Shell 解释器。

- 语法：

    ```Dockerfile

    SHELL ["executable", "parameters"]
    ```

- 示例：

    ```Dockerfile

    # Linux切换默认Shell为bash
    SHELL ["/bin/bash", "-c"]
    RUN echo $BASH_VERSION

    # Windows切换为PowerShell
    SHELL ["powershell", "-Command"]
    ```

- 注意事项：

    - 必须使用 Exec 格式（JSON 数组、双引号）。

    - Linux 默认值为 `["/bin/sh", "-c"]`，Windows 默认值为 `["cmd", "/S", "/C"]`。

    - 可多次使用，每次修改仅对后续指令生效。

---

### （五）容器启动与生命周期指令

#### 1. `CMD`（容器启动默认命令）

指定容器启动时**默认执行的命令与参数**，`docker run` 时若指定了启动命令，会完全覆盖 `CMD` 内容。

- 三种语法格式：

    1. **Exec 格式（推荐）**：`CMD ["executable", "param1", "param2"]`

        - 直接执行可执行文件，不启动 Shell，PID1 为目标进程，可正常接收系统信号，实现优雅退出。

    2. **默认参数格式**：`CMD ["param1", "param2"]`

        - 必须与 Exec 格式的 `ENTRYPOINT` 配合使用，为 `ENTRYPOINT` 提供默认参数。

    3. **Shell 格式**：`CMD command param1 param2`

        - 底层通过 `/bin/sh -c` 执行，PID1 为 sh 进程，无法正常接收停止信号，不推荐使用。

- 示例：

    ```Dockerfile

    # Exec格式（推荐）
    CMD ["nginx", "-g", "daemon off;"]

    # 为ENTRYPOINT提供默认参数
    ENTRYPOINT ["nginx", "-g"]
    CMD ["daemon off;"]
    ```

- 注意事项：

    - 一个 Dockerfile 中只能有一个 `CMD`，多个 `CMD` 仅最后一个生效。

    - `docker run <image> [command]` 会完全覆盖 `CMD` 的内容，不会追加参数。

    - Shell 格式的 `CMD` 会忽略所有 `ENTRYPOINT` 指令。

#### 2. `ENTRYPOINT`（容器入口点）

指定容器启动时执行的入口命令，**不会被 ** **`docker run`** ** 的命令行参数覆盖**，而是将参数传递给 `ENTRYPOINT`，仅可通过 `--entrypoint` 强制覆盖。

- 两种语法格式：

    1. **Exec 格式（推荐）**：`ENTRYPOINT ["executable", "param1", "param2"]`

        - 生产环境首选，PID1 为目标进程，可正常接收系统信号，支持与 `CMD` 配合实现默认参数+自定义参数。

    2. **Shell 格式**：`ENTRYPOINT command param1 param2`

        - 底层通过 `/bin/sh -c` 执行，PID1 为 sh 进程，无法接收停止信号，且会忽略 `CMD` 和 `docker run` 的参数，强烈不推荐。

- 核心用法（Exec 格式 + CMD 组合）：

    ```Dockerfile

    # 固定入口命令
    ENTRYPOINT ["echo", "Hello"]
    # 提供默认参数，可被docker run的参数覆盖
    CMD ["World"]
    ```

    - 执行 `docker run <image>`：输出 `Hello World`

    - 执行 `docker run <image> Docker`：输出 `Hello Docker`

- 注意事项：

    - 一个 Dockerfile 中只能有一个 `ENTRYPOINT`，多个仅最后一个生效。

    - 常用于制作可执行镜像（如命令行工具），固定入口命令，仅允许传递参数。

    - 可通过 `docker run --entrypoint <command> <image>` 强制覆盖入口点。

#### 3. `HEALTHCHECK`（健康检查）

设置容器的健康检查命令，Docker 会定期执行命令判断容器是否健康，用于服务可用性检测。

- 语法：

    ```Dockerfile

    # 设置健康检查
    HEALTHCHECK [OPTIONS] CMD <command>
    # 禁用基础镜像继承的健康检查
    HEALTHCHECK NONE
    ```

- 可选参数：

|    参数|    作用|    默认值|
|---|---|---|
|    `--interval=<duration>`|    两次健康检查的间隔时间|    30s|
|    `--timeout=<duration>`|    单次检查的超时时间，超时视为检查失败|    30s|
|    `--start-period=<duration>`|    容器启动初始化时间，此期间的失败不计入重试次数|    0s|
|    `--retries=<number>`|    连续失败多少次后，标记容器为不健康|    3次|
- 示例：

    ```Dockerfile

    HEALTHCHECK --interval=30s --timeout=3s --start-period=60s --retries=3 \
      CMD curl -f http://localhost:8080/health || exit 1
    ```

- 注意事项：

    - 一个 Dockerfile 中只能有一个 `HEALTHCHECK`，多个仅最后一个生效。

    - 命令退出码决定健康状态：`0=健康`，`1=不健康`，`2=保留值（禁止使用）`。

    - 健康状态可通过 `docker ps` 查看，`docker inspect` 可查看详细检查历史。

    - 检查命令必须是执行后立即退出的一次性命令，禁止使用持续运行的进程。

#### 4. `STOPSIGNAL`（容器停止信号）

设置容器停止时，Docker 发送给 PID1 进程的系统信号，默认值为 `SIGTERM`（15）。

- 语法：

    ```Dockerfile

    STOPSIGNAL <signal>
    ```

- 示例：

    ```Dockerfile

    STOPSIGNAL SIGINT
    STOPSIGNAL 9
    ```

- 注意事项：

    - 信号可使用名称（如 `SIGTERM`）或数字（如 `15`）。

    - 用于适配需要自定义信号才能优雅退出的应用。

    - `docker stop` 会先发送该信号，等待超时时间（默认10s）后，发送 `SIGKILL` 强制终止容器。

#### 5. `ONBUILD`（触发器指令）

设置构建触发器，当当前镜像被用作其他 Dockerfile 的基础镜像时，会在 `FROM` 执行后、其他指令执行前，触发执行 `ONBUILD` 绑定的指令。

- 语法：

    ```Dockerfile

    ONBUILD <INSTRUCTION>
    ```

- 示例：

    ```Dockerfile

    # 基础镜像Dockerfile
    FROM node:18
    WORKDIR /app
    ONBUILD COPY package*.json ./
    ONBUILD RUN npm install
    ONBUILD COPY . .
    CMD ["npm", "start"]

    # 子镜像Dockerfile，无需重复编写构建逻辑
    FROM 上述基础镜像
    # 构建时会自动执行3条ONBUILD指令
    ```

- 注意事项：

    - `ONBUILD` 仅在子镜像构建时触发一次，孙子镜像不会继承触发。

    - 不支持嵌套 `ONBUILD ONBUILD`，也不能绑定 `FROM`、`MAINTAINER` 指令。

    - 适合制作通用构建基础镜像，简化子镜像的 Dockerfile 编写。

---

### （六）网络与存储指令

#### 1. `EXPOSE`（端口声明）

声明容器运行时监听的端口与协议，仅作为镜像的文档说明，**不会自动发布端口**。

- 语法：

    ```Dockerfile

    EXPOSE <port> [<port>/<protocol>...]
    ```

- 示例：

    ```Dockerfile

    EXPOSE 80/tcp
    EXPOSE 443/tcp
    EXPOSE 53/udp
    ```

- 注意事项：

    - 默认协议为 TCP，可指定 UDP。

    - 端口发布需通过 `docker run -p`（指定端口映射）或 `-P`（自动映射所有声明端口）实现。

    - 即使不写 `EXPOSE`，容器内服务也可监听端口，该指令仅为元数据，不影响功能。

#### 2. `VOLUME`（匿名卷声明）

声明容器内的目录为匿名卷，容器运行时自动挂载，避免数据写入容器可写层，防止容器删除时数据丢失。

- 语法：

    ```Dockerfile

    VOLUME ["<path1>", "<path2>"]
    VOLUME <path1> <path2>
    ```

- 示例：

    ```Dockerfile

    VOLUME ["/var/lib/mysql", "/var/log/nginx"]
    VOLUME /data
    ```

- 注意事项：

    - 声明后，该目录的所有写入都会进入卷，而非容器可写层。

    - `VOLUME` 指令之后的 `RUN` 指令，若修改该目录的内容，会完全无效（卷在容器运行时才挂载）。

    - 容器运行时可通过 `docker run -v` 覆盖挂载的目录与卷类型。

    - 禁止对配置文件目录使用 `VOLUME`，会导致用户无法修改配置。

## 三、多阶段构建核心用法

多阶段构建是 Dockerfile 17.05+ 支持的核心特性，通过多个 `FROM` 指令划分构建阶段，最终镜像仅保留最后一个阶段的内容，彻底解决「构建依赖冗余、镜像体积过大」的问题。

### 核心示例

```Dockerfile

# 阶段1：构建阶段，包含完整编译环境
FROM golang:1.22 AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o myapp main.go

# 阶段2：运行阶段，极简基础镜像，仅保留编译产物
FROM alpine:3.19
WORKDIR /app
# 从builder阶段复制编译好的二进制文件
COPY --from=builder /app/myapp .
EXPOSE 8080
CMD ["./myapp"]
```

### 核心优势

1. 极致减小镜像体积，剔除编译工具、源码、中间文件、依赖缓存。

2. 提升镜像安全性，减少攻击面，运行镜像仅包含业务运行必需的文件。

3. 分离构建与运行环境，构建阶段使用完整工具链，运行阶段使用轻量基础镜像。

4. 优化构建缓存，依赖下载、编译步骤分层缓存，大幅提升重复构建速度。

## 四、镜像缓存机制与优化规则

Docker 构建时会为每条指令生成缓存层，指令与上下文无变化时直接复用缓存，大幅提升构建速度。

### 核心缓存规则

1. 从基础镜像开始，自上而下比对每条指令，指令字符串与对应上下文无变化则复用缓存。

2. `COPY`/`ADD` 指令会比对文件的校验和（checksum），校验和变化则缓存失效，后续所有层全部重新构建。

3. `RUN` 等其他指令仅比对指令字符串，指令不变则复用缓存，不关心执行结果是否变化。

4. 某一层缓存失效后，后续所有层的缓存全部失效。

### 缓存优化技巧

- 不变指令前置，易变指令后置：先复制依赖文件、安装依赖，最后复制业务源码，避免源码变化导致依赖层缓存失效。

- 合并关联的 `RUN` 指令，减少镜像层数，同时避免缓存异常（如 `apt-get update` 与 `install` 必须合并）。

- 合理使用 `.dockerignore`，排除无关文件，避免上下文变化导致缓存意外失效。

- 使用 BuildKit 缓存挂载（`RUN --mount=type=cache`），复用依赖缓存，不写入镜像层。

## 五、生产环境最佳实践

1. 优先使用官方轻量基础镜像（alpine、slim 系列），固定镜像版本，避免 `latest` 标签导致构建不一致。

2. 一个容器仅运行一个进程，保持容器的单一职责，便于日志收集、故障排查与水平扩展。

3. 合并 `RUN` 指令，命令执行后必须清理缓存，最小化镜像层数与体积。

4. 常规文件复制优先使用 `COPY`，仅自动解压场景使用 `ADD`，禁止用 `ADD` 下载远程文件。

5. 严格区分 `ARG` 与 `ENV`，构建时变量用 `ARG`，运行时环境用 `ENV`，敏感信息禁止写入 Dockerfile。

6. 生产环境使用非 root 用户运行容器，提前配置用户权限与文件归属。

7. 始终使用 Exec 格式编写 `ENTRYPOINT` 与 `CMD`，保证进程 PID1 正常接收系统信号，实现优雅退出。

8. 强制使用多阶段构建，分离构建与运行环境，最小化最终镜像内容。

9. 始终编写 `.dockerignore` 文件，排除源码、缓存、敏感文件、本地构建产物。

10. 为有状态服务添加 `HEALTHCHECK` 健康检查，标注 `LABEL` 元数据，声明 `EXPOSE` 端口，提升镜像可维护性。

11. 禁止在 Dockerfile 中硬编码密钥、密码、证书等敏感信息，通过构建机密、运行时环境变量或挂载配置注入。

12. 避免在 `VOLUME` 之后修改目录内容，禁止对配置目录使用 `VOLUME`。
> （注：文档部分内容可能由 AI 生成）
