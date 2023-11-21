---
layout:     post
title:      Hypervisor disk images
# subtitle:    "\"Hello World, Hello Blog\""
date:       2023-11-21
author:     Underdog Linux
header-img: img/post-bg-2015.jpg
catalog: true
tags:
    - Building a QNX Hypervisor system
---

如果您对hypervisor Host或Guest进行更改，则需要使用hypervisor 系统创建新的bootable disk image并将其传输到您的目标。

# Disk image partitions
hypervisor 磁盘镜像包括bootable 分区中的hypervisor主机和数据分区中的Guest：

**bootable partition**
类型 11 (DOS)。 包括至少一个用于hypervisor主机的可启动 IFS（通常是多个）。
您可以使用此分区将文件从任何可以访问 DOS 文件系统的开发主机复制到此磁盘。 如果您需要将文件从开发主机传输到目标，这非常有用。

**data partition**
177 型（QNX PowerSafe）。 包括参考映像中每个guest的 IFS，以及这些guest的每个 VM 的 qvm 配置文件（或多个文件）。

# Changing partition sizes
如果您需要更改分区的大小，在构建hypervisor系统之前（即在 HHBSP 根目录中运行 make 之前），您可以调整 `diskimage.cfg` 和 `*.partition.build.template` 中的磁盘配置 hypervisor Host BSP 的 `images/disk_config/` 目录中的文件(参阅[Adjust the disk image settings (optional)](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/build/build_hhbsp.html#build_hhbsp__3))

# Reducing boot time
您可以修改 `data.partition.build.template` 文件来加快启动时间：
- 从 IFS 构建文件中删除磁盘安装后不需要的所有文件。
- 将这些文件放在数据分区中（在`/guests/`目录中）。

启动速度会更快，因为可启动 IFS 越小，加载所需的时间就越少。
