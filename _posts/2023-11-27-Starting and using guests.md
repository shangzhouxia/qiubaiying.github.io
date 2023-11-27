---
layout:     post
title:      Starting and using guests
# subtitle:    "\"Hello World, Hello Blog\""
date:       2023-11-27
author:     Underdog Linux
header-img: img/post-bg-2015.jpg
catalog: true
tags:
    - Booting and Shutting Down
---

以下是在hypervisor的虚拟机中引导guest的说明。

> 有关如何启动hypervisor的说明，请参阅本章中特定于主板的说明：[Booting the QNX hypervisor host](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/start/hyp_boot.html)

当hypervisor完成启动后，它就准备好托管Guest了。 
在hypervisor完成托管该guest的虚拟机的配置之前，您无法启动该guest。
您可以将hypervisor的启动活动配置为自动启动 qvm 进程，该进程将在 VM 配置完成后加载和启动 guest 虚拟机，或者在启动任何 guest 虚拟机之前等待进一步的输入。

当您可以执行[Viewing hypervisor activity](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/start/view.html)中描述的任何活动时，您就可以知道您的hypervisor已成功完成启动。

# 先决条件
要在hypervisor上运行来宾（QNX OS 或 Linux），您的目标板上需要以下各项：
- 正在运行的hypervisor
- IFS 映像 (QNX OS) 或kernel 镜像 (Linux) 以及您要运行的 guest 虚拟机
- guest的配置文件； 该文件至少指定guest的 RAM、CPU 和虚拟设备的分配，以及带有guest IFS 的文件 (IFS) 的路径和名称

除了上述内容之外，Linux guest可能还需要初始 RAM 磁盘（initrd、initramfs）文件。


# Starting a guest

> 当您启动 qvm 进程并启动 guest 虚拟机时，终端控制台将切换到您已启动的 guest 虚拟机。 您可能会发现获取hypervisor Host的 IP 地址很有用，然后在启动Guest之前通过 SSH 打开第二个终端控制台进入hypervisor Host域(参阅下面的[Determining where terminal consoles are connected](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/start/guests.html#guests__console))

要启动 QNX Guest操作系统，请在连接到hypervisor 的终端程序的命令行中：
1. 转至包含要在其中运行 guest 虚拟机的 VM 的 qvm 配置文件的目录。
2. 运行 qvm 实用程序，将其指向要运行的 guest 虚拟机的配置文件。

例如，从包含guest的目录中，对于每个guest：
- QNX Neutrino 7.1 guest
> qvm @qnx71/qnx71-guest.qvmconf
- QOS 2.2.1 guest
> qvm @qos221/qos221-guest.qvmconf
- Linux guest
> qvm @linux/linux-guest.qvmconf

qvm 命令行中文件名前面的 at 符号（“@”）指定 qvm 配置文件的路径。

当收到指令时，hypervisor 应该：
1. 为将托管guest操作系统的 VM 启动 qvm 进程的新实例。
2. 解析guest操作系统的配置文件（例如，qnx71-guest.qvmconf）。
3. 使用guest加载 IFS（例如 qnx71-guest.ifs）。
4. 启动guest操作系统。
5. 将终端程序连接到guest，以便guest操作系统启动的输出进入开发主机系统上的终端； 这是默认配置，您可以更改。


# 值得尝试的事情
为了了解您的虚拟化系统，您可以尝试以下一些操作：

## 确定终端控制台的连接位置
对于hypervisor 新手用户来说，确定哪些终端控制台连接到虚拟机管理程序主机或 QNX 和 Linux guest系统可能很困难。

但是，如果您对系统的配置方式有所了解，则可以轻松确定终端控制台显示的内容。 
运行 `uname -a` 和 `pidin info` 或 `ps` 应该会为您提供所需的信息。 
例如，连接到配置为在两个 vCPU 上运行的 QNX 来宾的终端上的 pidin info 命令将显示两个 CPU，但连接到虚拟机管理程序主机的终端上的相同命令将显示所有四个 CPU。

例子：
```
$ uname -a
Linux localhost 5.4.219-qgki-debug-g74dd2c7acc02-dirty #1 SMP PREEMPT Fri Nov 24 19:08:32 CST 2023 aarch64

# uname -a
QNX localhost 7.1.0 2023/04/17-10:08:49EDT SA8295P_v2.1_ft0_ADP_Air_v1.0.1_UFS_NORMAL aarch64le
```


hypervisor Host和 QNX Guest都有一个 `/etc/os.version` 文件。 您可以打开这些文件来查看有关Host或Guest的信息。
> 译者注：并没有看到os.version文件

## 查看线程活动
要查看有关 qvm 进程中线程活动的信息，请在连接到hypervisor 主机的终端程序的命令行中使用 `pidin`，或者使用连接到hypervisor 序主机的 QNX Momentics IDE 工具。
```markdown
# pidin
     pid tid name               prio STATE       Blocked
       1   1 /procnto-smp-instr   0f READY
       1   2 /procnto-smp-instr   0f RUNNING
       1   3 /procnto-smp-instr   0f RUNNING
       1   4 /procnto-smp-instr   0f RUNNING
       1   5 /procnto-smp-instr   0f RUNNING
       1   6 /procnto-smp-instr   0f RUNNING
       1   7 /procnto-smp-instr   0f RUNNING
       1   8 /procnto-smp-instr   0f RUNNING
       1   9 /procnto-smp-instr  10r RECEIVE     1
       1  10 /procnto-smp-instr   1r RECEIVE     2
       1  11 /procnto-smp-instr  10r CONDVAR     (0xffffff80601471e8)
       1  12 /procnto-smp-instr  10r CONDVAR     (0xffffff80601471f0)
       1  13 /procnto-smp-instr 255r RECEIVE     4
       1  14 /procnto-smp-instr  10r RECEIVE     1
       1  15 /procnto-smp-instr  11r RECEIVE     1
       1  16 /procnto-smp-instr  10r RECEIVE     1
       1  17 /procnto-smp-instr  10r RECEIVE     1
       1  18 /procnto-smp-instr  10r RECEIVE     1
       1  19 /procnto-smp-instr  10r RECEIVE     1
       1  20 /procnto-smp-instr  10r RUNNING
       1  21 /procnto-smp-instr  10r RECEIVE     1
       1  22 /procnto-smp-instr  20r RECEIVE     1
       1  24 /procnto-smp-instr  11r RECEIVE     1
    8194   1 ifs/bin/slogger2    10r RECEIVE     1
    8195   1 n/bmetrics_service  20r RECEIVE     1
```

qvm 线程活动告诉您有关 vCPU 的信息，但不告诉您每个Guest运行的各种服务和应用程序的信息。 
要从 IDE 查看Guest内的线程活动，您可以创建新的 QNX 目标并输入Guest的 IP 地址。 在这种情况下，您必须在Guest上运行 qconn 服务，因为该服务支持 IDE。 
如果您的配置未启动此实用程序，则必须手动启动它(参阅[qconn](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.neutrino.utilities/topic/q/qconn.html))


## 虚拟网路
要启动虚拟网络，请在连接到 QNX Guest的控制台的命令行上运行以下脚本：
> /scripts/network-start.sh

脚本完成后，您可以获取Guest的 IP 地址、对Guest执行 ping 操作并执行其他网络活动，例如使用 SSH 访问该Guest、在其上挂载 NFS 共享等。
您也可以在 Linux guest上执行相同的操作。


## Block 设备
要启动块设备，请在连接到 QNX Guest的控制台的命令行上运行以下脚本：
> /scripts/block-start.sh

尝试像在非虚拟化系统中使用此类设备一样使用块设备。

块设备安装到Guest上的 /RAM 磁盘路径。 
VM 配置为使用hypervisor主机上的 [devb-ram](https://www.qnx.com/developers/docs/7.1/index.html#com.qnx.doc.neutrino.utilities/topic/d/devb-ram.html) 设备。 
您可以更改托管guest虚拟机的 qvm 配置，以便guest的块设备使用其他设备，例如 USB key。
