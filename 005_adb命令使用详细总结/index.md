# adb 命令使用详细总结


# ADB 命令使用详细总结
ADB（Android Debug Bridge，安卓调试桥）是 Android SDK 提供的**客户端-服务端架构**调试工具，可实现 PC 与 Android 设备/模拟器的全场景通信，是开发、测试、玩机的核心工具。其架构分为三部分：
- **Client（客户端）**：运行在 PC 端，用于发送命令
- **Daemon（adbd 守护进程）**：运行在设备端后台，执行接收的命令
- **Server（服务端）**：运行在 PC 端后台，管理客户端与守护进程的通信，默认占用 5037 端口

## 一、环境配置与基础验证
### 1. 环境安装
无需安装完整 Android Studio，直接下载官方精简包：
1.  下载 [Android SDK Platform Tools](https://developer.android.com/studio/releases/platform-tools)，解压到本地任意目录
2.  配置环境变量：将解压后的目录路径添加到系统环境变量 `Path` 中
3.  重启终端，执行验证命令

### 2. 基础验证命令
| 命令 | 核心作用 |
| :--- | :--- |
| `adb version` | 查看 ADB 版本，验证环境配置是否成功 |
| `adb help` | 查看 ADB 所有命令帮助文档 |

## 二、设备连接与管理
### 1. 基础连接方式
#### （1）USB 连接（最稳定）
1.  设备开启「开发者选项」，开启「USB 调试」
2.  设备连接 PC，在弹窗中勾选「始终允许此计算机调试」并确认
3.  执行 `adb devices` 验证连接状态

#### （2）无线连接（同局域网）
**安卓 10 及以下**：
1.  先通过 USB 连接设备，确保连接正常
2.  执行 `adb tcpip 5555`，开启设备 TCP 监听端口
3.  断开 USB，查看设备局域网 IP（设置-关于手机-状态信息）
4.  执行 `adb connect 设备IP:5555` 完成连接

**安卓 11 及以上（官方推荐，无需 USB）**：
1.  开发者选项中开启「无线调试」，进入查看配对码、IP 与端口
2.  执行 `adb pair 设备IP:配对端口 配对码` 完成配对
3.  执行 `adb connect 设备IP:连接端口` 完成连接

### 2. 核心设备管理命令
| 命令 | 核心作用 | 示例与注意事项 |
| :--- | :--- | :--- |
| `adb devices` | 列出所有已连接设备及状态 | 状态说明：`device`（正常连接）、`unauthorized`（未授权）、`offline`（设备无响应）；加 `-l` 显示设备详细信息（型号等） |
| `adb -s <设备序列号> <命令>` | 多设备连接时，指定目标设备执行命令 | 示例：`adb -s RZCT809FTQM shell wm size` |
| `adb -d <命令>` | 指定当前唯一通过 USB 连接的设备为目标 | 多设备场景快速指定，无需输入序列号 |
| `adb -e <命令>` | 指定当前唯一运行的模拟器为目标 | 同上，模拟器专属 |
| `adb kill-server` | 终止 ADB 服务端进程 | 解决连接异常、端口占用等问题，常与 start-server 配对使用 |
| `adb start-server` | 启动 ADB 服务端进程 | 服务未启动时自动执行，无需手动调用 |
| `adb connect <ip:port>` | 无线连接设备 | 示例：`adb connect 192.168.1.100:5555` |
| `adb disconnect <ip:port>` | 断开指定无线设备 | 不加参数时，断开所有无线连接 |
| `adb get-serialno` | 获取设备序列号 | 快速查看设备唯一标识 |
| `adb get-state` | 获取设备连接状态 | 输出：device/offline/unknown |

### 3. 设备权限与重启命令
| 命令 | 核心作用 | 注意事项 |
| :--- | :--- | :--- |
| `adb root` | 以 root 权限重启设备端 adbd 进程 | 仅已 root 设备支持，安卓高版本需关闭 AVB 校验 |
| `adb remount` | 重新挂载 /system 等系统分区为可读写 | 需先执行 adb root，安卓 10+ 需先执行 adb disable-verity |
| `adb reboot` | 正常重启设备 | 无参数直接重启到系统 |
| `adb reboot bootloader` | 重启进入 Fastboot 模式（线刷模式） | 刷机、解锁 Bootloader 常用 |
| `adb reboot recovery` | 重启进入 Recovery 模式（卡刷模式） | 卡刷 OTA、双清常用 |
| `adb reboot edl` | 重启进入 9008 深度刷机模式 | 仅高通芯片设备支持 |

## 三、高频核心命令分类详解
### 1. 应用管理命令（开发/测试高频）
核心通过 `pm`（Package Manager，包管理器）和 `am`（Activity Manager，活动管理器）实现，所有命令均需指定**应用包名**（非 APK 文件名）。

#### （1）安装与卸载
| 命令 | 核心作用 | 常用参数与示例 |
| :--- | :--- | :--- |
| `adb install <本地APK路径>` | 安装 APK 到设备 | 核心参数：<br>- `-r`：覆盖安装，保留应用数据<br>- `-d`：允许降级覆盖安装<br>- `-s`：安装到 SD 卡<br>- `-g`：安装时授予所有运行时权限<br>示例：`adb install -r -g D:\test.apk` |
| `adb uninstall <包名>` | 卸载指定应用 | 参数 `-k`：卸载应用但保留数据和缓存<br>示例：`adb uninstall -k com.example.app` |

#### （2）应用信息查询
| 命令 | 核心作用 | 示例与过滤技巧 |
| :--- | :--- | :--- |
| `adb shell pm list packages` | 列出设备所有已安装应用的包名 | 核心过滤参数：<br>- `-s`：仅列出系统应用<br>- `-3`：仅列出第三方应用<br>- `-f`：显示 APK 安装路径<br>- `-i`：显示应用安装来源<br>过滤示例：`adb shell pm list packages | grep wechat`（Windows 用 findstr） |
| `adb shell pm path <包名>` | 查看指定应用 APK 的安装路径 | 示例：`adb shell pm path com.tencent.mm` |
| `adb shell pm dump <包名>` | 查看应用全量详细信息 | 包含版本号、权限、组件、安装时间、签名信息等，可配合 grep 过滤指定字段 |
| `adb shell dumpsys package <包名>` | 同 pm dump，查看应用包详细信息 | 示例：`adb shell dumpsys package com.example.app | grep version` |

#### （3）应用状态与权限管理
| 命令 | 核心作用 | 注意事项 |
| :--- | :--- | :--- |
| `adb shell pm clear <包名>` | 清除应用的所有数据与缓存 | 等同于应用的「清除数据」，执行后应用恢复到首次安装状态 |
| `adb shell pm disable-user <包名>` | 禁用指定应用（含系统应用） | 免 root 即可禁用系统预装应用，无卸载风险 |
| `adb shell pm enable <包名>` | 启用已禁用的应用 | 与 disable-user 配对使用 |
| `adb shell pm grant <包名> <权限名>` | 给应用授予运行时权限 | 示例：`adb shell pm grant com.example.app android.permission.CAMERA` |
| `adb shell pm revoke <包名> <权限名>` | 撤销应用的运行时权限 | 同上，反向操作 |

#### （4）应用进程与组件控制
| 命令 | 核心作用 | 示例 |
| :--- | :--- | :--- |
| `adb shell am start -n <包名/Activity全路径>` | 启动指定应用的 Activity | 示例：`adb shell am start -n com.example.app/.MainActivity` |
| `adb shell am force-stop <包名>` | 强制停止应用，杀死所有相关进程 | 完全终止应用运行，后台服务也会停止 |
| `adb shell am kill <包名>` | 杀死应用后台进程，不影响前台使用 | 仅杀后台，不会强制关闭应用，与 force-stop 有本质区别 |
| `adb shell am startservice -n <包名/Service全路径>` | 启动指定服务 | 示例：`adb shell am startservice -n com.example.app/.TestService` |
| `adb shell am broadcast -a <广播动作>` | 发送自定义广播 | 示例：`adb shell am broadcast -a com.example.app.TEST_ACTION` |

### 2. 文件传输命令
实现 PC 与 Android 设备的双向文件/文件夹传输，注意安卓 10+ 引入分区存储，访问 `/sdcard/Android/data` 等目录需特殊权限。

| 命令 | 核心作用 | 示例与注意事项 |
| :--- | :--- | :--- |
| `adb push <本地路径> <设备路径>` | 将本地文件/文件夹推送到设备 | 示例：`adb push D:\test\ /sdcard/Download/` <br>注意：目标路径需有写入权限，失败可先执行 adb root |
| `adb pull <设备路径> <本地路径>` | 将设备文件/文件夹拉取到本地 | 示例：`adb pull /sdcard/Download/test.apk D:\` <br>本地路径省略时，默认拉取到终端当前目录 |

### 3. 日志与调试命令
#### （1）logcat 日志抓取（崩溃/异常排查核心）
`adb logcat` 是抓取设备系统与应用日志的核心命令，支持多维度过滤、格式化输出、持久化存储。

| 命令 | 核心作用 | 示例 |
| :--- | :--- | :--- |
| `adb logcat` | 实时输出设备全量日志 | 终端持续滚动，Ctrl+C 终止输出 |
| `adb logcat -c` | 清空设备现有日志缓存 | 排查新问题前执行，避免旧日志干扰 |
| `adb logcat > log.txt` | 将日志输出保存到本地文件 | 示例：`adb logcat -v time > app_log.txt` |
| `adb logcat -v time` | 带时间戳格式化输出日志 | 最常用的格式化参数，精准定位日志发生时间 |
| `adb logcat *:<级别>` | 按日志级别过滤 | 级别优先级：V(Verbose)<D(Debug)<I(Info)<W(Warn)<E(Error)<F(Fatal)<S(静默) <br>示例：`adb logcat *:E` 仅输出错误及以上级别日志 |
| `adb logcat <标签>:<级别> *:S` | 按日志标签过滤 | 示例：`adb logcat MyApp:D *:S` 仅输出 MyApp 标签的 Debug 及以上日志 |
| `adb logcat --pid=$(adb shell pidof -s <包名>)` | 仅输出指定应用的日志 | 精准过滤单个应用，排查专属问题 |

#### （2）其他调试命令
| 命令 | 核心作用 | 说明 |
| :--- | :--- | :--- |
| `adb bugreport > bugreport.zip` | 生成系统全量 bug 报告 | 包含系统状态、进程信息、历史日志、崩溃记录等，安卓 7.0+ 输出 zip 压缩包 |
| `adb jdwp` | 列出设备上可调试的 Java 进程 PID | 用于远程调试、端口映射 |
| `adb forward <本地端口> <设备端口>` | 端口正向转发 | 将 PC 端端口映射到设备端，示例：`adb forward tcp:8080 tcp:8080`，PC 访问 8080 端口即可访问设备 8080 端口 |
| `adb reverse <设备端口> <本地端口>` | 端口反向转发 | 安卓 5.0+ 支持，将设备端端口映射到 PC 端，示例：`adb reverse tcp:8080 tcp:8080`，设备访问 8080 端口即可访问 PC 8080 端口 |

### 4. 系统控制与模拟输入命令
#### （1）模拟输入（自动化核心）
通过 `input` 命令模拟用户触摸、按键、文本输入等操作，无需 root 即可执行。

| 命令 | 核心作用 | 示例 |
| :--- | :--- | :--- |
| `adb shell input tap <x> <y>` | 模拟点击屏幕指定坐标 | 示例：`adb shell input tap 500 1000` |
| `adb shell input swipe <x1> <y1> <x2> <y2> <时长(ms)>` | 模拟滑动操作 | 示例：`adb shell input swipe 300 1000 300 500 300`（向上滑动解锁） |
| `adb shell input text <内容>` | 输入文本内容 | 仅支持英文、数字和符号，不支持中文，示例：`adb shell input text "hello123"` |
| `adb shell input keyevent <键值>` | 模拟系统按键操作 | 高频键值示例：<br>- 4：返回键<br>- 3：主页键<br>- 26：电源键（亮屏/熄屏）<br>- 224：点亮屏幕<br>- 223：熄灭屏幕<br>- 85：播放/暂停<br>- 87：下一首<br>- 88：上一首 |

#### （2）系统属性与设置控制
| 命令 | 核心作用 | 示例 |
| :--- | :--- | :--- |
| `adb shell getprop <属性名>` | 查看系统属性 | 高频示例：<br>`adb shell getprop ro.build.version.release`（查看安卓系统版本）<br>`adb shell getprop ro.product.model`（查看设备型号）<br>不加属性名时，输出所有系统属性 |
| `adb shell setprop <属性名> <值>` | 修改系统属性 | 部分属性需 root 权限，示例：`adb shell setprop service.adb.tcp.port 5555` |
| `adb shell wm size` | 查看屏幕分辨率 | 加参数可修改：`adb shell wm size 1080x1920`，`adb shell wm size reset` 恢复默认 |
| `adb shell wm density` | 查看屏幕密度（DPI） | 加参数可修改：`adb shell wm density 480`，`adb shell wm density reset` 恢复默认 |
| `adb shell svc wifi enable/disable` | 开启/关闭 WiFi | 免 root 执行，同理 `svc data enable` 控制移动数据，`svc bluetooth enable` 控制蓝牙 |
| `adb shell svc power stayon true` | 设置充电时屏幕保持常亮 | 参数：true/false/usb/ac，分别对应所有场景/仅USB充电/仅AC充电 |
| `adb shell settings put global http_proxy <IP:端口>` | 设置系统全局 HTTP 代理 | 抓包场景高频使用，清空代理：`adb shell settings put global http_proxy :0` |

### 5. 性能与状态监控命令
| 命令 | 核心作用 | 示例 |
| :--- | :--- | :--- |
| `adb shell dumpsys activity activities` | 查看当前应用 Activity 栈 | 快速定位当前前台界面的 Activity |
| `adb shell dumpsys window | grep mCurrentFocus` | 查看当前前台界面的 Activity 完整路径 | Windows 用 findstr 替代 grep，快速获取包名和 Activity 名 |
| `adb shell dumpsys meminfo <包名>` | 查看应用内存使用详情 | 排查内存泄漏、内存占用过高问题 |
| `adb shell dumpsys cpuinfo` | 查看系统 CPU 使用率 | 全局 CPU 占用统计 |
| `adb shell dumpsys gfxinfo <包名>` | 查看应用渲染帧率、丢帧数据 | 分析界面卡顿、渲染性能问题 |
| `adb shell dumpsys battery` | 查看电池详细状态 | 包含电量、充电状态、温度、电压等 |
| `adb shell top` | 实时查看进程资源占用 | 高频参数：`-m 5` 仅显示占用最高的5个进程，`-d 1` 1秒刷新一次 |
| `adb shell ps -A` | 列出系统所有进程 | 过滤示例：`adb shell ps | grep com.example.app`，查看指定应用的进程 PID |
| `adb shell cat /proc/cpuinfo` | 查看设备 CPU 详细信息 | 核心数、架构、主频等 |
| `adb shell cat /proc/meminfo` | 查看设备内存硬件信息 | 总内存、可用内存等 |
| `adb shell screencap -p <设备保存路径>` | 屏幕截图 | 示例：`adb shell screencap -p /sdcard/screen.png`，执行后可通过 adb pull 拉取到本地 |
| `adb shell screenrecord <设备保存路径>` | 屏幕录屏 | 安卓 4.4+ 支持，默认最长录制 180 秒，高频参数：<br>`--time-limit 60` 设置录制时长60秒<br>`--bit-rate 5000000` 设置码率5Mbps<br>Ctrl+C 终止录制 |

## 四、ADB Shell 基础 Linux 命令
Android 基于 Linux 内核，adb shell 内支持大部分精简版 Linux 命令，无需 root 即可执行基础操作：
| 命令 | 核心作用 | 示例 |
| :--- | :--- | :--- |
| `ls` | 列出当前目录内容 | `ls -l` 显示详细权限与大小，`ls -a` 显示隐藏文件 |
| `cd <目录路径>` | 切换工作目录 | `cd /sdcard/Download/` 切换到下载目录 |
| `cat <文件路径>` | 查看文件内容 | `cat /sdcard/test.txt` |
| `rm <文件路径>` | 删除文件 | `rm -r <目录路径>` 删除文件夹及所有内容 |
| `mkdir <目录名>` | 创建文件夹 | `mkdir -p test/a/b` 递归创建多级目录 |
| `cp <源路径> <目标路径>` | 复制文件/文件夹 | `cp -r test/ /sdcard/` 递归复制文件夹 |
| `mv <源路径> <目标路径>` | 移动/重命名文件 | `mv old.txt new.txt` 重命名 |
| `chmod <权限> <文件路径>` | 修改文件权限 | 需 root 权限，示例：`chmod 755 test.sh` |
| `ping <IP/域名>` | 测试网络连通性 | `ping www.baidu.com` |
| `ifconfig` | 查看网络接口信息 | 查看设备 IP、MAC 地址等 |
| `df -h` | 查看磁盘分区使用情况 | 查看存储空间占用 |
| `du -sh <目录路径>` | 查看目录占用空间大小 | `du -sh /sdcard/` |

## 五、进阶用法
### 1. 应用备份与恢复
免 root 实现应用数据备份与恢复，部分系统应用/加密应用不支持：
- 全量备份：`adb backup -all -f full_backup.ab -apk`（-apk 包含安装包，不加仅备份数据）
- 单个应用备份：`adb backup -f app_backup.ab -apk com.example.app`
- 恢复备份：`adb restore backup.ab`

### 2. 刷机相关命令
- 卡刷 OTA 包：`adb sideload <OTA包路径>`（需设备进入 Recovery 模式的 sideload 功能）
- 关闭系统分区校验：`adb disable-verity`（需 root，安卓 8.0+，修改系统分区前执行）
- 开启系统分区校验：`adb enable-verity`

### 3. 自动化脚本
可将 ADB 命令写入脚本文件，实现批量操作：
- Windows：编写 `.bat` 批处理文件，示例：
```bat
@echo off
adb devices
adb logcat -c
adb logcat -v time > app_log.txt
pause
```
- macOS/Linux：编写 `.sh` Shell 脚本，添加执行权限后运行。

## 六、常见问题排查
### 1. adb devices 找不到设备/显示 unauthorized
1.  确认设备已开启开发者选项和 USB 调试，重新插拔 USB 线
2.  确认 USB 线为**数据线**（非仅充电线），更换 USB 口/线材
3.  设备端取消原有授权，重新勾选「始终允许」并确认
4.  Windows 需安装对应设备的官方 USB 驱动
5.  重启 ADB 服务：`adb kill-server && adb start-server`

### 2. 无线连接失败
1.  确认 PC 与设备在**同一局域网**，可互相 ping 通
2.  确认设备端口未被占用，防火墙未拦截 ADB 通信
3.  安卓 11+ 确认配对码正确，无线调试开关已开启，配对端口与连接端口为两个不同端口
4.  部分设备需关闭 VPN/代理，避免局域网通信被拦截

### 3. APK 安装失败
| 常见报错 | 解决方案 |
| :--- | :--- |
| 签名不一致 | 先卸载设备上的旧版本，再执行安装；或加 `-r` 参数覆盖安装 |
| 安装包解析失败 | 确认 APK 完整未损坏，适配设备 CPU 架构与系统版本 |
| 降级安装失败 | 加 `-d` 参数允许降级覆盖安装 |
| 权限被拒 | 开启设备「未知来源应用安装」权限，安卓 13+ 需开启「USB 安装应用」权限 |
| 存储空间不足 | 清理设备存储空间 |

### 4. 命令执行提示 Permission denied
1.  执行 `adb root` 获取 root 权限（仅 root 设备支持）
2.  系统分区读写失败，先执行 `adb disable-verity` 关闭校验，再执行 `adb root` + `adb remount`
3.  安卓 10+ 分区存储限制，避免直接访问 `/sdcard/Android/data` 等受限目录

### 5. 5037 端口被占用
1.  查看占用进程：Windows 执行 `netstat -ano | findstr 5037`，macOS/Linux 执行 `lsof -i:5037`
2.  结束占用进程，或重启 ADB 服务
3.  可通过环境变量 `ANDROID_ADB_SERVER_PORT` 自定义 ADB 服务端口


