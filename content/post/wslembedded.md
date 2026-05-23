---
title: "WSL2+VSCode的STM32开发指南"
date: 2026-05-22T12:00:00+08:00
lastmod: 2026-05-22T12:00:00+08:00
draft: false
categories:
  - Tech
tags:
  - WSL2
  - archlinux
  - STM32
---

> 这是一篇关于使用 WSL2、 VS Code 进行 STM32 开发的教程。

## WSL2 和相应工具

### WSL2

WSL（Windows Subsystem for Linux）是微软开发的一项技术，允许用户在Windows系统中直接运行完整的Linux环境，无需虚拟机。WSL2是WSL的升级版本，提供了更高的性能和完全的系统调用兼容性。

关于 WSL2 下载和相应发行版的选择网上已有很多教程本文不再赘述。

> 本文使用的发行版为 archlinux  
> [微软官方WSL文档](https://learn.microsoft.com/zh-cn/windows/wsl/)

### usbipd-win

usbipd-win 是 USB/IP 开源项目，它能将 USB 设备连接到 WSL2 上运行的 Linux 分发版。在 Windows 计算机上配置 USB/IP 项目可以实现常见的开发者 USB 场景，例如刷写 Arduino 或访问智能卡读卡器和本文所用到的连接 ST-link烧录 STM32。

> [微软官方下载和使用教程](https://learn.microsoft.com/zh-cn/windows/wsl/connect-usb)

### VS Code

Visual Studio Code（简称VS Code）是微软公司发布现代化、轻量级、跨平台的源代码编辑器，内置多种插件，提供了丰富的功能，支持多种编程语言。本教程将用 VSCode 作为代码编辑器和 STM32 的开发环境。

> [VScode下载链接](https://code.visualstudio.com/)

## 开发环境搭建

### archlinux 工具安装

> 默认已经下载 archlinux 基础工具

```shell
# 软件包更新
sudo pacman -Syu

# 下载java，推荐最低版本是java21
sudo pacman -S jdk-openjdk

# 下载usb查看工具和解压工具
sudo pacman -S usbutils unzip
```

### WSL2 中下载 STM32CubeMX

STM32CubeMX（包括STM32CubeMX和STM32CubeMX2）是一款图形化工具，可简化STM32产品的配置，并通过分步引导的过程生成相应的初始化代码。

[登录STM32CubeMX下载界面](https://www.st.com.cn/zh/development-tools/stm32cubemx.html)，并登录账号，在底下获取软件处点击 STM32CubeMX （STM32CubeMX2 目前只支持 C5 系列芯片）的 Windows 下拉，选择 Generic Linux，点击下载。

![STM32CubeMX的Linux版本获取](https://img.fishy.us.ci/Embedded/1.png)

打开文件资源管理器，将下载后的文件放到剪切到 Linux 目录下合适地方。我创建 `~/EmbeddedTool/temp` 存储下载时的临时文件，`~/EmbeddedTool/STM32CubeMX` 存储软件。将解压包剪切至 `temp` 文件夹。

![文件资源管理器](https://img.fishy.us.ci/Embedded/2.png)

```shell
# 之后进入文件目录解压
cd EmbeddedTool/temp
unzip stm32cubemx-lin-v6-17-0.zip

# 运行安装引导程序
./STM32CubeMX*
```

在运行安装程序过程中，会显示图形化界面报错，把报错信息复制给 ai 后，安装对应的包就可以。成功打开安装引导界面之后，正常安装就可以（把程序安装在之前创建的目录下方便管理）。

```shell
# 进入程序目录
cd ~/EmbeddedTool/STM32CubeMX

# 运行程序
./STM32CubeMX
```

![程序成功运行界面](https://img.fishy.us.ci/Embedded/3.png)

为了方便启动添加别名到 `shell` 配置文件中，我使用的是 `fish`，添加配置：
```shell
nano ~/.config/fish/config.fish

# 在文件末尾添加
alias STM32CubeMX="/home/fish/EmbeddedTool/STM32CubeMX/STM32CubeMX"
```

![shell配置文件](https://img.fishy.us.ci/Embedded/4.png)

之后在任意目录下输入 `STM32CubeMX` 就能打开程序。

### VS Code 开发 STM32 配置

[查看b站官方视频](https://www.bilibili.com/video/BV1JJ2cBHE2f?spm_id_from=333.788.videopod.sections&vd_source=8a4ff9cc9cd1b483b15aff6c6875315a)可以很好了解 VS Code 中 `STM32CubeIDE for Visual Studio Code` 插件的使用，不用再额外配置环境。之后下载 `WSL` 插件，使 archlinux 连接到 VS Code。

![WSL插件](https://img.fishy.us.ci/Embedded/5.png)

![连接到arch](https://img.fishy.us.ci/Embedded/6.png)

在 VS Code 左下角可以看到 VS Code 连接成功标志。

![连接到arch](https://img.fishy.us.ci/Embedded/7.png)

或者在 WSL 工作目录下输入 `code .` 也可以连接到 VS Code。

在 WSL 远程连接中再下载一份 STM32 的插件就可以正常使用了。

## 程序的烧录

### 连接USB

跟着教程使用使用 [usbipd](https://learn.microsoft.com/zh-cn/windows/wsl/connect-usb) 可以将 ST-Link 挂载到 WSL 中。在 WSL 终端中使用 `lsusb` 命令可以查看挂载USB设备。

![挂载成功的ST-Link](https://img.fishy.us.ci/Embedded/8.png)

但每次手动挂载都很麻烦，下载一个 VS Code 插件 `USBIP Connect` 来实现快速挂载。

![usb插件](https://img.fishy.us.ci/Embedded/9.png)

下载成功后点击 VS Code 下方状态栏的 `attack` 任务按键便可以实现快速挂载。

![usb挂载](https://img.fishy.us.ci/Embedded/10.png)

做完上述的一切之后，正准备调试时可能会发现在 `Debug` 界面上怎么都连接不上 ST-Link。可能是因为用户权限不足和 `udev`（设备管理器） 没有 ST-Link 的预设规则。从上面 `lsusb` 命令可以看到，ST-Link 挂在 `/001/002` 上。

```shell
# 查看实际设备文件
sudo ls -l /dev/bus/usb/001

#  输出
crw-rw---- 1 root root 189, 2 May 22 23:05 002
```

可以看到只有 `root` 可读写，可以使用命令 `sudo chmod 666 /dev/bus/usb/001/002` 来临时获得权限，此时可以发现连接到了一个 USB 设备，可以进行烧录了。因为 Linux `/dev/bus/usb/` 下的文件是设备插入瞬间动态内存创建的虚拟节点，所以每次重启 WSL 或 插拔 ST-Link 都会失效。

WSL2 现在已经默认支持 `systemd` 服务，我们可以自己创建 `udev` 的 ST-Link 预设文件。

```shell
# 创建并编辑规则文件
sudo nano /etc/udev/rules.d/99-stlink.rules

# 输入规则
# ST-Link V2
SUBSYSTEM=="usb", ATTR{idVendor}=="0483", ATTR{idProduct}=>
# ST-Link V2.1 (Nucleo)
SUBSYSTEM=="usb", ATTR{idVendor}=="0483", ATTR{idProduct}=>
# ST-Link V3
SUBSYSTEM=="usb", ATTR{idVendor}=="0483", ATTR{idProduct}=>
SUBSYSTEM=="usb", ATTR{idVendor}=="0483", ATTR{idProduct}=>
```

创建成功后再次使用 `USBIP Connect` 连接 ST-Link 可以看到，VS Code 已经能正确识别 ST-Link 了，而不是之前那种 普通 USB 的形式。

![成功识别设备](https://img.fishy.us.ci/Embedded/11.png)

## 总结

因为 WSL2 使用 WSLg 并支持 `systemd`，因此成功搭建了一套从 WSL2 启动 CubeMX 进行初始化，并从 WSL 远程连接烧录的 STM32 的 Linux 开发环境。

> 小白第一次写博客，主要就是想折腾看一看能不能 WSL 开发嵌入式，有什么问题希望多多指正。 
