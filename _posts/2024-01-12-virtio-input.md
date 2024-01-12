---
layout:     post
title:      vdev virtio-input
# subtitle:    "\"Hello World, Hello Blog\""
date:       2024-01-12
author:     Underdog Linux
header-img: img/post-bg-2015.jpg
catalog: true
tags:
    - Input Sharing
---

为输入设备创建接口

# 语法
```markdown
vdev virtio-input [display display_id]
                  [force_dev_presence enabled]
                  [global_alpha number]
                  [loc addr intr guest_intr]
                  [offset x_offset,y_offset]
                  [options [+|-]bitmask]
                  [pipeline number]
                  [protocol b]
                  [sched priority]
                  [size x_dimension,y_dimension]
                  [window window_id]
                  [zorder zorder]
                  screen input_type
```

> 一个例子
```markdown
vdev vdev-virtio-input.so
loc 0x1c240000
intr gic:43
size 1500,1640
screen multi-touch
display 4
protocol b
options +0x80
```

# 选项
### display display_id
在指定的显示器上创建一个内部透明窗口，以捕获来自该窗口的所有输入事件并将它们转发给Guest。

如果定义了`window`选项，则忽略此选项，因为在这种情况下使用外部窗口。 如果您不提供`display`或`window`，则默认情况下会在默认显示上创建一个内部窗口。

### force_dev_presence enabled
如果在不存在物理设备时需要初始化前端，则指定 true。 
如果未指定，则默认值为 false。 目前，仅键盘(keyboard)设备支持此功能。

### global_alpha number
指定内部透明窗口的全局 Alpha (SCREEN_PROPERTY_GLOBAL_ALPHA)。 如果定义了`window`选项，则忽略此选项，因为在这种情况下使用外部窗口。

### intr guest_intr
设置该虚拟设备的 Guest 中断； **仅当您还包含 loc 选项时**才包含此选项。
guest_intr 参数由两个以冒号分隔的部分组成：
1. guest设备可编程中断控制器 (PIC) 名称的标识符
2. 当中断设备希望引发中断时 asserted 的输入线

有关设置访客中断的更多信息，参阅QNX Hypervisor [User's Guide](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/about.html)的"Configuring guests"部分。

### loc addr
将设备空间地址设置为addr。 addr 参数通常以十六进制指定。

如果包含此选项，则 vdev 将其自身作为位于 addr 指定位置的**内存映射 I/O (MMIO) 设备呈现给 guest 虚拟机**。 
否则，vdev 将自身呈现为 PCI 设备。

您**必须将此选项与 intr 选项一起使用**。


### offset x_offset,y_offset
分别通过 x_offset 和 y_offset 调整所有事件的 x 和 y 坐标。 两个参数均以像素为单位指定。

Guest将在调整后的坐标（x+x_offset、y+y_offset）处看到事件。

**大多数情况下，您不需要使用此选项**。 
但是，如果Guest窗口的显示坐标与硬件显示器的显示坐标存在偏移，则它可能很有用。

### options [+|-]bitmask
应用选项的指定位掩码。 

可选前缀 + 和 - 添加或清除默认位掩码中的位。 
例如，如果您指定选项 +0x80，则会设置位 7 (VI_DEV_SENSITIVITY_CONTINUE)（如果尚未设置）。 
同样，如果指定选项 -0x80，则将取消设置位 7（如果尚未取消设置）。 
如果两个前缀均未使用（例如，选项 0x80），则选项将设置为位掩码 (0x80)。

默认情况下，选项设置为 0x12。 
其余位根据设备设置。 
例如，如果 `virtio-input` vdev 将自身显示为鼠标（请参阅[screen](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.qavf.overview/topic/input_config_vdev.html#input_config_vdev__screen)选项），则它会在收到 SCREEN_EVENT_POINTER 事件时设置 VI_DEV_POINTER_AS_MOUSE 位。

大多数情况下，您不需要使用 options 选项。

| Option | Bitmask |
| --- | --- |
| VI_DEV_EVSYN_MULTIPLE | 0x0001 |
| VI_DEV_EVSYN_ALWAYS | 0x0002 |
| VI_DEV_POINTER_AS_MOUSE | 0x0004 |
| VI_DEV_POINTER_HAS_WHEEL | 0x0008 |
| VI_DEV_0FILL (where 0 is zero) | 0x0010 |
| VI_DEV_TOUCH_ST | 0x0020 |
| VI_DEV_TOUCH_MT | 0x0040 |
| VI_DEV_SENSITIVITY_CONTINUE | 0x0080 |
| VI_DEV_ALL_OPTIONS | 0x00ff |

### pipeline number
指定内部透明窗口的管道(SCREEN_PROPERTY_PIPELINE)。 
如果定义了`window`选项，则忽略此选项，因为在这种情况下使用外部窗口。

### protocol b
将协议从类型 A 更改为类型 B。默认协议为类型 A。

此选项使 vdev 使用支持 B 型设备的multi-touch (MT) 协议的一部分，这些设备能够跟踪可识别的连接。 在这种情况下，协议描述了如何通过事件槽发送各个连接的更新。

更多信息，参阅:https://www.kernel.org/doc/Documentation/input/multi-touch-protocol.txt

```markdown
默认情况下，版本 11 之前的早期 Android 版本不支持 B 类设备。 要在早期版本中支持这些设备，您必须应用以下内核补丁:
 https://github.com/torvalds/linux/commit/16c10bede8b3d8594279752bf53153491f3f944f
 
对于后续的 Android 版本，不需要此补丁。
```

### sched priority
设置VM生成的脉冲的优先级以指示新输入可用。 默认值为 10。
大多数情况下，您可以忽略此选项。

### screen input_type
设置设备输入类型。 此**选项是必需的**。
此选项告诉虚拟设备如何向Guest驱动程序呈现自己。 您可以指定以下值之一：
- keyboard
显示为键盘设备。

- mouse|pointer
显示为带有按钮的指点设备。
指定`mouse`告诉虚拟设备发出相对坐标； 
指定`pointer`告诉它发出绝对值。

- touch|touchscreen
- single-touch|multi-touch
显示为触摸屏设备。

touch 和 touchscreen 参数是别名； 任何一种形式都会告诉虚拟设备显示为触摸屏。

如果指定`single-touch`，则虚拟设备仅捕获每个多点触摸事件的第一个触摸点；
如果指定`multi-touch`，虚拟设备将捕获每个多点触摸事件的所有触摸点，最多 5 个点。

如果您未指定这两个参数，虚拟设备将根据报告的最大触摸点数量自动选择协议。

以下是如何将虚拟设备宣传为 multitouch 设备的示例：

```markdown
screen touch
screen multi-touch
```

- 啥也不配置
捕获所有输入事件。 访客将看到所有支持的输入设备并接收与其相关的事件。

### size x_dimension,y_dimension
将绝对坐标设备（指针或触摸屏）的物理尺寸公布为 x_dimension 和 y_dimension。 两个参数均以像素为单位指定。

### window window_id
从指定的**外部窗口**读取所有输入事件并将它们转发给Guest。

`window_id` 参数采用要从中读取事件的窗口的 SCREEN_PROPERTY_ID_STRING。 请参阅[Screen Developer's Guide](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.screen/topic/manual/cscreen_about.html)以了解有关此字符串以及如何检索它的更多信息。


如果省略此选项，vdev 将创建一个全屏输入区域，从该区域读取所有输入事件，然后将它们转发给guest 。

### zorder zorder
指定内部透明窗口的 z 顺序 (SCREEN_PROPERTY_ZORDER)。 此窗口的默认值为 1000。

如果定义了`window`，则忽略此选项，因为在这种情况下使用外部窗口。

# 描述
（ARM 和 x86）input vdev 为输入设备创建接口。 
它支持 USB 键盘、USB 鼠标和触摸输入设备。

通常，vdev 在 x86 上显示为 PCI 设备。 
但**如果您指定 loc 和 intr 选项**，guest 将在 loc 指定的位置将 vdev 视为内存映射 I/O (MMIO) 设备。

您可以在 VM 配置中指定最多三个输入设备，其中每个 vdev 将自身呈现为Guest的不同设备。
参阅[Configuration examples](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.qavf.overview/topic/input_config_examples.html)获取更多信息。

> 您的主机系统必须至少连接一个物理输入设备。 vdev 希望找到一个输入设备； 如果没有，vdev 将阻止 qvm 进程启动。

