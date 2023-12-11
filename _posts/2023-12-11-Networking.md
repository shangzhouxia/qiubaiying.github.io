---
layout:     post
title:      Networking
# subtitle:    "\"Hello World, Hello Blog\""
date:       2023-12-11
author:     Underdog Linux
header-img: img/post-bg-2015.jpg
catalog: true
tags:
    - Using a QNX Hypervisor System
---

您可以建立Host与guest之间、guest与guest之间、guest与外界之间的通信。

当您为guest设计网络接口时，您可以使用：
- 一个 `virtio-net vdev` 连接到另一个 `virtio-net vdev`，通常在另一个 guest 虚拟机中
- 连接到在hypervisor 主机中运行的 `devnp-vdevpeer-net.so` 驱动程序实例的 `virtio-net vdev`
- pass-through 设备


`virtio-net vdev` 提供点对点通信；
更多信息，参阅《Virtual Device Reference》章节的[vdev virtio-net](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/vdev_ref/vdev_virtio-net.html)

`devnp-vdevpeer-net.so` 驱动程序在hypervisor 主机中运行。 
它使主机中的 `iopkt-* `服务能够通过托管guest的 qvm 流程实例中适当配置的 vdev 与guest进行通信。

更多信息，参阅《Utilities and Drivers Reference》章节的[devnp-vdevpeer-net.so](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/utils/devnp-vdevpeer-net.so.html)

> 不推荐使用 virtio-net 的 Tap 接口。


# MAC addresses in a hypervisor system
关于分配 MAC 地址，请注意以下事项：
- MAC 地址**在任何给定的以太网段中必须是唯一的**。 目标板上可以有多个以太网段，并且目标板可以与某些外部实体共享一个段。
- 如果您不指定 MAC 地址，vdev 和 devnp-vdevpeer-net 驱动程序将为您生成它们。 由于这些地址是生成的，因此每次重新启动 io-pkt-* 或 VM 时它们都会更改。
- 分配 MAC 地址时，应设置本地分配位（请参阅 IEEE 802.3-2015 第 1 部分，第 3.2.3 段，第 b 项）。


# Enabling peer-to-peer networking
为了支持hypervisor 中的guest之间以及guest与外部世界之间的连接，必须在主机中启用`peer-to-peer `网络以及网络驱动程序（例如 e1000）。

您可以通过在主机中启动 io-pkt-*、使用 `-d vdevpeer-net` 选项、指定物理网络驱动程序和要创建的对等接口来完成此操作。 
您可以在主机启动后使用命令行启动 io-pkt-*，也可以将 io-pkt-* 启动指令添加到hypervisor 主机的startup 脚本中。

以下命令启动将共享的主机网络驱动程序，并为hypervisor 系统中的三个guest中的每一个创建一个接口：
```markdown
io-pkt-v6-hc -d e1000 -d vdevpeer-net \
peer=/dev/qvm/qnx7-guest1/p2p,bind=/dev/vdevpeers/vp0,\
peer=/dev/qvm/qnx7-guest2/p2p,bind=/dev/vdevpeers/vp1,\
peer=/dev/qvm/linux-guest/p2p,bind=/dev/vdevpeers/vp2
```

io-pkt-* -d 选项启动 devnp-* 驱动程序（请参阅《QNX SDP Utilities Reference》中的 [io-pkt-v4-hc](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.neutrino.utilities/topic/i/io-pkt.html)、[io-pkt-v6-hc](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.neutrino.utilities/topic/i/io-pkt.html)）。 
在上面的示例中，仅当您的系统需要连接到外部世界时才需要 e1000 驱动程序（请参阅本章中的[Guest-to-world](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/network/guest2world.html)）。 
如果您只需要将guest相互连接，则可以省略此驱动程序。
