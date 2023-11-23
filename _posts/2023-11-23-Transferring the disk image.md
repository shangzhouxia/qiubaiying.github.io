---
layout:     post
title:      Transferring the disk image
# subtitle:    "\"Hello World, Hello Blog\""
date:       2023-11-23
author:     Underdog Linux
header-img: img/post-bg-2015.jpg
catalog: true
tags:
    - Building a QNX Hypervisor system
---

这些说明解释了如何将 QNX hypervisor 系统安装到支持的硬件平台上。

# 关于可移动媒介
您可以在其上运行 QNX hypervisor 的硬件平台支持各种可移动介质，包括 USB 闪存盘、SD 卡和 SATA 驱动器。
并非所有受支持的硬件平台都支持相同的可移动介质。 本指南提供有关 USB keys 和 SD 卡的说明，
因为每个受支持的平台至少支持其中一种类型的可移动介质。

提取后，镜像会扩展至约 2 GB。 对于可移动存储，我们建议：
- SD 或 micro SD 卡，具体取决于主板：8 GB Class 10
- USB key：8 GB USB 3.0 闪存驱动器

如果您的硬件平台具有 SATA 驱动器，您可以使用它来代替 USB 闪存盘或 SD 卡。

> 不要写入存储介质的分区文件（例如 /dev/sdb1 或 /dev/rdisk3s1）。 写入原始设备文件。

# USB keys
您可以在hypervisor 支持的任何硬件平台上使用 USB key。 如果您这样做：
- 请勿使用具有安全功能的 USB key。
- 如果您的 USB 闪存盘有问题，请尝试将其重新格式化为 `FAT32` 文件系统，然后将分区类型转换为 `GPT`。 

在 Windows 系统上，您可以使用 `MiniTool` 分区向导免费工具来进行这些更改。 
Linux系统有自己的工具，例如diskutil。

# SD cards
如果您的平台支持 SD 或 micro SD 卡，我们建议使用 UHS-I 卡以获得更好的读/写性能。 这些卡可以通过里面有数字“1”的“U”来识别，如下图：
![uhs-1.png](/img/uhs-1.png)


# Linux
在 Linux 主机系统上，使用以下命令行指令将镜像复制到可移动存储：
> sudo dd bs=1024k if=base_dir/diskimage of=/dev/sdb

其中，base_dir 是您的hypervisor 项目工作目录，diskimage 是您的hypervisor 系统磁盘映像，USB key在您的主机系统上显示为 /dev/sdb。

此命令使 `dd` 实用程序一次将数据以 1 MB 为单位写入可移动存储。

> 设备名称不应包含分区后缀。 例如，请勿使用 /dev/sdb1。 但是，在某些 Linux 变体上，设备名称可能是 /dev/mmcblk0。

# Windows
要将磁盘镜像从 Windows 主机复制到目标，
您需要在主机系统上安装 `Win32 Disk Imager`。 如果您没有此实用程序，请从此站点下载它，然后安装：
[http://sourceforge.net/projects/win32diskimager/](https://sourceforge.net/projects/win32diskimager/)

在 Windows 系统上，要将镜像复制到可移动存储：
1. 运行 `Win32 Disk Imager`
2. 浏览到放置镜像的位置，然后单击“Open”。
3. 单击“Write”将 .img 文件写入您的 USB 闪存盘。
4. 单击“Yes”开始写入镜像的过程。 完成后，您将看到“Write successful.”消息。
5. 单击“OK”，然后退出 `Win32 Disk Imager`。
