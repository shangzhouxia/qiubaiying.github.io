---
layout:     post
title:      Building the host
# subtitle:    "\"Hello World, Hello Blog\""
date:       2023-10-29
author:     Underdog Linux
header-img: img/post-bg-2015.jpg
catalog: true
tags:
    - Building a QNX Hypervisor system
---

下载hypervisor 包并配置构建环境后，构建hypervisor 主机所需的所有组件都应位于开发主机系统上的 `$QNX_TARGET` 上。

# 配置构建文件并构建镜像
1. 如果您需要hypervisor 包中提供的 BSP 以外的 BSP，请将 BSP 下载到开发主机上的工作区(参阅[Downloading a board-specific host BSP](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/build/downloads.html#downloads__board))
2. 根据需要，修改 BSP 构建文件以包含您需要的文件。 这些包括：
- `libmod_qvm.a `内核模块，需要在包含 procnto* 的构建文件行中添加 `[module=qvm]`前缀。 例如，更改：
```h
LD_LIBRARY_PATH=/usr/lib:/lib:/lib/dll procnto-smp-instr
```
    为
```h
[module=qvm] LD_LIBRARY_PATH=/usr/lib:/lib:/lib/dll procnto-smp-instr
```

```
使用 QHS 时，您必须通过添加` [module=qvm-safety]` 而不是 `[module=qvm] `来指定您想要内核模块的安全变体。
添加 vdev 和 SMMUMAN 服务时，也适用使用 -safety 变体的相同要求。
更多信息，参阅 QNX Hypervisor for Safety 2.2 Safety Manual.
```

- hypervisor 文件：qvm 以及（对于 x86，可选）qvm-check 实用程序。 例如：
```
/sbin/qvm = qvm
/bin/qvm-check = qvm-check
```

- 您想要为guest提供的每个 vdev 的 vdev-*.so 文件。 例如：
```
vdev-8259-safety.so
vdev-hpet-safety.so
vdev-ioapic-safety.so
...
vdev-virtio-console-safety.so
vdev-virtio-input-safety.so
vdev-virtio-net-safety.so
```

- smmuman 服务、架构和板支持库以及配置文件。 例如，对于受支持的 Renesas R-Car H3 主板上的 QNX hypervisor 系统：
```
/bin/smmuman-safety = smmuman-safety
/bin/smmu-rcar3-safety.so = smmu-rcar3-safety.so
/etc/smmuman/rcar-h3-safety.smmu = ./smmuman-config/rcar-h3-safety.smmu
```

3. 如果您将使用 `shmem-host` 共享内存示例应用程序进行测试，请下载 HHBSP(参阅[The HHBSP framework](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/build/hhbsp.html))，并将文件从 HHBSP 的 prebuilt/ 目录复制到 BSP 的 prebuilt/ 目录。

4. 包含您将在hypervisor 系统中运行的 VM 的 VM 配置文件 (`*.qvmconf`)。
5. 从 BSP 的根目录运行 make 来构建主机 IFS。

如果主机构建成功，您现在可以构建guest（即 QNX guest的 IFS，以及 Linux guest的内核、initrd 文件、磁盘等）并将主机和guest组装成可以传输到您的目标的映像。

