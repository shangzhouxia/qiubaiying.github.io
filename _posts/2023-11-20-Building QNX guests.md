---
layout:     post
title:      Building QNX guests
# subtitle:    "\"Hello World, Hello Blog\""
date:       2023-11-20
author:     Underdog Linux
header-img: img/post-bg-2015.jpg
catalog: true
tags:
    - Building a QNX Hypervisor system
---

就像为直接在硬件上运行而构建的 QNX OS 系统一样，为在 QNX hypervisor 环境中作为Guest运行而构建的 QNX OS 系统也使用 BSP，该 BSP 提供特定于架构的组件。

`qvm` 配置设置Guest将在其中运行的 VM。 
在大多数情况下，构建 QNX Guest只需要构建可启动映像，就像非虚拟化环境一样。


>构建 QNX Guest时：
>    - 请记住，Guest将在其中运行的 VM 必须符合Guest的期望：架构、主板特定信息、内存和 CPU、设备等。(参阅[Configuration](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/config/config.html)章节的[Assembling and configuring VMs](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/config/qvm.html))
>    - 请务必使用适合您的Guest操作系统版本（例如 QNX Neutrino 7.1）和您的主板架构的特定于架构的Guest BSP，而不是特定于主板的主板 BSP。
>    - 如果您向Guest添加pass-through设备，并且这些设备需要来自板特定 BSP 的驱动程序，请将这些驱动程序复制到Guest BSP 的 prebuilt/ 目录中的适当位置。

<br />

# 下载 QNX Guest BSP
特定于架构的 QNX Guest BSP 可从 QNX 软件中心获得;例如：
![qvm_swcref_pkgs.png](/img/qvm_swcref_pkgs.png)

# 编译 QNX Guest BSP
要构建 QNX Guest，假设您已在开发主机系统上设置了构建环境，请执行以下操作：
1. 设置环境变量，以便在Guest BSP 的根目录中构建 QNX 操作系统映像。
2. 如果您尚未这样做，请从 QNX 软件中心下载 x86 或 ARM QNX Guest的 BSP。
3. 解压缩 QNX Guest BSP 并将其放置在方便的位置。例如：
`# unzip ~/Downloads/BSP_hypervisor-guest-x86_br-710_be-710_SVN917988_JBN3.zip -d guest_bsp`
其中 guest_bsp 是放置Guest BSP 的位置。

4. 如果对Guest BSP 进行更改，请从Guest BSP 根目录运行 make。

现在，您可以将Guest IFS 添加到Host的bootable disk image中，并将其传输到目标。(参阅[Hypervisor disk images](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/build/create_image.html))
