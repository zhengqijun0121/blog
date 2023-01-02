# Bash 调试技巧


## Bash 调试技巧

`bash` 脚本是一种常用的脚本。当 `bash` 脚本执行出现错误的时候，我们可以通过调试的方式来定位问题所在。

### Bash 调试常用选项

| 选项        | 说明                    |
| ----------- | ----------------------- |
| -v          | 打印执行命令            |
| -x          | 类似 `-v`，并扩展了命令 |
| -e          | 发生错误终止执行        |
| -n          | 只检查语法，不执行脚本  |
| -o pipefail | 返回管道的错误状态      |

### 跟踪脚本的执行

#### 输出调试信息

正常情况下，通过 `bash test.sh` 或 `./test.sh` 执行脚本，利用 `echo` 或者 `printf` 命令可以输出一些信息，但是不能输出命令的具体执行情况。

下面有两种方式利用 `-x` 选项，可以打印执行命令，方便调试。

1. 执行命令

```bash
bash -x test.sh
or
bash --debug test.sh
```

2. 修改脚本

```bash
#!/bin/bash

set -x  # Enable debugging output
# some code here
set +x  # Disable debugging output
```

#### 输出行号和函数名

`PS4` 是 Linux/Unix 下的一个用于控制脚本调试显示信息的环境变量。

其中常用的变量有：

- `${BASH_SOURCE}`：表示当前执行脚本的相对路径
- `${LINENO}`：表示行号
- `${FUNCNAME}`：表示当前执行函数名

```bash
#!/bin/bash

PS4='+${BASH_SOURCE}:${LINENO} ${FUNCNAME}: '

function hello()
{
    echo "Hello World"
}
```

### 检查语法错误

语法错误是 shell 脚本执行错误的原因之一。执行脚本的时候加上 `-n` 选项, 当脚本有语法错误，不会继续执行，而是打印错误信息。

```bash
bash -n test.sh
```

### 发生错误终止运行

一般情况下，脚本执行时发生错误了，还是会继续执行后面的命令。执行脚本的时候加上 `-e` 选项，可以避免发生错误的时候，脚本继续执行。

```bash
bash -e test.sh
```

### 管道命令错误终止运行

`-e` 选项有个特殊的情况，不适用于管道命令。执行脚本的时候加上 `-o pipefail` 选项，可以检测管道命令发生错误。

```bash
bash -o pipefail test.sh
```

### 检查不存在的变量

执行脚本的时候，遇到不存在的变量，默认会忽略它。执行脚本的时候加上 `-u` 选项，可以检测变量是否存在。

```bash
bash -u test.sh
```


