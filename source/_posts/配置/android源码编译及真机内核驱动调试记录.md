---
title: android源码编译及真机内核驱动调试记录
date: 2021-09-14 10:57:17
tags: Androi,内核
categories: 配置
description: 编译安卓源码并调试真机内核。
---

# 安卓源码下载

安卓源码可以参照安卓官方的[说明](https://source.android.com/setup/build/downloading?hl=zh-cn)到google仓库下载，但是我挂梯子网络也不行，下不下来，就到了[tuna](https://mirrors.tuna.tsinghua.edu.cn/help/AOSP/)，按照上面的指引，同步了repo。

##### 获取版本号

下载之前第一件事是确定好自己要下载的android源码的build id和标记，这个主要由手机型号决定，代号、标记和Build号的对应关系到[这里](https://mirrors.tuna.tsinghua.edu.cn/git/AOSP/platform/manifest)查，我查到我要下载的Pixel 3的2020年左右的源码Build号是`QQ2A.200501.001.B2`，标记是`android-10.0.0_r35`。

##### 下载源码

按照tuna的指引，先下载它提供的repo并更新tuna镜像源:

````sh
curl https://mirrors.tuna.tsinghua.edu.cn/git/git-repo -o repo
chmod +x repo
export PATH=$PATH:/path/to/repo
export REPO_URL='https://mirrors.tuna.tsinghua.edu.cn/git/git-repo'
````

跳过这一步可能会出现网络问题。接下来下载自己需要版本的源码:

````sh
mkdir android-10.0.0_r35
cd android-10.0.0_r35
repo init -u https://mirrors.tuna.tsinghua.edu.cn/git/AOSP/platform/manifest -b android-10.0.0_r35
repo sync
````

这里需要很长时间来下载，总大小大概90+G。

##### 下载驱动二进制

只有源码的话，还缺少了一些硬件的驱动和库，这些东西不是开源的，需要单独下载，到[google](https://developers.google.cn/android/drivers)官网下载对应Build ID的两个binary，一个是google的，一个是qualcomm的。

# 安卓源码编译

主要参考[编译指示](https://source.android.com/source/building.html)进行编译。

##### 拷贝专有二进制文件

首先将之前下载的两个驱动二进制文件解压，解压后是两个脚本，把这两个脚本拷贝到源码根目录下，直接运行即可，它们会将对应的文件安装在`vendor/`目录下。

##### 编译

首先清理之前的编译输出:

````sh
make clobber
````

然后初始化环境:

```sh
source build/envsetup.sh
```

设置编译目标，使用`lunch`命令查看可选的目标，由于我的目标是代号`blueline`的pixel手机，选择了`blueline`的debug版本。

````sh
lunch aosp_blueline-eng
````

编译可选目标如下:

| Buildtype |                       用途                       |
| --------- | :----------------------------------------------: |
| user      |             有限的权限；适合一般用户             |
| userdebug | 类似user模式，但有root权限和debug能力，适合debug |
| eng       |         带有额外的debug工具的开发配置。          |

然后开始编译:

````sh
make -j4
````

我一开始开8个线程在i7-9700上编译了三个多小时，98%左右因为占用CPU过高被kill了，然后开4个线程继续编译，一直报错，是`PYTHONPATH`和`PYTHONHOME`相关的错误，尝试把它们unset了没用，于是又只`make clobber`清理了一遍，然后从头开4个线程编译，然后还是失败了好多次。后面装了个python2，并把`/usr/bin/python`链接到python2后，编译终于成功了。

##### 刷写系统

在源码目录下，`source build/env_setup.sh`后，依次执行:

````sh
adb reboot bootloader #进入bootloader
fastboot flashall -w #刷写
````

等待刷写成功就行了。

## 内核调试

一开始问一个学长怎么搞android kernel debug，他说要焊一个串口出来，当年他就是因为要焊串口才放弃了...后来他给我发了一个[blog](https://www.trendmicro.com/en_us/research/17/a/practical-android-debugging-via-kgdb.html)，大意是用tcp连接adb，然后关掉usb端口，用usb端口来给kgdb通信。我在关闭usb的那一步卡住:

````sh
echo 0 > /sys/class/android_usb/android0/enable
````

我这个版本的kernel并没有这个关闭usb的选项，power/level也只有auto和on两个状态...遂转为手动调试。

另外在用`cat /proc/kallsyms`打印内核符号地址时，发现打印出地址全是0，查了一下发现需要改`/proc/sys/kernel/kptr_restrict`变量，该变量有三个取值:

| 2    | 内核将符号地址打印为全0, root和普通用户都没有权限 |
| ---- | ------------------------------------------------- |
| 1    | root用户有权限读取, 普通用户没有权限              |
| 0    | root和普通用户都可以读取                          |

`echo 0 > /proc/sys/kernel/kptr_restrict`即可。
