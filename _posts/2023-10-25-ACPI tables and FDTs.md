---
layout:     post
title:      ACPI tables and FDTs
# subtitle:    "\"Hello World, Hello Blog\""
date:       2023-10-25
author:     Underdog Linux
header-img: img/post-bg-2015.jpg
catalog: true
tags:
    - QNX Hypervisor Configuration
---

> [Assembling and configuring VMs](https://www.qnx.com/developers/docs/7.1/index.html#com.qnx.doc.hypervisor.user/topic/config/qvm.html)的子部分

QNX  hypervisor虚拟机为其guest提供高级配置和电源接口 （ACPI:Advanced Configuration and Power Interface） 表以及扁平化设备树 （FDT）。

在 QNX hypervisor 系统中，guest可用的设备在其 VM 的配置中指定（即在托管 qvm 流程实例的配置中）。 
如果 guest 虚拟机需要 ACPI 表或 FDT 来枚举可用的设备，您可以修改 ACPI 表和 FDT，并让 qvm 进程实例组装 VM 将它们加载到 guest 虚拟机内存中，以便 guest 虚拟机在启动时找到它们 。

# ACPI tables (x86)
在 x86 平台上的 QNX hypervisor VM 中运行的guest可以访问其 VM 的 ACPI 表。 从guest的角度来看，这些表在虚拟机中的位置与在硬件中的位置相同； 也就是说，如果表位于host-physical内存中的位置 `0x12340000`，则guest将在`guest-physical`内存中的位置 `0x12340000` 处找到表。 
检查您的主板规格以了解 ACPI 表的位置。
<br />
您还可以创建自己的 ACPI 表来补充主板固件和hypervisor提供的表，并使用 VM 配置加载选项将其加载到guest内存中。 例如：
> acpi load ./acpi_foo

将导致组装 VM 的 qvm 进程实例将 `acpi_foo` 文件作为 ACPI 表加载到guest内存中。
(参考[VM Configuration Reference](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/vm/vm.html)章节的[load](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/vm/load.html))。

# FDTs (ARM)
在 ARM 平台上，您可以创建 FDT 并使用 VM 配置加载选项将其加载到guest内存中。
(参考[VM Configuration Reference](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/vm/vm.html)章节的[load](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/vm/load.html))

例如，某些操作系统（例如 Linux）可能还需要 FDT，其中包含有关传递给它们的设备的信息。 在这种情况下，您必须将描述pass-through设备的 FDT 传递给guest：
1. 从硬件平台附带的 FDT 开始。
2. 如果 FDT 无法以人类可读的格式提供，请使用 dtc 实用程序将二进制 FDT 转换为 *.dts 人类可读的文件。
3. 编辑人类可读的 FDT 文件，**删除**除描述您将传递给guest的设备的条目之外的所有条目。
4. 使用 dtc 实用程序从编辑的文件创建新的 FDT 二进制文件 (*.dtb)。
5. 在将托管需要 FDT 信息的guest的 VM 的配置中，使用 VM 配置加载选项将其加载到guest内存中。
例如，以下 VM 配置将 `fdt_foo.dtb` FDT 二进制文件加载到guest内存中，并将具有三个通道的 USB 设备传递给来宾：

```c
fdt load ./fdt_foo.dtb
					
## USB2.0 Host (EHCI/OHCI) channels 0,1,2
pass loc 0xEE080000,0x1000,rw
pass loc 0xEE0A0000,0x1000,rw
pass loc 0xEE0C0000,0x1000,rw
pass intr gic:144
```

如果您的guest需要来自 FDT 的附加信息（例如 CPU、时钟），您可以使用相同的方法将包含相关信息的 FDT 加载到guest的内存中。

> - 有关如何从用户提供的 FDT 引用 vdev FDT 节点的信息，参阅[Referencing a vdev FDT node from a user-provided FDT](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/config/guests.html#guests__irq_parent)
> - 更多有关FDT的信息，参阅[www.devicetree.org](http://www.devicetree.org/)
> - dtc 实用程序可从[https://git.kernel.org/cgit/utils/dtc/dtc.git](https://git.kernel.org/cgit/utils/dtc/dtc.git) 获取；Device Tree Compiler Manual 可以在以下位置找到：[web.mit.edu/freebsd/head/contrib/dtc/Documentation/manual.txt](web.mit.edu/freebsd/head/contrib/dtc/Documentation/manual.txt)
