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

### 安装后解决问题: `/usr/local/cuda/lib64/libcudnn*.so* is not a static symbol`

安装后使用`ldconfig`的时候可能会有错误提示:

````
/sbin/ldconfig.real: /usr/local/cuda-11.2/targets/x86_64-linux/lib/libcudnn_cnn_infer.so.8 is not a symbolic link
/sbin/ldconfig.real: /usr/local/cuda-11.2/targets/x86_64-linux/lib/libcudnn.so.8 is not a symbolic link
/sbin/ldconfig.real: /usr/local/cuda-11.2/targets/x86_64-linux/lib/libcudnn_adv_infer.so.8 is not a symbolic link
/sbin/ldconfig.real: /usr/local/cuda-11.2/targets/x86_64-linux/lib/libcudnn_ops_infer.so.8 is not a symbolic link
/sbin/ldconfig.real: /usr/local/cuda-11.2/targets/x86_64-linux/lib/libcudnn_cnn_train.so.8 is not a symbolic link
/sbin/ldconfig.real: /usr/local/cuda-11.2/targets/x86_64-linux/lib/libcudnn_adv_train.so.8 is not a symbolic link
/sbin/ldconfig.real: /usr/local/cuda-11.2/targets/x86_64-linux/lib/libcudnn_ops_train.so.8 is not a symbolic link
````

可能是拷贝后静态链接被损坏导致的，需要一个一个解决，以`libcudnn.so`为例:

````sh
sudo rm libcudnn.so.8 libcudnn.so
sudo ln libcudnn.so.8.2.0 libcudnn.so.8
sudo ln libcudnn.so.8 libcudnn.so
````

一个一个解决就好了。



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

```python
import torch
print(torch.__version__)
print(torch.version.cuda)
```

实际上我在`import torch`时报错:

````
>>> import  torch
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/home/varas/shijy/install_torch/pytorch/torch/__init__.py", line 214, in <module>
    raise ImportError(textwrap.dedent('''
ImportError: Failed to load PyTorch C extensions:
    It appears that PyTorch has loaded the `torch/_C` folder
    of the PyTorch repository rather than the C extensions which
    are expected in the `torch._C` namespace. This can occur when
    using the `install` workflow. e.g.
        $ python setup.py install && python -c "import torch"

    This error can generally be solved using the `develop` workflow
        $ python setup.py develop && python -c "import torch"  # This should succeed
    or by running Python from a different directory.
````

按照提示执行`python setup.py develop`就好了。



## 参考链接

[安装NVIDIA显卡驱动和CUDA Toolkit](https://www.jianshu.com/p/ba6beab8ad7f)

[Install CUDA 11.2, cuDNN 8.1.0, PyTorch v1.8.0 (or v1.9.0), and python 3.9 on RTX3090 for deep learning](https://medium.com/analytics-vidhya/install-cuda-11-2-cudnn-8-1-0-and-python-3-9-on-rtx3090-for-deep-learning-fcf96c95f7a1)