---
layout:     post
title:      Understanding QNX Virtual Environments
# subtitle:    "\"Hello World, Hello Blog\""
date:       2023-09-21
author:     Underdog Linux
header-img: img/post-bg-2015.jpg
catalog: true
tags:
    - QNX Hypervisor 
---

QNX hypervisors 旨在满足Popek/Goldberg 理论规定的hypervisors的期望。

# The Popek/Goldberg Theorem
Popek/Goldberg定理规定，hypervisor 应满足以下三个标准：
- Equivalence(等价)
在hypervisor中运行的虚拟机 （VM） 与底层硬件实质上相同。guest 无需知道它正在 VM 中运行。
上述声明并不排除使用半虚拟化设备(参见[Para-virtualized devices](http://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/virt/vdevs.html#vdevs__para))或其他需要虚拟化意识的策略。这些策略可用于提供功能和提高性能。
- Safety(安全)
除了guest 访问 pass-through内存之外，虚拟机管理程序始终保持对硬件的控制，而不管guest正在做什么。
它控制guest访问硬件设备的能力，将guest访问host物理内存的能力限制在分配给它们的内存区域内，对调度具有最终控制权，管理中断路由，并且能够终止guest，而不管guest可能试图做什么。

- Performance
运行在虚拟机中的程序的执行仅比直接运行在硬件上慢一点点。

> See “The Popek/Goldberg Theorem” in Edouard Bugnion, Jason Nieh and Dan Tsafrir, Hardware and Software Support for Virtualization (Morgan & Claypool, 2017).
