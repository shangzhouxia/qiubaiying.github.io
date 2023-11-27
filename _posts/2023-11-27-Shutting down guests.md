---
layout:     post
title:      Shutting down guests
# subtitle:    "\"Hello World, Hello Blog\""
date:       2023-11-27
author:     Underdog Linux
header-img: img/post-bg-2015.jpg
catalog: true
tags:
    - Booting and Shutting Down
---

有多种方法可以关闭 QNX hypervisor 虚拟机中的Guest。

> 为了**避免在 I/O 操作期间终止驱动程序**，并可能使硬件处于不可恢复的状态，请使用 **SIGTERM** 或 **SIGINT** 信号来终止 qvm 进程。

关于如何关闭hypervisor，参考[Shutting down the QNX hypervisor](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/start/hyp_stop.html)。
有关如何处理 VM 终止的信息，参考[QNX Hypervisor: Protection Features](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/qhs/qhs.html)章节的[Handling a qvm termination](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/qhs/qvm_term.html)。

## 内部（受控）guest shutdown
guest （更具体地说，在guest操作系统上运行的应用程序）可以启动自己的关闭。 
例如，QNX guest中的应用程序可以使用 `shutdown_system()` 函数自行关闭或重新启动。
简而言之，任何最终调用重新启动内核标注的操作都将起作用。

从**命令行**，在 QNX guest中使用 shutdown，或在 Linux 或 Android guest中reboot 。

如果guest 自行关闭或尝试重新启动，hypervisor 将结束托管guest 的 VM 的 qvm 进程，并释放该进程使用的所有资源，其中包括guest 使用的资源(参阅[Handling a qvm termination](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/qhs/qvm_term.html))。

**主机无法查看guest，因此如果主机需要在guest关闭之前执行操作，则guest必须明确通知主机它正在关闭**。 
执行此操作的一个简单方法是：
1. 使用shared memory vdev ([vdev shmem](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/vdev_ref/vdev_shmem.html)) 设置在 guest 和 host 之间共享的一小块内存区域(参阅[Memory sharing](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/share/share_mem.html))
2. 当它关闭时，guest会将pre-defined 值写入该共享内存中的某个位置。
3. 当它在`shared memory`中读取该值时，hypervisor主机开始执行它需要完成的任何任务（例如，屏蔽对guest的中断，静默任何物理设备）以响应guest的关闭。
[Quiescing devices during guest shutdown](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/start/quiescing.html)中给出了静默物理设备的说明。

如果guest 分配有物理设备，则guest 应通知可能与之共享这些设备的任何其他guest ，它正在关闭，因此设备将不再可用。

## 外部(不可控)guest shutdown
只要有可能，请在guest 中使用受控关闭（在 QNX 客户机中`shutdown`，或在 Linux 或 Android 客户机中`reboot`）。 
如果您不能这样做，则可能会出现不受控制的guest 关闭，如此处所述。

从host 系统终端，您可以使用 `slay` 等实用程序向托管guest的 VM 的 qvm 进程传递信号（参阅[slay](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.neutrino.utilities/topic/s/slay.html)）：
- **SIGTERM** 或 **SIGINT**：

向 qvm 实例发送 SIGTERM 或 SIGINT 信号以模拟按下电源按钮。 例如：`slay -s SIGTERM qvm`。 这使得 qvm 友好地要求 guest 程序终止。

**在 x86 上**，这会触发 `ACPI power button notification mechanism`，这将允许guest在注意到它时干净地关闭。 
Linux 和 Android guest（后者如果设置正确）具有适当的 ACPI 处理并且会注意到，但 QNX guest不会。 
如果您发送 **SIGTERM** 或 **SIGINT** 但 guest 虚拟机没有关闭，qvm 会假定其中没有安装任何处理。 发送第二个信号将无条件终止访客。

**在 ARM 上**，没有电源按钮通知的标准，因此除非编写了特定于guest的 vdev，否则不会执行任何处理。 
因此，发送 **SIGTERM** 或 **SIGINT** 会立即终止guest系统，而不会发出任何通知。

- **SIGQUIT**:

向托管 qvm 实例发送 **SIGQUIT** 信号以终止它，而无需通知guest，无论其中是否进行任何断电处理。 这模拟立即切断电源。 例如：**slay -s SIGQUIT qvm**。

如果终止 VM（qvm 进程实例），则必须注意将passed through给 VM 中guest的所有硬件设备置于静止状态。 
您可以使用 vdev 来完成此操作(参阅[Virtual Device Developer's Guide](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.vdev/topic/about.html))。

> 如果您的guest以不受控制的方式终止，则该guest在后续启动时可能会以未定义的方式运行（例如，它可能具有损坏的文件系统(`corrupt filesystem`)）。

# Watchdogs
Guests 可以配置为在其托管 VM（qvm 实例）中使用`watchdog  vdev`。 
如果Guest未能及时kick 其看门狗，看门狗可能会触发 **SIGQUIT** 立即终止托管 qvm 实例。 这是使用看门狗的常见方法，但这不是唯一可能的操作(参阅[Watchdogs](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/qhs/watchdog.html))
