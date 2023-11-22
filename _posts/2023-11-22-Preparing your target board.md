---
layout:     post
title:      Preparing your target board
# subtitle:    "\"Hello World, Hello Blog\""
date:       2023-11-22
author:     Underdog Linux
header-img: img/post-bg-2015.jpg
catalog: true
tags:
    - Building a QNX Hypervisor system
---

本节介绍如何准备受支持的硬件板以启动和运行 QNX hypervisor系统。

为 QNX hypervisor系统准备目标板与为非虚拟化系统准备目标板没有什么不同。 
您需要设置 DIP 开关来配置板行为（例如在何处查找 IPL 和启动代码），并连接以太网、USB 和串行电缆，以便可以连接到主机系统和网络（请参阅 您的主板的`BSP User's Guide`以及主板制造商的文档）。

# Intel Supermicro Denverton
在 Intel Supermicro Denverton 主板上，系统 shell 仅在串行端口上处于活动状态。 
它在 VGA 控制台上未激活。 确保通过串行端口将开发主机连接到目标板。 
如果您通过 VGA 控制台连接，您将看到一个徽标，并且主板似乎在启动过程中的某个位置停滞了。

在 Supermicro Denverton 等 x86 主板上，默认 `smmuman` 配置指示服务使用主板的 ACPI 表来获取负责重新映射 PCI 设备 DMA 的 VT-d 寄存器的位置。 
确保您的启动不包含 -B 选项，该选项指示启动不获取 ACPI 表。

更多信息，参阅[VM Configuration Reference](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/vm/vm.html)章节的[pass](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/vm/pass.html)，和 [QNX Hypervisor: Protection Features](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/qhs/qhs.html)章节的[DMA device containment (smmuman)](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/qhs/dmadevcontain.html)。

# Configuring the BIOS settings
为了支持启动hypervisor，Supermicro Denverton 需要特定的 BIOS 设置。 要配置并验证这些：
1. 通过连接电源为 Supermicro Denverton 板通电。
2. 按`Delete` 键进入BIOS 设置屏幕。
3. 在 BIOS 设置屏幕中，验证 BIOS 设置如下所示：
    **Initial screen** > **Boot Order panel**
    - UEFI boot must be selected
    - usbstick must appear first in the Boot Drive Order
4. 将这些设置保存到 BIOS，然后重启主板。

# Renesas R-Car H3
为了运行虚拟化系统，ARM 板（例如 Renesas R-Car H3 板）需要启动到异常级别 2 (EL2) 的固件。 在某些较旧的主板上，出厂时未安装所需的固件。 如果您尝试在这些板上启动hypervisor，启动将失败并显示如下消息：
```markdown
# qvm @qnx71.qvmconf
QVM disabled: hypervisor support not available
```

如果您的主板无法启动至 EL2，则您可能拥有 QNX hypervisor不支持的旧主板版本。 请联系您的主板制造商，获取具有更新 BSP 且受支持的更新主板。

对于 R-Car H3 主板，为了支持 QNX hypervisor，您的主板必须设置为 64 位模式并以 Hyperflash 模式启动。 
请参阅您的主板的`BSP User's Guide`，了解用于您的主板型号和版本的 DIP 开关设置。
