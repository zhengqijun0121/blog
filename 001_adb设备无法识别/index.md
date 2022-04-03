# adb 设备无法识别


## 设备没有权限

执行 `adb devices` 命令输出

```bash
List of devices attached
3ed62e70        no permissions
```

## 查看 USB 设备

执行 `lsusb` 命令输出

```bash
Bus 002 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
Bus 001 Device 094: ID 17ef:6050 Lenovo 
Bus 001 Device 124: ID 05c6:901d Qualcomm, Inc. -> 新增 usb 设备
Bus 001 Device 002: ID 060b:7a03 Solid Year 
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
```

## 配置 USB 设备权限

修改 `/etc/udev/rules.d/android.rules` 文件内容

```bash
SUBSYSTEM=="usb", ATTRS{idVendor}=="05c6", ATTRS{idProduct}=="901d", MODE="0666"
```

## 使配置生效

执行 `sudo service udev restart` 命令


