---
layout:     post
title:      Shutting down the QNX hypervisor
# subtitle:    "\"Hello World, Hello Blog\""
date:       2023-11-27
author:     Underdog Linux
header-img: img/post-bg-2015.jpg
catalog: true
tags:
    - Booting and Shutting Down
---

可以使用以下方法关闭 QNX hypervisor：
1. 如果 QNX hypervisor遇到 `undefined state`，它将转至`Design Safe State`(参阅[QNX Hypervisor: Protection Features](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/qhs/qhs.html)章节的[Design Safe States](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/qhs/dss.html))
2. 所有可用于关闭或重新启动 QNX 操作系统的机制（例如 `slay -s SIGTERM qvm`）均可供 QNX hypervisor使用。 
这些机制适用于在hypervisor 主机域中运行的应用程序，但不适用于Guest或其应用程序。

有关如何关闭hypervisor guest的说明，参阅[Shutting down guests](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/start/guest_stop.html)
