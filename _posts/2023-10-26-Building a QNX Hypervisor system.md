---
layout:     post
title:      Building a QNX Hypervisor system
# subtitle:    "\"Hello World, Hello Blog\""
date:       2023-10-26
author:     Underdog Linux
header-img: img/post-bg-2015.jpg
catalog: true
tags:
    - Building a QNX Hypervisor system
---

本章介绍如何构建（组装）QNX Hypervisor 系统并将bootable 镜像传输到支持的目标平台。

当您构建 QNX hypervisor时，您无需重新编译该hypervisor。 
如果您添加或修改了组件（例如驱动程序或 vdev），您当然必须编译它们。 
但是，QNX hypervisor序**仅作为二进制文件**提供。 你不编译它们； 当您将bootable 镜像与hypervisor系统组装在一起时，您可以不加修改地包含它们。

# 先决条件
> 在开始工作之前，请检查您的主板的硬件和固件是否支持虚拟化。
> 即使您的主板硬件支持运行 QNX 虚拟化系统所需的所有虚拟化功能，主板上的固件也可能无法启用甚至可能明确禁用其中某些功能，在这种情况下您将无法运行hypervisor 。
> 您可以联系您的主板供应商，以确认您正在考虑的主板支持您所需的虚拟化功能，并且您的主板具有适当的固件。

要构建hypervisor 系统，您需要配置和构建hypervisor 主机及其guest。 这些说明假设您知道：
- about QNX Board Support Packages (BSPs)
- about QNX buildfiles
- how to use the QNX Software Center
- how to build an IFS with a QNX OS

