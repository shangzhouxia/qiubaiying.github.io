---
layout:     post
title:      Filesystem Sharing
# subtitle:    "\"Hello World, Hello Blog\""
date:       2023-11-06
author:     Underdog Linux
header-img: img/post-bg-2015.jpg
catalog: true
tags:
    - Filesystem Sharing
---

共享文件系统框架允许guest通过他们选择的挂载点访问host上的目录。
该框架通过提供虚拟文件系统设备来实现这一点，通过该设备，guest可以与共享host目录中的文件进行交互，就好像它们是本地文件一样。

Host可以控制它共享的目录。 
许多guest可以访问相同的共享目录，但每个guest可以为包含该目录的虚拟文件系统选择自己的挂载点。

> 该框架被认为是实验性的，因为定义虚拟设备接口或使用该接口的guest操作系统组件的 VirtIO 规范可能会发生变化。 因此，guest或Host组件可能需要在未来版本中进行更新。

本章介绍了框架的体系结构，其中hypervisor 管理guest应用程序和共享Host目录之间的交互、使Host目录可供guest使用的虚拟文件系统设备配置，以及guest和Host所需的共享文件系统框架组件。

[Framework architecture](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.qavf.overview/topic/virtfs_arch.html)
<br />
共享文件系统框架（或虚拟文件系统框架）使在虚拟机 (VM) 中运行的guest能够访问 QNX Hypervisor Host共享的目录中的文件。

[Required components](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.qavf.overview/topic/virtfs_components.html)
<br />
要支持 QNX Hypervisor 系统中的虚拟文件系统，您的Guest和Host映像中必须具有某些组件，并在Guest虚拟机中配置 `virtio-fs` vdev。

[vdev virtio-fs](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.qavf.overview/topic/vdev_virtio-fs.html)
<br />
模拟支持 FUSE 协议的虚拟文件系统
