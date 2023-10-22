---
layout:     post
title:      配置hypervisor host
# subtitle:    "\"Hello World, Hello Blog\""
date:       2023-10-19
author:     Underdog Linux
header-img: img/post-bg-2015.jpg
catalog: true
tags:
    - QNX Hypervisor Configuration
---

hypervisor 主机域通过构建文件（`buildfile`）进行配置，与 QNX OS image完全相同。

有关如何使用 QNX 构建文件的详细说明，包括构建文件结构、内容和语法，请参阅 《QNX SDP Building Embedded Systems guide》的`OS Image Buildfiles`一章。

hypervisor 主机的构建文件与标准 QNX OS images的构建文件在以下部分有所不同：

### 虚拟化扩展模块
包含有关包含虚拟化扩展模块 (`libmod_qvm.a`) 的说明。 
添加此模块就像添加自适应分区 (APS: `adaptive partitioning`) 模块一样。 
例如，受支持的 ARM 平台的hypervisor 主机的构建文件可能如下开始：

```h
[+compress]
[image=0x40100000]
[uid=0 gid=0]
[virtual=aarch64le,raw] boot = {
  [+keeplinked] startup-rcar_h3 -P1 -W -vv
  [+keeplinked module=qvm module=aps] PATH=/proc/boot:/sbin:/bin:/usr/bin:/opt/bin/sbin:/usr/sbin
  LD_LIBRARY_PATH=/proc/boot:/lib:/usr/lib:/lib/dll:/opt/lib procnto-smp-instr -ae -v
}
```

此示例中特别令人感兴趣的是 `module=qvm`，它将虚拟化模块带入构建中。
> 使用 QHS 时，您必须通过添加 [module=qvm-safety] 而不是 [module=qvm] 来指定您想要内核模块的安全变体。<br />
更多信息，参考 QNX Hypervisor for Safety 2.2 Safety Manual

### Host domain code
包括包含和启动 qvm 的指令，以及将在hypervisor 主机域中运行的其他代码。 
例如，以下包含并启动 qvm 进程实例并组装 VM：
```c
qvm
```

### VM (qvm) configuration files
包括 qvm 进程的不同实例在创建和组装 VM 时应使用的 qvm 配置文件的路径，并指示在映像中放置这些配置文件的位置。 
例如：

```c
/vm/config/qnx71.qvmconf = guests/qnx71/qnx71.qvmconf
```

### Virtual device (vdev) shared objects
在共享对象列表中包含虚拟设备模块 (vdev-*.so)。 
例如：
```c
vdev-8259.so
vdev-ser8250.so
vdev-timer8254.so
vdev-mc146818.so
vdev-virtio-console.so
vdev-virtio-blk.so
vdev-virtio-net.so
vdev-shmem.so
vdev-pckeyboard.so
vdev-ioapic.so
vdev-pci-dummy.so
vdev-hpet.so
vdev-pl011.so
vdev-vgpu-gvtg.so
vdev-progress.so
```

### 未使用的组件移除
构建文件应仅包含 hypervisor 主机所需的组件。

您应该删除所有不需要的组件，以便它们不会包含在hypervisor 主机映像中。 
例如，如果将设备（例如 USB）传递给guest，则hypervisor 主机（或另一个guest）可能无法访问该设备，因此从hypervisor 主机构建文件中删除驱动程序。
（参考`VM Configuration Reference`章节的[pass](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/vm/pass.html)）

# Safety variants for the QNX Hypervisor for Safety (QHS)
> 当您使用 QHS 时，您必须指定您需要内核模块和 vdev 的安全变体。 在您的主机构建文件中：<br />
> - 对于内核模块安全变体，请使用 [module=qvm-safety] 而不是 [module=qvm]。
> - 使用 vdev-*-safety.so 作为每个 vdev 的安全变体； 例如，vdev-foo-safety.so。
> - 使用您的主板所需的 smmuman-*-safety 文件(参考 SMMUMAN User's Guide的[Installation](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.smmuman.user/topic/use/install.html))
> <br /> 更多信息参考 《QNX Hypervisor for Safety 2.2 Safety Manual》

# Safety variant support for PCI pass-through devices
如果您的hypervisor 系统将pass through PCI 设备，则必须在hypervisor 主机中包含 `pci_server-qvm_support.so` 模块。
有关说明，请参阅《QNX SDP Utilities Reference》中的 [pci-server](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.neutrino.utilities/topic/p/pci-server.html)。
