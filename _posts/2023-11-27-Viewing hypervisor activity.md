---
layout:     post
title:      Viewing hypervisor activity
# subtitle:    "\"Hello World, Hello Blog\""
date:       2023-11-27
author:     Underdog Linux
header-img: img/post-bg-2015.jpg
catalog: true
tags:
    - Booting and Shutting Down
---

成功启动hypervisor后，您应该能够使用终端工具来查看hypervisor正在执行的操作。 
例如，如果您在串行终端的命令行中输入 `pidin info`，您可能会看到“Release QNX 7.1”，并列出处理器信息。

```markdown
# pidin info
CPU:AARCH64 Release:7.1.0  FreeMem:12GB/23GB BootTime:Jan 01 00:00:01 GMT 1970
 Actual resident free memory:12Gb
Processes: 109, Threads: 1116
Processor1: 1091556528 Octa Kryo-695 Gold 2092MHz FPU
Processor2: 1091556528 Octa Kryo-695 Gold 2092MHz FPU
Processor3: 1091556528 Octa Kryo-695 Gold 2092MHz FPU
Processor4: 1091556528 Octa Kryo-695 Gold 2092MHz FPU
Processor5: 1091556544 Octa Kryo-695 Gold Plus 2380MHz FPU
Processor6: 1091556544 Octa Kryo-695 Gold Plus 2380MHz FPU
Processor7: 1091556544 Octa Kryo-695 Gold Plus 2380MHz FPU
Processor8: 1091556544 Octa Kryo-695 Gold Plus 2380MHz FPU
```

如果您的hypervisor 主机连接到 DHCP 服务器，您可以获取目标板的 IP 地址，如下所示：
1. 命令行输入 `ifconfig`
    该命令应返回 wm0 设备的有效 IP 地址。
2. 记下这个地址。 它是hypervisor 的 IP 地址。 
    当您从开发主机连接到目标上的hypervisor 时，您需要输入它。

当您获得hypervisor 的 IP 地址后，您可以使用 IDE 连接到hypervisor ：
1. 在开发主机上启动 QNX Momentics IDE（有关执行此操作的说明，请参阅 `QNX Momentics IDE User's Guide`和 `IDE Release Notes`）。
2. 切换到 QNX System Information perspective。
3. 右键单击 `Target Navigator` 视图，然后选择 `New QNX Target`。
4. 在出现的对话框中，输入hypervisor 的 IP 地址，然后按 Enter。

> 您需要在目标上运行 `qconn` 守护进程(参阅[qconn](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.neutrino.utilities/topic/q/qconn.html))

您现在应该能够使用 IDE 工具来处理目标上运行的hypervisor 。
以下是在 IDE 中可以尝试的一些操作：
- 打开`Target File System Navigator`视图以探索目标上的文件系统。 您应该在 `/dev/shmem` 中看到内置 RAM 磁盘； 这是**放置临时文件的方便位置**。
- 打开`System Information`和`Process Information`视图以查看有关目标上运行的进程的详细信息。
- 打开`System Resources`视图可查看目标上所有内核的总 CPU 使用率（尚未提供每个内核的使用率）。
- 启动另一个 shell 窗口； 首选 SSH：打开`Target Navigator`视图，选择您的目标，然后右键单击以选择 SSH。

您还可以在另一个终端程序（例如 PuTTY）中启动另一个 SSH 会话，并以`userid root, password root`身份登录。 您可以稍后在修改hypervisor bootable 镜像时更改这些内容。

您现在应该准备好启动hypervisor的guest(参阅[Starting and using guests](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/start/guests.html))。
