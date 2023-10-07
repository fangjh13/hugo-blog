---
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowReadingTime: true
ShowRssButtonInSectionTermList: true
ShowWordCount: true
TocOpen: false
UseHugoToc: true
date: "2018-04-02T12:41:09+08:00"
lastmod: "2018-04-02T12:51:42+08:00"
showToc: true
tags: [Raspberry Pi]
title: 使用usb转TTL(PL2303模块) 串口连接树莓派3B
---

### 使用 usb 转 TTL(PL2303 模块) 串口连接树莓派 3B

> By default, on Raspberry Pis equipped with the wireless/Bluetooth module (Raspberry Pi 3 and Raspberry Pi Zero W), the PL011 UART is connected to the BT module, while the mini UART is used for Linux console output. On all other models the PL011 is used for the Linux console output.

在树莓派 3 以前，官方是将“硬件串口”分配给 GPIO 中的 UART(GPIO14&GPIO15)，因此可以独立调整串口的速率和模式，直接连接就可以。而在 3 以后的树莓派中因为新增了蓝牙使用掉了 UART。这样默认就不开启了，以下是手动开启的方法。

使用 HDMI 启动树莓派编辑`/boot/config.txt`在后面新增以下两行

```shell
# 激活串口输出
enable_uart=1
# 禁用蓝牙
dtoverlay=pi3-disable-bt
```

运行`sudo systemctl disable hciuart`禁用蓝牙服务，然后重启

运行`ls -l /dev/serial*`查看是否是这个样子

```shell
pi@raspberrypi:~$ ls -l /dev/serial*
lrwxrwxrwx 1 root root 7 Mar 13 23:23 /dev/serial0 -> ttyAMA0
lrwxrwxrwx 1 root root 5 Mar 13 23:23 /dev/serial1 -> ttyS0
```

以上就是树莓派上所有的设置了，mac 和 windows 如何使用 PL2303 模块连接请参考以下链接

Mac

[https://pbxbook.com/other/mac-tty.html](https://pbxbook.com/other/mac-tty.html)

Windows

[https://blog.csdn.net/cugbabybear/article/details/23048741](https://blog.csdn.net/cugbabybear/article/details/23048741)

#### Reference

- [https://www.raspberrypi.org](https://www.raspberrypi.org/documentation/configuration/uart.md)
