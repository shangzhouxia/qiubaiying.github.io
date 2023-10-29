---
layout:     post
title:      Getting the host components
# subtitle:    "\"Hello World, Hello Blog\""
date:       2023-10-29
author:     Underdog Linux
header-img: img/post-bg-2015.jpg
catalog: true
tags:
    - Building a QNX Hypervisor system
---

构建hypervisor 系统所需的组件可从 `QNX Software Center`获取。

假设您的系统上安装了正确的 QNX 构建环境(参阅本章节的[Supported build environments](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/build/build_env.html)),
您只需从 QNX 软件中心获取一个软件包即可构建和运行hypervisor 映像。 
如果您想要修改特定于板的驱动程序和 vdev 等组件，您可以获取其他软件包。 请参阅以下部分：
- [Downloading the QNX hypervisor product package](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/build/downloads.html#downloads__host)
- [Downloading a board-specific host BSP](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/build/downloads.html#downloads__board)(可选)

当您下载这些软件包时，QNX 软件中心会将它们放置在您指定的安装根目录中（例如` ~/qnx710/`）。

> 您不需要guest BSP 即可仅构建hypervisor 主机。 您可以立即下载它们，也可以稍后在准备好构建 QNX guest时下载它们(参阅[Building guests](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/build/build_guest.html))
>  如果您要使用 HHBSP，则还需要下载它(参阅[The HHBSP framework](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/build/hhbsp.html))

# Downloading the QNX hypervisor product package
QNX hypervisor 产品包包含构建hypervisor 所需的所有组件。 该软件包可从QNX软件中心获取，如下图所示：
![qvm_hyp_pkg.png](/img/qvm_hyp_pkg.png)

假设您不更改默认设置，当您下载软件包时，QNX 软件中心会将 QNX hypervisor 安装到您指定下载的安装根目录内的目录中； <br />例如，`~/qnx710/target/qnx710/ (Linux) `或` C:\Users\userid\qnx710 (Windows)`。

# Downloading a board-specific host BSP
您从 QNX 软件中心下载的hypervisor 包包含hypervisor 支持的硬件平台的 BSP。
但是，如果您要修改hypervisor 组件，则可能需要获取额外的 BSP。 下图显示了瑞萨 R-Car H3 BSP 的封装。
![qvm_bsp_board.png](/img/qvm_bsp_board.png)

要获取特定于板的 BSP，只需下载适当的软件包并将其解压到方便的位置即可。

> BSP User's Guides 可从 QNX 下载中心的 [QNX SDP 7.1 Board Support Documentation](http://www.qnx.com/download/group.html?programid=48507)获取。
