---
layout:     post
title:      Starting VMs
# subtitle:    "\"Hello World, Hello Blog\""
date:       2023-11-27
author:     Underdog Linux
header-img: img/post-bg-2015.jpg
catalog: true
tags:
    - Booting and Shutting Down
---

启动 VM 时，您可以在命令行中输入所有配置信息，也可以将 qvm 进程指向配置文件。

# 使用配置文件启动虚拟机
启动 VM 的**推荐方法**是将 qvm 进程指向 qvm 配置文件。 
在启动 qvm 进程之前，您必须首先导航到包含要使用的配置文件的目录，如下所示：

```markdown
% cd /guests/qnx-guest-1/

% qvm @config
```
其中 config 是当前目录中配置文件的名称（例如 qnx71.qvmconf）。

qvm进程打开文件并解析其内容，其中定义了VM(参阅[Assembling and configuring VMs](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/config/qvm.html))

```markdown
如果您不从包含配置文件的目录中启动 qvm 进程，您将收到一条错误消息，指出IFS can't be found。 
这种要求您位于包含目录中的设计背后的原因是，您可以移动guest系统并仍然使用相同的命令启动它，并在 *.qvmconf 文件中使用相对路径进行加载，这减少了对此的维护 文件。

qvm 命令行指令中文件名前面的 at 符号（“@”）指定一个文件为 qvm 配置文件。
```

# 启动没有配置文件的虚拟机
不需要 qvm 配置文件； 这些文件很方便，值得推荐，特别是对于复杂的虚拟机。 
但是，您可以在没有配置文件的情况下启动 qvm 流程实例。 只需在命令行中输入配置信息即可。 
例如，以下命令在 ARM 平台上启动并配置具有 PL1011 vdev 的 Linux Guest：

```markdown
qvm cpu sched 8 ram 0x80000000,128m load /vm/images/linux.img \
cmdline "console=ttyAMA0 earlycon=pl011,0x1c090000 debug user_debug=31 loglevel=9" \
initrd load /vm/images/ramdisk.img \
vdev pl011 loc 0x1c090000 intr gic:37 hostdev /dev/ser2
```
通过命令行输入的 qvm 进程的配置信息使用与 qvm 配置文件相同的语法。

上面显示的命令行配置中未指定`system`选项，因为此配置用于测试临时配置。
