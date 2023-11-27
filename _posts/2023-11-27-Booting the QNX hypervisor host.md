---
layout:     post
title:      Booting the QNX hypervisor host
# subtitle:    "\"Hello World, Hello Blog\""
date:       2023-11-27
author:     Underdog Linux
header-img: img/post-bg-2015.jpg
catalog: true
tags:
    - Booting and Shutting Down
---

您可以启动并运行 QNX hypervisor Host，而无需启动任何guests。

```markdown
许多主板供应商为其 SoC 解决方案提供单独的安装指南和用户指南，解释如何在其主板上启动和运行hypervisor 。
本用户指南中的信息适用于使用此处描述的hypervisor 版本的所有供应商解决方案，除非提供有关选项的限制或建议。
```

hypervisor 启动后，您可以查看其活动(查看[Viewing hypervisor activity](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/start/view.html)),
然后启动guest(参阅[Starting and using guests](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/start/guests.html))。
