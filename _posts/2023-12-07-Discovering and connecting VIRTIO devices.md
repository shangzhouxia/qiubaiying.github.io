---
layout:     post
title:      Discovering and connecting VIRTIO devices
# subtitle:    "\"Hello World, Hello Blog\""
date:       2023-12-07
author:     Underdog Linux
header-img: img/post-bg-2015.jpg
catalog: true
tags:
    - Using a QNX Hypervisor System
---

当guest关闭时，无论是受控关闭还是非受控关闭，qvm 进程都应尝试静默guest使用的任何物理设备。
这**可确保guest驱动程序不会继续写入物理内存**，这可能会使硬件处于意外甚至损坏的状态。

当guest关闭并因此其设备驱动程序消失时，主机硬件不能保持在相同状态，因为设备可能正在异步操作（例如，进行 DMA 写入、生成中断）。 为了保护硬件的完整性，可以将guest运行的 `qvm` 进程设计为在guest终止时静默任何此类设备（即使其进入休眠状态）。

为了遵循此设计，`qvm` 进程可以包含一个 `vdev`，用于静默设备并配置此 `vdev`，以便在 `qvm` 进程终止时、释放驱动程序进程的资源之前运行其回调函数。 回调函数必须执行任何所需的清理，包括静止。 这需要尽快关闭设备，以便当 `qvm` 进程消失时，设备不再：
- 写入物理内存（非常危险）
- 生成中断（危险性较小，因为hypervisor 可以确认 IRQ 并忽略它）

有关创建自定义 `vdev` 的信息，参阅[ Virtual Device Developer's Guide](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.vdev/topic/about.html)

有关定义 vdev 的控制函数以注册在进程关闭期间运行的回调的信息，参阅[Handling a qvm termination](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/qhs/qvm_term.html)

```markdown
guest 可以尝试让设备停顿； 只是hypervisor 不能依赖guest这样做。 
运行 QNX Hypervisor for Safety 变体时，您必须提供 vdev 以在关闭期间使物理设备停顿。 
对于非安全 QNX Hypervisor 变体，建议但不是必需的。
```

有多种方法可用于将guest中的设备驱动程序连接到其相应的hypervisor 主机设备。

在guest中运行的设备驱动程序可以使用以下方法之一连接到主机中的 `VIRTIO vdev`：
- [PCI discovery](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/use/virtio_guest.html#virtio_guest__pci)
- [Direct memory mapping](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/use/virtio_guest.html#virtio_guest__mmap)

您使用的方法取决于您的guest 的配置方式，以及您希望更改设备位置的难易程度。
例如，在生产系统中，`direct memory mapping`（在配置中写入设备内存中的位置）可能是可以接受的，甚至是更可取的，但在项目的开发阶段，您可能需要更灵活的方法，因此可能需要 使用 `PCI discovery`。

> 有关 `devb-ahci` 和 `devb-virtio` 等驱动程序及其选项的更多信息，请参阅 [Utilities Reference ](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.neutrino.utilities/topic/about.html)。

# PCI discovery
`PCI discovery`方法在为 x86 平台设计的guest中最常见，但 ARM 平台上的客户机也可以使用它。 
PCI 硬件不需要支持这种发现机制，因为在hypervisor中，guest PCI 是完全虚拟的。

以下步骤说明了如何使用 PCI mapping 来使guest中的驱动程序能够发现并连接到 VIRTIO 设备。 
该示例使用 `virtio-blk vdev`，但也可以使用其他 VIRTIO vdev，例如 virtio-console 或 virtio-net。

使用 virtio-blk 作为我们的示例，以及 QNX guest：

1. 在hypervisor 主机中，启动 AHCI SATA 接口 (`devb-ahci`) 的驱动程序。
2. 配置托管guest的虚拟机，通过 VIRTIO 将 AHCI SATA 驱动程序的位置（位于 `/dev/hd1t178`）映射到guest； 例如：
```markdown
vdev virtio-blk
        hostdev /dev/hd1t178
        name virtio-blk_qvm178
```
有关在虚拟机中配置此 vdev 的更多信息，请参阅 [vdev virtio-blk](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/vdev_ref/vdev_virtio-blk.html)。

3. 启动 QNX guest。
4. 在来宾中，运行 `pci-server`（请参阅Utilities Reference中的 [pci-server](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.neutrino.utilities/topic/p/pci-server.html)）； 例如：
```markdown
pci-server –bus-scan-limit=8
```

5. 在 QNX guest 中，使用 `pci-tool` 查询 PCI 设备（请参阅Utilities Reference中的 [pci-tool](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.neutrino.utilities/topic/p/pci-tool.html)）； 例如：
```markdown
[x86 guest QNX 7.1]% pci-tool -v

B000:D00:F00 @ idx 0
        vid/did: 1c05/0002
                BlackBerry QNX, n/a QVM PCI host bridge
        class/subclass/reg: 06/00/00
                Host-to-PCI Bridge Device
 
B000:D01:F00 @ idx 1
        vid/did: 1af4/1042
                <vendor id - unknown>, <device id - unknown>
        class/subclass/reg: 01/80/00
                Other Mass Storage Controller
 
B000:D02:F00 @ idx 2
        vid/did: 1af4/1041
                <vendor id - unknown>, <device id - unknown>
        class/subclass/reg: 02/80/00
                Other Network Controller
 
B000:D03:F00 @ idx 3
        vid/did: 1c05/0001
                BlackBerry QNX, n/a QVM guest shared memory factory
        class/subclass/reg: 05/80/00
                Other Memory Controller
 
B000:D04:F00 @ idx 4
        vid/did: 1af4/1043
                <vendor id - unknown>, <device id - unknown>
        class/subclass/reg: 07/80/00
                Other Simple Communications Controller
```

请注意，磁盘控制器的 VID/DID（vendor ID/device ID）为 1af4/1042； 这是大容量存储设备的 VIRTIO 标准参考。 
其他设备（例如内存设备或网络设备）也将显示为 PCI 设备。

在 Linux 客户机中，您可以使用 `lspci`。 您可能会看到VID/DID为1c05/0042； 这是 BlackBerry VIRTIO 供应商 ID。 Linux 内核模块知道这是一个块设备。

6. 在 QNX 客户机中，启动块设备的 VIRTIO 驱动程序 (devb-virtio)； 例如：
```markdown
devb-virtio
```
驱动程序会扫描PCI空间，找到具有`1af4/1042`供应商ID/设备ID的设备，并将其挂载为块卷； 例如，上面的命令将在 guest 虚拟机中创建 `/dev/hd0`。

7. 如果块存储卷已格式化为 QNX6 power-safe 文件系统，则可以挂载它； 例如：
```markdown
mount -t qnx6 /dev/hd0 /mydisk
```

# Direct memory mapping
以下步骤说明如何在主机中映射设备，然后将映射传递到guest中的驱动程序。 
该示例使用 `virtio-blk vdev`，但也可以使用其他 VIRTIO vdev，例如 `virtio-console` 或 `virtio-net`。

使用 virtio-blk 作为我们的示例，以及 QNX guest：

1. 在hypervisor 主机中，启动相应接口的驱动程序（例如 devb-ahci）。
2. 使用主机文件的路径配置虚拟机以用于设备的内容(参阅[Virtual Device Reference](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/vdev_ref/vdev_ref.html)章节的[vdev virtio-blk](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/vdev_ref/vdev_virtio-blk.html))

使用guest-physical内存中未由guest中的任何其他驱动程序使用的位置，以及未由guest中的任何其他驱动程序或服务使用的中断。 在此示例中，我们使用 0x1c0d0000 和 41：
```markdown
vdev virtio-blk
        loc 0x1c0d0000
        intr gic:41
        hostdev /dev/hd1t178
        name virtio-blk_qvm178
```

3. 启动 QNX 来宾。 不需要 PCI server 。
4. 在 guest 虚拟机中启动 VIRTIO 块设备驱动程序 (`devb-virtio`)，使用驱动程序的启动选项映射内存并在 `virtio-blk vdev` 配置中指定中断； 在我们的示例中，选项如下：
```markdown
devb-virtio virtio smem=0x1c0d0000,irq=41
```

guest驱动程序现在应该通过 VIRTIO 设备找到主机中的设备。

> 您可以使用FDT来扩展direct mapping以支持设备的动态发现。 例如，在 ARM 平台上，您可以使用 FDT 中的条目向guest提供有关虚拟设备的信息。
有关在 QNX hypervisor 系统中使用 FDT 的更多信息，参阅[Configuration](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/config/config.html)章节的[ACPI tables and FDTs](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/config/acpi_fdt.html)
