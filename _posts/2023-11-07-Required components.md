---
layout:     post
title:      Required components
# subtitle:    "\"Hello World, Hello Blog\""
date:       2023-11-07
author:     Underdog Linux
header-img: img/post-bg-2015.jpg
catalog: true
tags:
    - Filesystem Sharing
---

要在 QNX Hypervisor 系统中支持虚拟文件系统，Guest和Host镜像中必须包含某些组件，并在Guest的虚拟机中配置 `virtio-fs` vdev。

# Host components
要通过虚拟文件系统与Guest共享Host目录，hypervisor Host构建文件必须在Host镜像中包含以下组件：

- 虚拟文件系统 vdev 的共享对象，virtio-fs.so
- 对于每个Host想要访问Host目录的Guest的虚拟机，必须包含 `virtio-fs` vdev。关于配置Vdv，参阅[vdev virtio-fs](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.qavf.overview/topic/vdev_virtio-fs.html)
- 每个 VM 的配置文件（`*.qvmconf` 文件）的路径
- FUSE 文件系统守护程序、passthrough和支持协议库 libfuse3.so
- 用于启动此守护程序 `fuselauncher` 的实用程序。
此实用程序必须在Host启动时由 `virtio-fs` vdev 启动，因此虚拟文件系统在Guest的 VM 启动时可用。

更多有关于配置hypervisor host的信息，参阅QNX Hypervisor [ User's Guide](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/about.html)中的"Configuration"一节。

# Guest components
`virtio-fs` vdev 是一个半虚拟化(para-virtualized)设备。
它在hypervisor  VM 中运行，但不模拟存储文件的物理设备。
相反，它提供了操作系统在非虚拟化环境中提供的文件系统功能。

由于 vdev 是半虚拟化设备，因此Guest必须知道它运行在虚拟化环境中，并将 VirtIO 文件系统内核驱动程序（例如，Android 或 Linux 客户机的 `virtio-fs.ko`）添加到其镜像中。

参阅`QNX Hypervisor User's Guide`中的[Building guests](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/build/build_guest.html)获取更多指导信息。
