---
layout:     post
title:      Quiescing devices during guest shutdown
# subtitle:    "\"Hello World, Hello Blog\""
date:       2023-12-07
author:     Underdog Linux
header-img: img/post-bg-2015.jpg
catalog: true
tags:
    - Booting and Shutting Down
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
