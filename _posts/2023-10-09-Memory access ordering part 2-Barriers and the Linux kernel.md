---
layout:     post
title:      Memory access ordering part 2-Barriers and the Linux kernel
# subtitle:    "\"Hello World, Hello Blog\""
date:       2023-10-09
author:     Underdog Linux
header-img: img/post-bg-2015.jpg
catalog: true
tags:
    - Linux 
---
> 原文：[Memory access ordering part 2: Barriers and the Linux kernel](https://community.arm.com/arm-community-blogs/b/architectures-and-processors-blog/posts/memory-access-ordering-part-2---barriers-and-the-linux-kernel)

我上一篇文章介绍了内存访问排序(memory access ordering)的概念。然而，它没有为这个问题提供任何解决方案，也没有必须具体说明这种排序在哪些方面可能很重要。

现在，并非所有软件开发人员都需要深入了解内存访问顺序或barriers。除非你的代码直接与硬件交互，直接与在其他内核上执行的代码交互，或者直接加载或生成要执行的指令，否则事情大多会正常工作。如果与硬件的交互完全通过设备驱动程序 （意味着：没有设备控制寄存器直接映射到应用程序） ，则驱动程序负责强制排序。如果您与在不同内核上运行的软件的通信使用多线程 API（例如使用 Pthreads 或 Java 线程），则该 API 负责强制执行排序。如果您的程序在实现需求分页的操作系统上执行，则显然操作系统有责任强制执行此类操作的排序。

但是，如果要编写设备驱动程序、实现自己的线程通信或创建 JIT 编译器，则不知道barriers 的正确使用可能会导致意外且难以诊断的问题。如果您的程序需要特定的内存访问顺序才能被系统中的多个内核或设备看到，则解决方案称为barriers 。

虽然底层架构概念本身很有趣，但它们并不是大多数关注barriers 的软件开发人员需要了解的。出于这个原因，这篇文章只介绍了 Linux 内核中的barriers 使用。我保证在以后的帖子中回到详实的细节。

# 那么，barriers是什么？
barrier（在某些体系结构中称为fence）是显式强制实施某种类型的内存访问顺序的操作。
在更高级别，这可能意味着编译器指令阻止加载/存储操作在源代码中的一行之间重新排序，但让编译器可以自由地将内存访问重新排列在同一端。
在较低级别，这可能意味着专用指令停止在内核上执行，直到保证所有以前的内存访问对系统中的其他代理可见。代理是系统中能够启动总线事务的任何设备，例如处理器或 DMA 控制器。

> 图 1 显示了影响负载存储指令排序的barrier 示例。

![example_of_barrier.png](/img/example_of_barrier.png)


# Linux Kernel中的Barriers 
由于编译器指令、屏障指令和其他系统操作在供应商、体系结构和整体系统组件集之间会有所不同，因此 Linux 内核定义了一组需要为每个体系结构实现的可移植屏障操作。由于具有最弱内存模型（实际上是允许最多重新排序的模型）支持的体系结构是 DEC Alpha，因此将其用作参考体系结构。在这方面，没有其他架构超越DEC Alpha，但ARMv7-A非常接近。Linux 内核中可用的屏障的完整文档可以在 linux/Documentation/memory-barriers.txt 中找到，但我将在这里快速介绍一下。

## Linux barrier API
### General barrier
一般barrier没有运行时影响，它只是对编译器的指令，用于防止出于优化目的对内存访问进行重新排序。

| Statement | Description |
| --- | --- |
| barrier() | 仅编译器屏障。编译器不会将内存访问从此语句的一端重新排序到另一端。这对处理器实际执行生成的指令的顺序没有影响。 |

### Mandatory barriers
Mandatory barriers用于在整个系统级别强制实施内存一致性。最常见的例子是与外部存储器映射外设通信时。无论目标体系结构如何，所有强制屏障都保证至少扩展到编译器屏障。  

| Statement | Description |
| --- | --- |
| mb() | 完整的系统内存屏障。指令流中 mb（） 之前的所有内存操作都将在提交 mb（） 之后的任何操作之前提交。</br>此顺序对系统中的所有总线主节点都可见。它还将确保从单个处理器的访问到达从属设备的顺序。 |
| rmb() | 与 mb（） 类似，但只保证读取访问之间的顺序。也就是说，在 rmb（） 之前的所有读取操作都将在 rmb（） 之后的任何读取操作之前提交。 |
| wmb() | 与 mb（） 类似，但只保证写入访问之间的顺序。也就是说，wmb（） 之前的所有写入操作都将在 wmb（） 之后的任何写入操作之前提交。 |

### SMP conditional barriers

SMP 条件barriers用于确保缓存一致性 SMP 系统中不同内核之间的内存视图一致。在没有CONFIG_SMP的情况下编译内核时，所有 SMP 屏障都将转换为普通编译器屏障。

> Note: SMP barriers是mandatory barriers的子集，而不是超集（这是一个常见的误解）。SMP barriers不能取代mandatory barriers，但mandatory barriers可以取代 SMP barriers。


| Statement | Description |
| --- | --- |
|smp_mb()| 与 mb（） 类似，但仅保证 SMP 系统中内核/处理器之间的排序。</br>在 smp_mb（） 之前的所有内存访问都将在 smp_mb（） 之后的任何访问之前对 SMP 系统中的所有内核可见。 |
| smp_rmb()|与 smp_mb（） 类似，但仅保证读取访问之间的顺序。  |
| smp_wmb()| 与 smp_mb（） 类似，但只保证写入访问之间的排序。 |

### Implicit barriers(隐性)
内核中可用的Locking constructs充当隐式 SMP 屏障，其方式与用户空间中的 pthread 同步操作相同。当使用这些来保护共享资源时，不需要使用显式屏障（为了确保该资源的一致性）。然而，这并不能消除与外部主节点通信时对显式障碍的需求。
由于大量设备驱动程序未使用所需的屏障，因此当使用 `CONFIG_ARM_DMA_MEM_BUFFERABLE` 编译内核时，ARM 体系结构的 I/O 访问器宏（`readb（）`、`iowrite32（）` 等）充当显式内存屏障。这是在 linux-2.6.35 中添加的。

### Other barriers
Linux 内核中还有其他可用的障碍。这篇文章只涵盖了最常见的要求。有关详细信息，请参阅 Linux 内核文档。

## 使用范例
Linux 内核补丁提交指南指出`"All memory barriers {e.g., barrier(), rmb(), wmb()} need a comment in the source code that explains the logic of what they are doing and why."`。
尽管并不总是遵守这一点，但这意味着内核源代码本身可以成为使用barriers的有用参考。例如，以下内容取自
> linux/drivers/net/8139too.c：
```c
	/*
	 * Writing to TxStatus triggers a DMA transfer of the data
	 * copied to tp->tx_buf[entry] above. Use a memory barrier
	 * to make sure that the device sees the updated data.
	 */
	wmb();
	RTL_W32_F (TxStatus0 + (entry * sizeof (u32)),
		   tp->tx_flag | max(len, (unsigned int)ETH_ZLEN));
```
此代码在将某些数据写入缓冲区以移交给 DMA 引擎后执行。
wmb（） 确保在启动 DMA 事务的写入之前提交对缓冲区的写入，从而消除了数据损坏的风险。由于只需要这两个特定访问之间的排序，并且它们都是写入，因此 wmb（） 是正确的选择。请注意，此屏障也是 SMP 安全的，因为它的描述是 smp_wmb（） 功能的超集。

另外一个例子：
> linux/drivers/net/bnx2.c
```c
	/* Memory barrier necessary as speculative reads of the rx
	 * buffer can be ahead of the index in the status block
	 */
	rmb();
	while (sw_cons != hw_cons) {
```
如注释所述，在这种情况下，barrier 的主要目的是防止处理器（以及编译器）在实际进入控制块之前执行 while 循环中描述的读取访问。

# 使用barriers的代价
使用barriers 的原因是为了防止我们的工具和硬件执行不安全的优化。此外，存在不同类型的障碍，以便准确描述您需要强制执行的内存顺序。这意味着开始在任何地方插入障碍，或者在需要障碍的地方使用 mb（），可能会对软件性能产生负面影响。花额外的时间来弄清楚在特定情况下你是否真的需要一个障碍，如果是的话，它应该是哪个特定的障碍，这是非常值得的。

# Future posts
我的下一篇文章将介绍 ARM 体系结构中可用的barrier 指令和操作。
[Read Part3](https://community.arm.com/processors/b/blog/posts/memory-access-ordering-part-3---memory-access-ordering-in-the-arm-architecture)











