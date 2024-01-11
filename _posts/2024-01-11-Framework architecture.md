---
layout:     post
title:      Framework architecture
# subtitle:    "\"Hello World, Hello Blog\""
date:       2024-01-11
author:     Underdog Linux
header-img: img/post-bg-2015.jpg
catalog: true
tags:
    - Input Sharing
---

共享输入框架使在虚拟机 (VM) 中运行的Guest 能够访问连接到 QNX Hypervisor Host的设备的输入。

> 本章假设您已经熟悉以下文档中介绍的概念：
> - QNX Hypervisor [User's Guide](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/about.html)
> - The [virtualization frameworks architecture](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.qavf.overview/topic/overview.html)

# 标准机制
输入框架实现了 VirtIO 1.1 规范的第 5.8 节，该规范定义了一种可扩展的、独立于平台的方法来创建虚拟输入设备。
该框架提供了一种虚拟设备 (vdev) `virtio-input`，它提供一个或多个物理输入设备的功能，并与hypervisor guest 虚拟机的标准 VirtIO 输入驱动程序配合使用。

> 译者注
> VirtIO 1.1 规范: https://docs.oasis-open.org/virtio/virtio/v1.1/csprd01/virtio-v1.1-csprd01.html#x1-3390008

`virtio-input` vdev 支持与Guest共享输入设备，并支持来自以下设备类型的输入：
- USB keyboard
- USB pointer
- Touch


# 输入共享
**host 管理** guests 和物理输入设备之间的所有交互：
- hypervisor Host**拥有物理输入设备**并**运行这些设备的驱动**程序。
- 如果 VM 中的 guest 虚拟机需要访问输入，则 VM 会向 guest 虚拟机提供 `virtio-input` vdev，并且 guest 虚拟机运行 VirtIO 输入驱动程序以与 vdev 交互。
- 为了相互通信，guest驱动程序和 vdev 使用 `virtqueues`，它充当 VirtIO 设备上批量数据传输的标准机制。


下图说明了通过hypervisor Host管理的Guest和输入设备之间的交互。
> Figure 1. Shared Input framework architecture


![shared_input_framework_overview.png](/img/shared_input_framework_overview.png)

> 有关 QNX Hypervisor 中设备共享的更多信息，请参阅QNX Hypervisor [User's Guide](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/about.html)的“Understanding QNX Virtual Environments”一章中的“Devices”部分。

# 输入事件的生命周期
如上图所示，`virtio-input` vdev 与 QNX 屏幕图形子系统（简称为 Screen）配合使用来管理对输入事件的访问。

以下是框架如何处理输入事件的典型示例：

1. 主机上运行的 Screen 实例接收该事件。
通常，输入事件在用户与物理设备交互时发生。 
然而，在某些情况下，应用程序通过调用 Screen 的 screen_inject_event() 函数以编程方式生成输入事件。
> 了解有关 Screen 及其如何管理输入的更多信息, 参阅 [Screen Developer's Guide](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.screen/topic/manual/cscreen_about.html)

2. `virtio-input` vdev 从 Screen 获取输入事件。
您可以配置 vdev 以控制它将向来宾提供哪些事件类型；
参阅[vdev virtio-input](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.qavf.overview/topic/input_config_vdev.html)

3. `virtio-input` vdev 将事件映射到等效的 Linux input event code，然后将映射的事件写入 virtqueue。

`virtio-input` vdev 将从 Screen 接收到的指针（鼠标）和键盘事件映射到 Linux input-event codes(https://www.kernel.org/doc/Documentation/input/event-codes.txt);

对于接收到的触摸事件，vdev 将它们映射为multi-touch (MT) 协议定义的一系列事件。 
该协议既适用于仅支持单点触摸（type A）的触摸硬件，也适用于支持多点触摸（type B）的硬件；
参阅  https://www.kernel.org/doc/Documentation/input/multi-touch-protocol.txt

对于 B 型硬件，vdev 支持最少两根手指触摸，最多五个手指触摸。

Linux input-event code被写入 virtqueue，充当 vdev 和 guest 驱动程序之间的通信通道。

4. Guest驱动程序读取 virtqueue 缓冲区的内容。
在Guest中，驱动程序将事件直接传递到用户空间到对该事件感兴趣的应用程序。 
任何Guest应用程序都必须设计为与 Linux input-event codes配合使用，以处理指针和键盘事件。
