---
layout:     post
title:      Virtual machines
# subtitle:    "\"Hello World, Hello Blog\""
date:       2023-09-28
author:     Underdog Linux
header-img: img/post-bg-2015.jpg
catalog: true
tags:
    - QNX Hypervisor 
---

正在运行的hypervisor包括hypervisor微内核及其虚拟化模块，以及虚拟机进程（qvm）的一个或多个实例。

# 什么是虚拟机器
在QNX hypervisor环境中，VM在qvm进程实例中实现。qvm进程是一个操作系统进程，在内核之外的 hypervisor 主机中运行。每个实例都有一个标记它的标识符，以便微内核知道它是一个qvm进程。

如果您还记得有关VM的任何信息，请记住，从guest操作系统的角度来看，承载该guest 的VM是硬件。
这意味着，正如在物理板上运行的操作系统期望某些硬件特性（架构、板细节、内存和CPU、设备等）一样，在VM中运行的操作系统也期望这些特性：在其中运行的VM必须符合guest的期望。

配置VM时，您正在组装硬件平台。不同之处在于，您不需要组装物理内存卡、CPU等，而是指定机器的虚拟组件，qvm进程将根据您的规范创建和配置这些组件。

关于事物出现位置的规则与真实硬件的规则相同：
- 不要安装两个试图响应同一物理地址的东西。
- 您的VM配置集合的环境必须是您将运行的软件（ guest 操作系统）准备好处理的环境。

硬件类比也适用于另一个方向。VM不需要知道其客户在做什么，就像硬件需要知道OS在做什么一样。事实上，虚拟机不知道guest在做什么(“the guest is a blob”; 参考 [Terminology](http://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/apx/terminology.html) 章节)。

有关配置VM的更多信息，参考[Configuration](http://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/config/config.html)章节的[Assembling and configuring VMs](http://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/config/qvm.html)。

# qvm服务
每个qvm流程实例都提供关键的hypervisor服务。
## VM 组装和配置
为了创建guest OS 可以运行的虚拟环境，qvm进程实例在启动时执行以下操作：
- 读取、解析和验证VM配置文件（==*.qvmconf==）以及在启动时使用进程命令行输入的配置信息，如果配置无效则退出，并将有意义的错误消息打印到日志中。
- 设置中间阶段表（ intermediate stage tables ）（ARM:阶段2页表，x86:扩展页表（EPT））。
- 创建（组装）并配置其VM，包括：
    - 为guests分配RAM（r/w）和ROM（read only）
    - 给guests的每个虚拟CPU（vCPU）提供线程
    - pass -through device给guest
    - 为guest定义和配置虚拟设备（vdevs）

## VM operation
在VM操作期间，qvm进程实例执行以下操作：
- 捕获出站和入站访客访问尝试并确定如何处理它们（即，如果地址是vdev，则调用vdev代码（访客退出，然后在vdev代码完成时访客进入）；如果地址确实超出了界限，那么就这样处理它）。
- 在放弃物理CPU之前保存其guest 的上下文。
- 在将guest 重新执行之前恢复其guest 的上下文。
- 负责任何故障处理。
- 执行确保VM完整性所需的任何维护活动。

# Guest 启动和关机
VM中的guest可以像在硬件上一样启动。从guest的角度来看，它开始在物理CPU上执行。然而，这个CPU实际上是一个qvm vCPU线程。
guest可以启用中断，就像在非虚拟化系统中运行一样。

当 guest's VM中的第一个vCPU线程开始执行时，VM可以知道该guest 已启动。
guest应负责初始化guest 关机。qvm进程检测通过PSCI或ACPI等常用方法启动的关机。如果您想使用qvm无法自动识别的方法，可以编写一个vdev来检测关机操作并做出适当的响应。

有关编写 vdev 的详细信息，参考[Virtual Device Developer's Guide ](http://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.vdev/topic/about.html)。

> 如果hypervisor检测到VM中存在未定义的条件，则hypervisor 将终止该VM的qvm进程实例，该实例将终止guest。
参考 [Design Safe States](http://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/qhs/dss.html)章节的[QNX Hypervisor: Protection Features](http://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/qhs/qhs.html)

# 管理 guest 上下文
在虚拟化环境中，CPU 虚拟化扩展负责从guest's 的操作中识别guest何时需要退出。
但是，当 CPU 触发guest 退出时，托管guest 的 qvm 进程实例（即 VM）会保存guest 的上下文。
qvm 进程实例完成guest启动的操作，然后在允许guest重新进入之前恢复guest的上下文
（请参阅“[Performance Tuning](http://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/perform/perform.html)”一章中的“[Guest exits](http://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/perform/guest_exits.html)”）。

# 管理权限级别
qvm进程实例管理来宾entrances and exits的权限级别，以确保guests 可以运行，并保护系统不受错误代码的影响。

在guest 入口，qvm进程实例要求CPU为guest 提供运行所需的特权级别，但不能超过此级别。
在guest 退出时，qvm进程实例要求CPU返回到在hypervisor 主机中运行所需的特权级别。
只有CPU硬件可以更改权限级别。
qvm进程执行CPU更改权限级别所需的操作。
该机制（因此是操作）是特定于体系结构的。

# guest访问虚拟和物理资源
每个qvm进程实例都管理其guest对虚拟资源和物理资源的访问。
当guest 尝试访问其guest 物理内存中的地址时，这种访问可以是以下情况之一，托管guest 的qvm进程会检查访问尝试并按所述进行响应：
- **Permitted**
guest 正在尝试访问其拥有的内存区域。
qvm进程实例不做任何事情。
- **Pass-through device**
guest 正在尝试访问分配给物理设备的内存，并且来宾的VM被配置为知道guest 可以直接访问该设备。
qvm进程实例不做任何事情。guest 直接与设备通信。
- **Virtual device**
guest 正在尝试访问分配给虚拟设备（虚拟或准虚拟）的地址。
qvm进程实例请求适当的特权级别更改，并将执行传递给所请求设备的代码。
例如，guest CPUID请求触发qvm进程实例来模拟硬件并响应来宾，与硬件在==非虚拟化系统中==的响应完全相同。
- **Fault**
guest 正在尝试访问其无权访问的内存。
qvm进程实例向guest 返回适当的错误。

以上是针对通过 CPU 的访问尝试。DMA 访问控制由 SMMU 管理器管理（参考[DMA device containment (smmuman)](http://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/virt/mem.html#mem__dma)）
