---
title: Android刷系统+Root记录
date: 2021-09-13 12:23:45
tags: Android
categories: 配置
description: 一次刷android系统和root的记录。
---

环境:

+ ubuntu 20.04
+ google pixel 3手机

## 刷写系统

到[google android images](https://developers.google.com/android/images)下载目标系统的factory image，并解压缩，到[https://developer.android.com/studio/releases/platform-tools.html](platform tools)下载adb/fastboot等工具。

把刚才下载的`platform-tools`添加到环境变量里。

adb连接设备后:

```sh
adb reboot bootloader #重启至Bootloader界面
fastboot flashing unlock #解锁Bootloader
```

到解压的factory image目录下，执行`flash-all.sh`。等待系统刷写完成即可。

## root

解压factory image目录下的一个zip文件，获得`boot.img`，并通过`adb push boot.img /sdcard`命令存放到手机的sd卡内。下载[Magisk](https://github.com/topjohnwu/Magisk/releases)的安装包到手机并安装，而后开始root:

+ magisk首页选择`安装->选择并修补一个文件`，选择刚才拷贝过来的`boot.img`，等待patch完成。

+ 把生成的文件通过`adb pull xxx.img`命令拷贝到电脑。

+ 将新的img刷写到手机:

  + ````sh
    adb reboot bootloader
    fastboot flash boot xxx.img
    ````

等待刷写完成后，直接运行`adb root`仍然拿不到root shell，需要进入普通adb shell后，运行`su`命令切换到root命令行。
