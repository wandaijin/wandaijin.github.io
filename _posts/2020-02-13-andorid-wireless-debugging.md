---
layout: post
title:  "Android Wireless Debugging"
categories: android bootloader
---

## 使用WLAN连接设备进行调试

1. 保证设备和电脑处于同一个局域网中，并且可以相互联通(可能需要)
2. 使用 USB 数据线将设备连接到主机。
3. 设置目标设备以监听端口 5555 上的 TCP/IP 连接， `adb tcpip 5555`。然后可以断开USB连接。
4. 找到 Android 设备的 IP 地址。如下两种方法
  - `adb shell ifconfig | grep "inet " | grep -v 127.0.0.1 | awk '{print $2}'`
  - 在设备中的`设置->WLAN`网络设置中查看
5. 通过 IP 地址连接到设备。
  `adb connect device_ip_address:5555`
6. 确认主机已连接到目标设备。
  `adb devices`


### 在mac/linux上可以使用，简化命令输入

```
adb tcpip 5555
adb connect $(adb shell ifconfig | grep "inet " | grep -v 127.0.0.1 | awk '{print $2}' | cut -d: -f2):5555
```

### 查看连接的设备，并发送命令到指定设备(serial_number是`adb devices`命令的第一列)

```
adb devices [-l]
adb [-d | -e | -s serial_number] command
```

注意：
如果adb连接断开，可以通过`adb connect device_ip_address:5555`重新连接。
如果上述操作未解决问题，重置adb主机`adb kill-server`, 然后重新开始

拓展知识：
客户端：用于发送命令。客户端在开发计算机上运行。您可以通过发出 adb 命令从命令行终端调用客户端。
守护进程 (adbd)：在设备上运行命令。守护进程在每个设备上作为后台进程运行。
服务器：管理客户端和守护进程之间的通信。服务器在开发计算机上作为后台进程运行。


### 参考资料
1. https://developer.android.com/tools/help/adb.html#wireless
2. https://developer.android.google.cn/tools/help/adb.html#wireless
