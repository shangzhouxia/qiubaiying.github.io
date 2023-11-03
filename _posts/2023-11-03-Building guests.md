---
layout:     post
title:      Building guests
# subtitle:    "\"Hello World, Hello Blog\""
date:       2023-11-03
author:     Underdog Linux
header-img: img/post-bg-2015.jpg
catalog: true
tags:
    - Building a QNX Hypervisor system
---

当您构建在 QNX 虚拟化环境中运行的guest时，您必须为适当的硬件架构构建它们，并配置它们将在其中运行的虚拟机。

您需要拥有与hypervisor 和您正在构建的guest相对应的构建环境和工具(参阅[Supported build environments](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/build/build_env.html))。

# Validate the guests without the hypervisor
在非虚拟化系统上识别和调试问题比在运行hypervisor 的系统上要容易得多。
如果可能，在将guest添加到hypervisor 系统之前，应直接在硬件上运行来构建并测试它。

1. 如果您正在构建 QNX guest，请按照 `BSP User’s Guide` 中的说明为您的硬件平台构建 BSP 并将其复制到您的目标。
<br />对于非 QNX guest，请按照说明构建该访客并将其复制到目标。

2. 启动系统并确保它按要求运行。
<br />具体而言，请确保要配置为guest pass-through设备的所有设备都正常运行。
与具有hypervisor 的系统相比，这些设备的问题在非虚拟化系统中更容易解决。

3. 对将在hypervisor 系统中作为guest运行的每个操作系统重复上述步骤。

当您确信将作为guest运行的所有操作系统在非虚拟化系统中按要求运行时，可以将它们添加到hypervisor 系统映像中，并将它们复制到目标以作为guest运行。

> 在没有hypervisor 的情况下测试guest（即直接在硬件上运行）可能并不总是可行的。例如：
> - guest可以使用半虚拟化设备，根据定义，这些设备不作为硬件存在。
> - guest所需的启动驱动程序可能无法直接在开发板上工作;这通常可以通过更改guest的构建文件来纠正，以（暂时）使用适当的驱动程序直接在主板上启动。

# Supported guest formats in images
hypervisor 可以采用以下格式启动放置在系统上的guest作为可引导映像：
- ELF (including multiboot)
- Linux zImage


# Configuring the VMs
hypervisor 系统中的每个客户机都托管在hypervisor  qvm 进程中。
<br />启动时，qvm 进程的每个实例都会读取其指定的配置文件，并从此文件中指定的组件组装 VM。
此 VM 将成为运行客户机的虚拟环境。

如果更改guest中访问硬件（物理或虚拟）的任何内容，则需要确保正确配置了将托管guest的虚拟机的 qvm 配置文件。

在构建要在 QNX 虚拟化环境中运行的guest时，请务必记住，必须将每个guest配置为与运行该guest的虚拟机相匹配。
这意味着它必须具有访问设备所需的驱动程序，无论它们是物理设备还是虚拟设备。
在 VM 中，驱动程序必须配置为位于guest期望的位置。


在 QNX 操作系统系统中，板卡专用驱动程序和其他组件通过 BSP 引入构建中。
对于将在虚拟机中运行的 QNX 操作系统来说，就像在硬件上运行的 QNX 操作系统一样，也是如此。

对于 ARM 和 x86 硬件平台，您需要支持的硬件平台的 BSP 来构建hypervisor 主机域，以及hypervisor guest BSP 来构建每个guest。
<br />这些产品适用于 QNX Neutrino 7.1 和 QOS 2.2.1 客户机。
如果为guest添加新设备，则可能需要向 BSP 添加相应的新驱动程序，并使用它们重建guest。

对于 QNX guest而言，对于非 QNX guest也是如此：虚拟机必须向guest提供guest操作系统期望找到的虚拟环境（即，它所构建的硬件组件）。
唯一的区别是，QNX 操作系统（包括在虚拟化环境中作为guest运行的 QNX 操作系统）使用 BSP 来引入特定于架构和特定于主板的组件，而其他操作系统可能使用其他机制来实现相同的目的。

更多信息参阅[Assembling and configuring VMs](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/config/qvm.html)和 [Configuration](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/config/config.html) 章节。


