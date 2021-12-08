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

| Buildtype |                       用途                       | ADB选项                          |
| --------- | :----------------------------------------------: | -------------------------------- |
| user      |             有限的权限；适合一般用户             | 需要手动在开发者选项打开调试模式 |
| userdebug | 类似user模式，但有root权限和debug能力，适合debug | 默认开启，但没有root             |
| eng       |         带有额外的debug工具的开发配置。          | 默认开启且root                   |

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

# 刷内核

要在内核中开启KASAN等工具的话，需要自己编译一遍内核，Android源码中构建中不包含内核的构建。主要参考[官方文档](https://source.android.com/devices/tech/debug/kasan-kcov)和[这个博客](https://blog.kyrios.cn/2021-07-android-11-building-on-pixel-3/)。

首先下载内核源码，pixel 3的话是`crossbatch`内核

````c
mkdir kernel && cd kernel
repo init -u https://android.googlesource.com/kernel/manifest -b android-msm-crosshatch-4.9-android10
repo sync
````

如果想刷某个历史版本，可以通过repo回退到某个时间之前：

````sh
repo forall -c 'commitID=`git log --before "2020-06-01" -1 --pretty=format:"%H"`; git reset --hard $commitID'
````

直接编译默认的内核:

````sh
sudo apt-get install liblz4-tool
export ARCH=arm64
./build/build.sh
````

默认编译使用的是`private/msm-google/arch/arm64/configs/b1c1_defconfig`，要对其修改，先在该目录下创建一个新文件：

````sh
cd private/msm-google/arch/arm64/configs/b1c1_defconfig
cp b1c1_defconfig b1c1-kasn_defconfig
````

添加如下选项来启用KASAN:

````makefile
CONFIG_KASAN=y
CONFIG_KASAN_INLINE=y
CONFIG_KCOV=y
CONFIG_SLUB=y
CONFIG_SLUB_DEBUG=y
CONFIG_FRAME_WARN=n #避免因为栈大小过大而报错
    
# CONFIG_KERNEL_LZ4=y # 注释掉这行
````

修改kernel根目录下的`build.config`:

````sh
DEFCONFIG=b1c1-kasn_defconfig
KERNEL_DIR=private/msm-google
. ${ROOT_DIR}/${KERNEL_DIR}/build.config.common.clang

 POST_DEFCONFIG_CMDS="update_nocfi_config"
# POST_DEFCONFIG_CMDS="check_defconfig update_nocfi_config" # 有可能因为添加了check_defconfig，视情况注释掉
````

编译时会因为加了kasan导致很多栈桢超长，还是会有些地方有warning，由于开启了WError，会当成error处理，最后会编译失败，这时需要修改`b1c1-kasn_defconfig`的`CONFIG_WERROR`：

````makefile
CONFIG_WERROR=n
````

改了还是有问题，再改`private/msm-google/Makefile`，注释掉这几行：

````makefile
# ifdef CONFIG_CC_WERROR
# KBUILD_CFLAGS>+= -Werror
# endif
````

编译时还是会报错，主要是在编译外部模块时有问题，发现报错是在`msm-google-modules`里面，找到有`-Werror`的地方，发现是`private/msm-google-modules/wlan/qcacld-3.0/Kbuild`里面有一个`-Werror`，注释掉就好了。

接下来的编译应该没问题了。编译完成后，输出的`Image.lz4`在`out/android-msm-pixel-4.9/dist`目录下，将其拷贝到aosp的`device/google/crosshatch-kernel/Image.lz4`，然后重新编译aosp:

````sh
make -j4
````

编译完成后，重新刷系统：

````sh
adb reboot bootloader
fastboot flashall -w
````

好了，刷完以后触屏失灵了，adb也连不上了，因为adb授权要在触屏点确认，所以也没办法授权adb，查了下，原因是禁止加载签名未知的模块，要解决这个问题由两个办法：

+ 允许未授权的adb访问 [ref](https://www.i4k.xyz/article/cc0410/96479443)
+ 把触控板模块直接编译到内核镜像里面 [ref](https://android.stackexchange.com/questions/239254/aosp-android-11-kernel-build-for-pixel3a-sargo-touchscreen-not-working)

双管齐下，两个一起弄，首先修改aosp中的`build/make/core/main.mk`开启允许未授权的adb访问：

````makefile
# 第1处
ifeq($(user_variant),user)
    ADDITIONAL_DEFAULT_PROPERTIES += ro.adb.secure=1        ===>改成0
endif
# 第2处
ifeq (true,$(strip $(enable_target_debugging)))
    xxxxxxxxxxxxxx
else
    ADDITIONAL_DEFAULT_PROPERTIES += ro.debuggable=0        ===>改成1
endif
````

把触控板模块编译到内核镜像需要修改kernel代码中的`private/msm-google/arch/arm64/configs/b1c1-kasan_defconfig`，将其中的带`TOUCHSCEEN`的选项改为`=y`:

````makefile
CONFIG_INPUT_TOUCHSCREEN=y
CONFIG_TOUCHSCREEN_FTS=y
CONFIG_TOUCHSCREEN_FTM4=y
CONFIG_TOUCHSCREEN_SEC_TS=y
CONFIG_TOUCHSCREEN_TBN=y
````

重新编译后，把它放到aosp中重新编译aosp，生成新的`boot.img`后刷进手机。

好了，现在不能用adb，也不能用触屏授权，怎么进fastboot模式呢？查了一下，方法如下：

+ 按住`电源键+上音量键+下音量键`大概七八秒，重启手机
+ 重启时进入google logo界面后立即按住`电源键+下音量`，然后就进入了fastboot模式

接下来刷boot.img:

````sh
fastboot flash boot boot.img
fastboot reboot
````

还是不行，后面在[这里](https://www.akr-developers.com/d/526-pixel3-aosp/3)找到一句话：

> 对于一般的机子，以上操作足以使得内核工作正常，但是Pixel3比较别致，如果这些驱动伴随着内核一起启动，会由于用户空间还没准备好而导致它们失效...

按照这个链接里给的方法，都搞不出来...

在回宿舍的班车上想了下这个问题，实际上很好绕过这个问题：

+ 首先编译一套安卓原装的系统和内核进去，这时各种驱动都可以正常运行，这时用adb连接手机，进行授权。

+ 然后刷入自定义的内核进去，这时各种驱动都会失效，但是这时候adb授权信息还在系统里，所以可以通过adb连接。

+ 手动安装`ko`，使触摸屏、wlan等硬件正常工作：

  ````shell
  # 重新加载安卓文件系统为可写
  adb remount
  # 将内核目录下生成的`.ko`拷贝到手机中：
  adb push out/android-msm-pixel-4.9/dist/*.ko /vendor/lib/modules
  # 到adb shell里安装各个ko
  adb shell
  modprobe -d /vendor/lib/modules *.ko
  ````

这个操作在自编译内核时可以，但在开启KASAN再自编译后就不行了，`modprobe`时会提示:

````
insmod: failed to load wlan.ko: Exec format error
````

dmesg里报错：`disagrees about version of symbol module_layout`。这个原因说是内核模块和内核编译的版本不一样，没设置好，试了[这里](https://www.akr-developers.com/d/526-pixel3-aosp/3)说的三个提交都不行：

> https://github.com/luk1337/android_kernel_oneplus_sm8250/commit/153318ee16f44c26a8a3d2da6fecdb9f8c266c18
>
> https://github.com/luk1337/android_kernel_oneplus_sm8250/commit/eaed1807d37a3201e1fcf31b3d148f4d3f39e27d
>
> https://github.com/luk1337/android_kernel_oneplus_sm8250/commit/2ac459f2b93908c5418c7afa7643f290107196d9



