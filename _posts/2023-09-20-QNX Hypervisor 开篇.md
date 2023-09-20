---
layout:     post
title:      About This Document
# subtitle:    "\"Hello World, Hello Blog\""
date:       2023-09-20
author:     Underdog Linux
header-img: img/post-bg-2015.jpg
catalog: true
tags:
    - QNX Hypervisor 
---

本用户指南介绍了 QNX Hypervisor 架构，并提供了有关安装和运行 QNX Hypervisor 系统、更改系统组件和配置以及使用虚拟设备 (vdev) 等hypervisor功能的说明。
有关如何构建您自己的 vdev 的信息，参见[Virtual Device Developer's Guide ](http://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.vdev/topic/about.html)和[Virtual Device Developer's API Reference](http://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.vdev.api/topic/manual/about.html)。

## QNX Hypervisor (QH)

> QNX hypervisors 有两种版本：QNX Hypervisor 和 QNX Hypervisor for Safety。
> QNX Hypervisor 变体 (QH)（包括 QNX Hypervisor 2.2）不是经过安全认证的产品。 不得用于安全相关生产系统。
> 如果您正在构建安全相关系统，则必须使用已构建并批准用于您正在构建的系统类型的 QNX Hypervisor for Safety (QHS) 变体，并且只能按照其安全手册中的规定使用它 。 当前的 QHS 版本是 QNX Hypervisor for Safety 2.2。
> 请注意，本指南中提供的有关 QHS 和其他安全相关组件的信息仅供您参考。 请参阅相应的安全手册（例如，QNX OS for Safety 2.2.1 安全手册、QNX Hypervisor for Safety 2.2 安全手册），获取有关如何配置、启动和使用 QNX 安全相关组件的权威说明。

## 本指南的内容

| 要了解                                   | 参考                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| ------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| QNX 虚拟环境，包括 QNX hypervisor 系统的架构      | [Understanding QNX Virtual Environments](http://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/virt/virt.html)                                                                                                                                                                                                                                                                                                                            |
| QNX hypervisor使用的保护功能                 | [QNX Hypervisor: Protection Features](http://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/qhs/qhs.html)                                                                                                                                                                                                                                                                                                                                 |
| QNX hypervisor系统中的物理和虚拟设备             | [Devices](http://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/virt/devs.html) ， [Virtual devices](http://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/virt/vdevs.html)， [Physical devices](http://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/virt/pdevs.html) 和 [Virtual Device Reference](http://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/vdev_ref/vdev_ref.html) |
| 构建并启动hypervisor 及其guests              | [Building a QNX Hypervisor system](http://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/build/build.html)                                                                                                                                                                                                                                                                                                                                |
| 启动和停止虚拟机管理程序                          | [Booting the QNX hypervisor host]() 和 [Shutting down the QNX hypervisor](http://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/start/hyp_stop.html)                                                                                                                                                                                                                                                                                       |
| 配置hypervisor 主机域、虚拟机 (VM) 和guests     | [Configuration](http://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/config/config.html)                                                                                                                                                                                                                                                                                                                                                 |
| 创建和配置 VM                              | [Assembling and configuring VMs](http://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/config/qvm.html)和[VM Configuration Reference](http://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/vm/vm.html)                                                                                                                                                                                                                 |
| hypervisor 系统中的网络                     | [Networking](http://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/network/network.html)                                                                                                                                                                                                                                                                                                                                                  |
| guests之间以及guests和hypervisor 主机之间的内存共享 | [Memory sharing](http://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/share/share_mem.html)                                                                                                                                                                                                                                                                                                                                              |
| 调试hypervisor                          | [Monitoring and Troubleshooting](http://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/debug/debug.html)                                                                                                                                                                                                                                                                                                                                  |
| 调优hypervisor 以获取最佳性能                  | [Performance Tuning](http://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/perform/perform.html)                                                                                                                                                                                                                                                                                                                                          |
| 随 QNX hypervisor 提供的实用程序和驱动程序         | [Utilities and Drivers Reference](http://www.qnx.com/developers/docs/7.1/com.qnx.doc.hypervisor.user/topic/utils/utils.html)                                                                                                                                                                                                                                                                                                                                 |

## 非hypervisor 组件

本用户指南包含并标识了一些 QNX Hypervisor 用户感兴趣但 QNX Neutrino OS 核心组件文档中未描述的non-hypervisor QNX 组件的文档。

## QNX Hypervisor 公共论坛

QNX Hypervisor 公共论坛托管有关不同 QNX hypervisor 主题以及为 QNX hypervisor 系统构建guests 系统并运行它们的技术说明和常见问题解答。
要访问 QNX Hypervisor 公共论坛，请联系您的 [ QNX representative](http://www.qnx.com/company/contact/)以激活您对 Foundry27 的访问权限，然后使用您的 myQNX 帐户登录并访问:[QNX Hypervisor论坛](http://community.qnx.com/sf/sfmain/do/viewProject/projects.qnx_hypervisor_public_forum)。





