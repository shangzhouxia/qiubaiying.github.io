---
layout:     post
title:      构建QNX Hypervisor系统的方法
# subtitle:    "\"Hello World, Hello Blog\""
date:       2023-10-26
author:     Underdog Linux
header-img: img/post-bg-2015.jpg
catalog: true
tags:
    - Building a QNX Hypervisor system
---

构建 QNX Hypervisor 系统需要将虚拟化添加到 QNX Neutrino 微内核系统。

有两种方法可以做到这一点：
1. 如果您已经启动non-hypervisor QNX Neutrino 目标系统，请按照[Build a hypervisor based on a BSP](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/build/build_methods.html#build_methods__1)部分进行操作。 
这解释了如何将虚拟化扩展模块和hypervisor 二进制文件添加到现有版本中。
2. 如果您没有现有的可启动目标,参阅[Build a hypervisor in the HHBSP framework (default settings)](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/build/build_methods.html#build_methods__2)和[Build a hypervisor in the HHBSP framework (custom settings)](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/build/build_methods.html#build_methods__3)部分。
有一些脚本和示例镜像配置文件向您展示如何收集从头开始制作 hypervisor-enabled 的bootable 目标镜像所需的二进制文件。

然而，在这两种情况下，过程是相同的：
1. 将虚拟化模块 (`libmod_qvm.a`) 添加到可启动的 QNX Neutrino 系统。
2. 使hypervisor 二进制文件可供目标系统使用。 这些二进制文件是 qvm 及其支持的 `vdev-*.so` 文件。
3. 在启动`qvm`进程、`smmuman`服务、`PCI`服务等之前确保运行环境正确。


# 构建基于 BSP 的hypervisor 
使用此方法，您可以从满足硬件要求的主板的 BSP 开始，以支持hypervisor，编辑构建文件以包含hypervisor模块、vdev 和其他组件，并以其他方式配置您的构建，然后在任何位置构建主机 很方便。 <br />然后，您可以将主机及其guest组装成镜像并将其传输到您的目标。

当您准备production 系统时，这是一个很好的方法，因为它使您可以完全控制成品系统的内容和配置。
![qvm_nohhbsp.png](/img/qvm_nohhbsp.png)

有关如何使用此方法的分步说明，参阅[Building the host](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/build/build_host.html)和[Building guests](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/build/build_guest.html)。

# 在HHBSP框架中构建hypervisor（默认设置）
这种构建hypervisor 的方法使用：
- the hypervisor host BSP (**HHBSP**) framework
- 主机板特定的 BSP
- 所有组件都位于其默认位置
- 默认环境和/或创建变量和构建配置

下图说明了HHBSP框架及其使用不同的BSP：
与板无关的 HHBSP，具有用于为hypervisor 主机创建 IFS 的板特定 BSP、用于为 QNX guest创建 IFS 的 QNX guest BSP 以及用于为 Linux guest创建镜像的 Linux 包。
![qvm_hhbsp.png](/img/qvm_hhbsp.png)

此方法可能是首次构建hypervisor 系统时使用的最佳方法。 
它将构建一个可用于熟悉hypervisor 及其功能的系统。

有关如何使用此方法的分步说明，参阅[The HHBSP framework](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/build/hhbsp.html)和[Building in the HHBSP](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/build/build_hhbsp.html)。

# 在HHBSP框架中构建hypervisor（自定义设置）
这种构建hypervisor 的方法也使用 HHBSP 框架，但不需要您将组件（特定于板的 BSP、guest BSP）放置在特定位置。
<br />您可以修改配置文件 (`configure.mk`) 或使用命令行设置环境变量，以便可以将 BSP 保留在 HHBSP 框架之外并只需指向它们。

当您开始开发hypervisor 项目时，这可能是用于原型设计的最佳方法。 
您可以在方便的位置拥有不同的 BSP，修改 BSP，并且当您想要尝试新配置和新组件时，只需更改环境变量以指向某些位置。

有关如何使用此方法的分步说明，参阅[The HHBSP framework](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/build/hhbsp.html)和[Building in the HHBSP](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/build/build_hhbsp.html)。
