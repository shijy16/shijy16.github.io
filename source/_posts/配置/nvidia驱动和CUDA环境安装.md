---
title: nvidia驱动和CUDA环境安装
date: 2021-05-13 12:15:16
tags: GPU
categories: 配置
description: 最近在新机器上配置GPU驱动和AI环境，这里做一个记录，方便日后维护和配置其他机器。
---

## 卸载所有nvidia相关文件

一开始在安装了驱动后，安装CUDA的时候一直提示检测到驱动已安装，是否还要继续。搞得我很懵逼。又强行把所有nvidia驱动卸载了，不过正常安装前都需要把之前的所有nvidia相关软件卸了。

```
sudo apt --purge remove *nvidia*
sudo apt autoremove
```

## 禁用Nouveau kernel driver

`Nouveau kernel driver`是一个开放源码显卡驱动程序，linux发行版自带，一般作为桌面程序默认的显卡驱动，在安装N卡驱动前 或后需要将该驱动屏蔽，强制系统使用新安装的N卡程序。

创建文件`/etc/modprobe.d/blacklist-nouveau.conf`，添加如下内容:

```
blacklist nouveau
blacklist lbm-nouveau
options nouveau modeset=0
alias nouveau off
alias lbm-nouveau off
```

然后执行`sudo update-initramfs -u`。

如果`xserver`在运行中，也需要先关闭`xserver`：

```
service lightdm stop
```

## 直接安装驱动和CUDA

> 我就是装了驱动后再装CUDA，然后装CUDA的时候一进去就提示检测到驱动，我以为是冲突了，就把所有驱动卸了，实际上应该不管这个提示，继续安装CUDA就好。

驱动不需要手动安装，因为我在手动安装时提示可以自动安装，且自动安装的版本会更适合机器，所以直接自动安装驱动:

```
sudo ubuntu-drivers autoinstall
```

然后重启系统:

```
sudo reboot
```

确认安装正常:

```
nvidia-smi
```

显示支持`11.2`版本的CUDA，到[CUDA下载页](https://developer.nvidia.com/cuda-toolkit-archive)找对应版本的CUDA，然后下载CUDA的`.run`安装文件:

```
wget https://developer.download.nvidia.com/compute/cuda/11.2.2/local_installers/cuda_11.2.2_460.32.03_linux.run
sudo sh cuda_11.2.2_460.32.03_linux.run
```

安装时首先会提示检测到安装了驱动，选择继续安装，在第二页取消勾选driver。继续安装即可。

安装完成后，在`~/.bashrc`中添加环境变量:

```
export LD_LIBRARY_PATH="/usr/local/cuda/lib64":$LD_LIBRARY_PATH
export PATH="/usr/local/cuda/bin":$PATH
```

使用`nvcc --version`查看是否安装成功。

## 安装Cudnn

进入[cudnn](https://developer.nvidia.com/cudnn)下载页，注册开发者账号填问卷后，下载压缩包，然后解压缩，进行如下拷贝和权限设置即可:

```
sudo cp cuda/include/cudnn.h /usr/local/cuda/include
sudo cp cuda/lib64/libcudnn* /usr/local/cuda/lib64
sudo chmod a+r /usr/local/cuda/include/*.h 
sudo chmod a+r /usr/local/cuda/lib64/libcudnn*
```

## 在cuda11.2下安装torch

官方稳定版torch还没有支持11.2，需要手动编译源码安装。用Anaconda创建一个python3.9环境，然后安装依赖:

```
conda install numpy ninja pyyaml mkl mkl-include setuptools cmake cffi typing_extensions future six requests dataclasses
```

获取pytorch并安装:

```
git clone --recursive https://github.com/pytorch/pytorch
cd pytorch
# if you are updating an existing checkout
git submodule sync
git submodule update --init --recursive
export CMAKE_PREFIX_PATH=${CONDA_PREFIX:-"$(dirname $(which conda))/../"}
python setup.py install
```

之后python命令行中验证安装:

```
import torch
print(torch.__version__)
print(torch.version.cuda)
```

安装完成。

## 参考链接

[安装NVIDIA显卡驱动和CUDA Toolkit](https://www.jianshu.com/p/ba6beab8ad7f)

[Install CUDA 11.2, cuDNN 8.1.0, PyTorch v1.8.0 (or v1.9.0), and python 3.9 on RTX3090 for deep learning](https://medium.com/analytics-vidhya/install-cuda-11-2-cudnn-8-1-0-and-python-3-9-on-rtx3090-for-deep-learning-fcf96c95f7a1)