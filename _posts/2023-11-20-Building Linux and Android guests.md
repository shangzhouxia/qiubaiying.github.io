---
layout:     post
title:      Building Linux and Android guests
# subtitle:    "\"Hello World, Hello Blog\""
date:       2023-11-20
author:     Underdog Linux
header-img: img/post-bg-2015.jpg
catalog: true
tags:
    - Building a QNX Hypervisor system
---

QNX hypervisor 支持 Linux 和 Android guest，前提是这些guest是为虚拟硬件构建的，hypervisor 可以将自身呈现给 VM 中的Guest。

# Building Linux guests
以下信息只是在 QNX hypervisor 系统中实施 Linux guest系统所需执行的操作的概述。 
有关如何配置和构建 Linux 系统的信息，请参阅 Linux 文档。

当您构建非 QNX guest以在 QNX 虚拟化环境中运行时，您必须为正确的硬件架构构建它们，并配置它们将在其中运行的虚拟机以符合其期望。
要以guest身份实现 Linux 操作系统，您需要执行以下操作：
1. 从您最喜欢的 Linux 源获取适合构建您的 Linux 操作系统的 Linux 构建环境，并对其进行配置。
2. 按照 Linux 说明在 Linux 工作目录中构建 guest 虚拟机。
3. 编写一个 VM 配置文件（例如 `linuxvm1.qvmconf`），该文件将 Linux 操作系统期望在其运行平台上找到的组件组装在 VM 中。
对于示例文件，参阅下面的[Configuration file for VM hosting a Linux guest (example)](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/build/linux.html#linux__linuxconf)

4. 当您构建了 Linux guest后，您可以将其包含在新的hypervisor 磁盘映像中，然后将其传输到您的目标。


# Adding drivers, applications, and utilities
为了在虚拟化环境中发挥作用，您的 Linux guest可能需要包含一些额外的驱动程序、应用程序和实用程序。 例如：
1. 要使用hypervisor 提供的 VIRTIO 硬件，必须将 Linux guest核配置为包含 VIRTIO 驱动程序（例如，块、网络）。 有关更多信息，请参阅 Linux 文档。
2. 如果您想在 Linux guest中使用共享内存演示应用程序，则需要在构建 Linux 内核时包含该应用程序。 该应用程序的源代码可以在 HHBSP `src/apps/hypervisor/demos/shmem-linux/` 目录中找到(参阅[The HHBSP framework](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/build/hhbsp.html))
3. 如果要使用虚拟硬件watchdog，则必须在 Linux 内核中包含watchdog设备的驱动程序：SP805 （ARM） 或 IB700 （x86），并在将托管 Linux guest的 VM 中包含和配置相应的watchdog vdev。(参阅[QNX Hypervisor: Protection Features](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/qhs/qhs.html)章节的[Watchdogs](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/qhs/watchdog.html))

# 使命令行参数可供 Linux guest使用
启动 Linux guest的方式与启动 QNX guest的方式相同：
通过命令行启动 VM，
<br />
或者通过向hypervisor 主机startup 例程添加用于启动托管 qvm 进程实例的指令来启动 VM(参阅[Starting and using guests](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/start/guests.html))。

但是，当您启动 Linux guest时，您可能需要为其提供一些命令行参数。 
为此，您可以使用 `qvm cmdline` 配置选项将启动参数传递给 Linux 内核。(参阅[VM Configuration Reference]()章节的[cmdline](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/vm/cmdline.html))
例如，我们可以使用以下命令告诉 Linux 内核在哪里可以找到控制台：
- On ARM platforms:
```markdown
cmdline
    console=ttyAMA0 earlycon=pl011,0x1c090000
```
将 Linux 内核指向位于guest物理内存中位于 0x1c090000 处的虚拟 PL011 UART 设备。
不要忘记将此设备包含在 VM 配置中。

- On x86 platforms:
```markdown
cmdline
    pmtmr=0 nolapic_timer
    tasks=standard pkgsel/language-pack-patterns=
    pkgsel/install-language-support=false
    console=ttyS0,115200n8 reboot=t
```
指向虚拟 8250 UART（ttyS0 是 Linux 中 8250 UART 设备的设备名称）。 
不要忘记将此设备包含在您的虚拟机配置中。

# Booting Linux guests
要从磁盘启动，Linux guest需要知道根文件系统安装在哪个分区（即安装在位置 / 的文件系统）。 
获得此信息后，您可以在 VM 配置文件中使用command-line参数指定该信息。 例如：
```markdown
cmdline
    root=/dev/vda1
```
其中 `vda1` 指虚拟机配置中的第一个 `virtio-blk` vdev 条目(更多信息，参阅[Making command-line arguments available to a Linux guest](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/build/linux.html#linux__cmdline))

如果您将 `virtio-blk` 设备用于磁盘，**请记住，在 Linux 中，这些设备显示为** /dev/vda*、/dev/vdb* 等。
如果您的设备是pass-through设备，您需要知道其设备条目（可能是 /dev/hda*）并让 Linux guest知道它以便它可以启动。

# Configuration file for VM hosting a Linux guest (example)
下面是 Linux 客户机的 `*.qvmconf` 文件的示例。 
请注意，在设计为在 x86 平台上运行的操作系统期望找到 VGA 设备和 BIOS 的区域中使用保留和 rom 选项。
```markdown
# A minimalist VM configuration for a Linux guest on an x86 system
ram 0,0xa0000
reserve loc 0xa0000,0x20000
rom 0xc0000,0x40000
ram 1m,1023 cpu
cpu
load ./linux
initrd load ./initrd.gz
cmdline "pmtmr=0
	nolapic_timer tasks=standard pkgsel/language-pack-patterns=
	pkgsel/install-language-support=false console=ttyS0,115200n8
	reboot=t root=/dev/vda1"
vdev ioapic
        loc 0xf8000000
        intr apic
        name myioapic
vdev ser8250
	intr myioapic:4
vdev timer8254
	intr myioapic:0
vdev mc146818
	reg 0x0b,0x02
vdev shmem
vdev virtio-net
        peer /dev/vdevpeers/vp0
		peerfeats 0x00007fc3
        loc pci:0:1
        name guest_to_host
vdev virtio-blk
        hostdev /dev/hd1t131
vdev pckeyboard
vdev 8259
	loc 0x20
vdev 8259
	loc 0xa0
vdev hpet
        loc 0xf8008000
        intr myioapic:0,myioapic:8,myioapic:10
pass loc pci:0:2.0=pci:0:2.0
pass loc pci:0x8086/0x5aa8
vdev pci-dummy clone pci:0:31.0
```

# 构建并包含 Android guest 虚拟机
构建Android guest，需要:
- 主机系统上的 Android 构建环境（例如，在 HHBSP 的 `images/guest_bsps/android/` 目录中）
- Android kernel

按照构建适用于您的 Android 操作系统的 Linux guest的概要说明进行操作。

> 请记住，您的 Android guest必须针对您正在使用的架构和主板进行构建，并且您的托管 VM 必须配置为符合guest的期望。
