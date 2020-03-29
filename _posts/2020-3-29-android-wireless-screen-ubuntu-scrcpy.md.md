# 安卓Ubuntu 18.04 lts投屏scrcpy方法

scrcpy是screen copy的简写，是一个免费的开源软件，通过命令行和快捷键执行，实现安卓设备向电脑的高清投屏。我个人体验，感觉操作方便简洁，相当nice

1. scrcpy usb 有线投屏，首先启动adb服务

安卓设备：

1. usb线连接
2. 设置：usb偏好为“文件传输”（从正常使用scrcpy的角度来看，也可以不设置。但是设置完之后，手机和电脑的文件互传就会变得相当方便（用文件管理器剪切粘贴即可），我每次都会用～）
3. 设置：开启开发者模式，开启usb调试

电脑：

1. 启动adb服务

安卓设备那里没什么好说的，操作很简单；所以这里只谈“开启adb服务”。

“开启adb服务”包括第一次安装并配置并开启、第一次以后的直接开启。

**无论是第一次，还是第一次以后，都首先，把安卓设备那三步操作完成。**

##### 第一次，安装并配置adb服务

```bash
sudo apt-get install android-tools-adb
adb start-server
lsusb
```

输出以下信息

```bash
Bus 002 Device 002: ID 8087:0024 Intel Corp. Integrated Rate Matching Hub
Bus 002 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Bus 001 Device 006: ID 04f2:b2ea Chicony Electronics Co., Ltd Integrated Camera [ThinkPad]
Bus 001 Device 009: ID 0a5c:21e6 Broadcom Corp. BCM20702 Bluetooth 4.0 [ThinkPad]
Bus 001 Device 004: ID 147e:2020 Upek TouchChip Fingerprint Coprocessor (WBF advanced mode)
Bus 001 Device 012: ID 1c4f:0065 SiGma Micro 
Bus 001 Device 002: ID 8087:0024 Intel Corp. Integrated Rate Matching Hub
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Bus 004 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
Bus 003 Device 025: ID 2717:ff48  
Bus 003 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub

```

找到自己的安卓设备哪一行，2717:ff48  在下面会用到

创建设备文件：

```bash
echo 0x12d1 > ~/.android/adb_usb.ini
touch /etc/udev/rules.d/90-android.rules
gedit /etc/udev/rules.d/90-android.rules
```

将以下内容写入刚刚创建的文件，注意，下面的2717:ff48 要改成自己的安卓设备的id（见上）：

```bash
SUBSYSTEM"usb", ATTRS{idVendor}"2717", ATTRS{idProduct}=="ff48", MODE="0666"
```

更改文件权限

```bash
chmod 666 /etc/udev/rules.d/90-android.rules
```

重启adb服务

```bash
service udev restart
adb kill-server
adb start-server
```

执行以下命令，如有设备，则adb配置成功了

```bash
adb devices
```

第一次以后，开启adb服务

```
adb devices
adb start-server
adb devices
```

2.安装scrcpy    github仓库[https://github.com/Genymobile/scrcpy]:

for linux

```
apt install scrcpy
```

for macOS

```
brew install scrcpy
```

3.开启scrcpy服务

首先插入安卓设备

```
scrcpy
```

帮助信息

```bash
scrcpy --help
```

3.scrcpy无线投屏方式

- 将手机和电脑连接到同一个wifi

- 查看手机IP地址(设置-关于手机-设备状态)

- 开启adb TCP/IP 

- ```bash
  adb tcpip 5555
  ```

- 拔掉手机

- 无线连接手机

- ```bash
  adb connect DEVICE_IP:5555
  ```

- 重新运行scrcpy

- ```bash
  scrcpy
  ```

全屏投放

```bash
scrcpy --fullcreen
scrcpy -f #short version
```

息屏投放

```bash
scrcpy --turn-screen-off
scrcpy -S
```

