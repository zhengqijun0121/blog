# Android 安装 APK


# Android 安装 APK

## adb 命令安装

- 查看当前设备已安装包名

```bash
adb shell pm list packages
```

- 安装 `APK`

```bash
adb install -r app-debug.apk
```

- 卸载 `APK`

```bash
adb uninstall com.test.demo
```

## pm 命令安装

先用 `adb` 命令将安装包推到机器中

- 查看当前设备已安装包名

```bash
pm list packages
```

- 安装 `APK`

```bash
pm install -r app-debug.apk
```

- 卸载 `APK`

```bash
pm uninstall com.test.demo
```


