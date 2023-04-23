---
title: "Ubuntu18安装GPU,CUDA,NVIDIA Docker"
datePublished: Sun Apr 23 2023 07:14:02 GMT+0000 (Coordinated Universal Time)
cuid: clgt2pqaj000109md8h8zfhll
slug: ubuntu18gpucudanvidia-docker
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1682234992612/97a38336-882b-4641-a420-380b2d72d727.jpeg
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1682234030723/85c004d9-8f72-411e-9019-bbe5d30e74ce.jpeg
tags: docker, nvidia

---

# 安装nvidia驱动,使用命令行安装NVIDIA驱动程序

如果不知道安装那个驱动比较好，推荐是最适合的选择。以下输出是教程计算机的结果，您可能会看到不同的输出，具体取决于您的系统。

```bash
ubuntu-drivers devices
```

输出如下

```bash
== /sys/devices/pci0000:00/0000:00:01.0/0000:01:00.0 ==
modalias : pci:v000010DEd00001F95sv00001028sd0000097Dbc03sc02i00
vendor   : NVIDIA Corporation
model    : TU117M [GeForce GTX 1650 Ti Mobile]
driver   : nvidia-driver-440 - distro non-free recommended
driver   : xserver-xorg-video-nouveau - distro free builtin
```

通常，最好安装推荐的驱动程序，当你决定使用驱动程序版本后请使用 [`apt 软件包管理器`](https://www.myfreax.com/how-to-use-apt-command/)安装英伟达显卡驱动 `nvidia-driver-440`。

安装完成后，然后在终端运行命令 `sudo reboot` 重启系统。当返回到系统时，您可以在终端再次运行命令 `nvidia-smi` 启动监视工具查看图形卡/显卡的状态。

`nvidia-smi` 命令将显示所用驱动程序的版本以及有关NVIDIA显卡的其它信息

```bash
sudo apt install nvidia-driver-440
sudo reboot
nvidia-smi
```

我们推荐大多数用户使用 Ubuntu 软件源默认提供的 NVIDIA 驱动程序。如果您想使用最新的 NVIDIA 驱动程序。

打开终端运行 `add-apt-repository` 命令添加 micahflee/ppa 软件源到你的 Ubuntu 系统。

```bash
sudo add-apt-repository ppa:micahflee/ppa
sudo apt update

ubuntu-drivers devices
```

# 二.安装对应的CUDA版本

使用`nvidia-smi`输出如下内容

```bash
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 470.42.01    Driver Version: 470.42.01    CUDA Version: 11.4     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|                               |                      |               MIG M. |
|===============================+======================+======================|
|   0  Tesla T4            On   | 00000000:3B:00.0 Off |                    0 |
| N/A   32C    P8     9W /  70W |      4MiB / 15109MiB |      0%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+

+-----------------------------------------------------------------------------+
| Processes:                                                                  |
|  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
|        ID   ID                                                   Usage      |
|=============================================================================|
+-----------------------------------------------------------------------------+
```

可以看到此驱动推荐的对应cuda版本是 11.4

[CUDA Toolkit Archive](https://developer.nvidia.com/cuda-toolkit-archive)到这个网址选择对应的系统版本，复制命令一键完成。

# 三.安装nvidia-docker runtime

[Migration Notice](https://nvidia.github.io/nvidia-container-runtime/)到这个网址查看支持的操作系统和版本。

进行测试，如果出现显卡信息就可以了，下面命令是在容器内进行

```SQL
docker run -it --rm --gpus all centos nvidia-smi
```

```SQL
nvidia-smi --list-gpus
```

列出所有GPU