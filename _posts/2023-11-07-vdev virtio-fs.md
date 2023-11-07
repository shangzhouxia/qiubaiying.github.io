---
layout:     post
title:      vdev virtio-fs
# subtitle:    "\"Hello World, Hello Blog\""
date:       2023-11-07
author:     Underdog Linux
header-img: img/post-bg-2015.jpg
catalog: true
tags:
    - Filesystem Sharing
---

模拟支持 FUSE 协议的虚拟文件系统。

# 语法
```h
vdev virtio-fs [directio enabled]
               [host_path dir]
               [loc addr intr guest_intr]
               [num_req_queues num]
               [sched priority]
               [tag name]
```

# 选项
**directio enabled**
是否启用direct I/O 支持。 
当此选项为 true 时，将启动 FUSE 守护程序并启用此功能。 
如果您共享的目录由资源管理器（例如 pps）管理，则可能需要启用direct I/O 才能使某些功能正常工作； 例如，阻塞读取如下所示：
> cat /data/qnxhost/services/networking/status_public?wait,delta

**host_path** dir
通过虚拟文件系统与Guest共享的Host目录。 
该目录的内容将出现在Guest选择的 `mountpoint` 下。
root 用户可以创建和共享他们想要的任何目录。 访客应用程序不需要额外的权限； 如果他们有权访问挂载点，他们就可以访问Host目录文件，就像访问本地文件一样。

**intr** guest_intr
设置该虚拟设备的Guest中断； **仅当您还包含 loc 选项时才包含此选项**。
`guest_intr` 参数由两个以冒号分隔的部分组成：
1. Guest设备可编程中断控制器 (PIC) 名称的标识符
2. 当中断设备希望引发中断时断言的输入线
例如：
> gic:43

有关设置Guest中断的更多信息，参阅 QNX Hypervisor [User's Guide](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/about.html)中的Configuring guests章节。

**loc** addr
将设备空间地址设置为addr。 addr 参数通常以十六进制指定。
如果包含此选项，则 vdev 将其自身作为位于 addr 指定位置的内存映射 I/O (MMIO) 设备呈现给 guest 虚拟机。 
否则，vdev 将自身呈现为 PCI 设备。
**您必须将此选项与 `intr` 选项一起使用**。

**num_req_queues** num
要创建的请求队列的数量。

**sched** priority
设置VM生成的脉冲的优先级以指示新输入可用。 
默认值为 10。
**大多数情况下，您可以忽略此选项**。

**tag** name
虚拟文件系统的名称。 
挂载文件系统时，Guest必须通过此名称引用它。

# 描述
（ARM 和 x86）`virtio-fs` vdev 模拟一个虚拟文件系统，其中Host目录可供Guest访问，就像它们是本地目录一样。
Guest可以访问虚拟文件系统中的文件并执行 `read()`、`write()` 和 `stat()` 等操作。 
<br />
在Guest中，vdev 的驱动程序使用 USerspace 中的文件系统 (FUSE) 协议封装文件系统请求。 
在Host中，vdev 与 FUSE 守护进程交互以执行这些请求，并将响应从该守护进程传递回Guest。

<br />
通常，vdev 在 x86 上显示为 PCI 设备。 
但如果您指定 loc 和 intr 选项，Guest将在 `loc` 指定的位置将 vdev 视为内存映射 I/O (MMIO) 设备。

您可以定义多个`virtio-fs`实例； 
每个实例可以与一个Guest共享一个Host目录。 
许多Guest可以访问相同的共享目录，但每个Guest都可以为此目录选择自己的挂载点。

# 例子
假设Host想要与Guest共享其 `/etc` 目录。 
在Guest虚拟机的配置文件中，必须配置 `virtio-fs` vdev 实例，并将 `host_path` 设置为 `/etc`：
```markdown
vdev virtio-fs
  loc 0x1c0fe000
  intr gic:56
  tag qhostfs    # Tag to use in the guest's mount command.
  num_req_queues 1
  host_path /etc # If you set this to a resource manager path (e.g., /pps),
                 # set directio to "true" below.
  directio false # Set to "true" to pass -D to the FUSE deamon.
```

`loc` 和 `intr` 选项意味着 vdev 将显示为 MMIO 设备，位于 0x1c0fe000 的Guest物理地址，并将使用 gic 的 PIC 名称和中断线 56。

有关这些选项的更多信息，请参阅 QNX Hypervisor [User's Guide](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/about.html)中的`Common vdev options`。

由于**在本例中共享的目录不受资源管理器管理，因此我们将 directio 设置为 false**。 
我们还将 `num_req_queues` 设置为 1 以仅使用一个请求队列。

vdev 配置为共享目录分配 qhostfs `tag`。
然后，Guest在为虚拟文件系统创建挂载点时必须使用此tag名称，通过该挂载点来访问该Host目录。 
请注意，Guest在定义挂载点之前可能必须创建相应的本地目录，如下所示：
```markdown
mkdir /mnt/user/10/emulated/10/Download/qnx_host
mount -t virtiofs qhostfs /mnt/user/10/emulated/10/Download/qnx_host
```
