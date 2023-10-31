---
layout:     post
title:      Scaling configuration file
# subtitle:    "\"Hello World, Hello Blog\""
date:       2023-10-31
author:     Underdog Linux
header-img: img/post-bg-2015.jpg
catalog: true
tags:
    - mtouch
---

scaling配置文件是 mtouch 服务使用的自由格式 ASCII 文本文件。

该文件可以包含额外的制表符和空行，以便进行格式设置。文件中的关键字区分大小写。
注释可以放在文件中的任何位置（引号内除外）。注释以 # 字符开头，在行尾结束。

默认情况下，mtouch 服务需要将扩展配置文件命名为` scaling.conf`，并位于` /etc/system/config`。
此文件是特定于主板的，因此是要使用的主板的主板支持包 （BSP） 的一部分。

如果您的缩放配置不在上述位置或未命名为 scaling.conf，则可以使用 touch 配置文件中的` scaling` 参数来指定不同的位置和名称。
有关详细信息，请参阅本指南中的 [mtouch]()一章。

您可以使用 scaling.conf 文件为每个触摸坐标定义模式。
您可以为每个配置定义一种模式。
推荐的模式是根据显示器的分辨率配置的缩放模式。
例如，scaling.conf 文件中的条目如下所示：
```
1280x720:mode=scale
```

mode默认是`direct`，原因如下
- 如果未定义模式或 scaling.conf 文件不存在
- 如果找不到匹配的显示分辨率

最常用的模式是`direct  `和`scale`，支持的模式如下：

**calib**
触摸坐标根据校准数据进行缩放和移动。
这是一种提供手动校准的机制。
此选项允许您为四个角提供四个点。基于这些点，使用简单的算法进行校准。
这些点应该**在四个角附近，但不在最极端的角上**，而是在距离角大约半英寸的地方。

以下是可用于缩放和移位的参数：
- disp_x=[X1:X2:X3:X4]
- disp_y=[Y1:Y2:Y3:Y4]
- mtouch_x=[mX1:mX2:mX3:mX4]
- mtouch_y=[mY1:mY2:mY3:mY4]

例如，要根据校准数据进行缩放：
```
1024x600:mode=calib,disp_x=[102:922:922:102],disp_y=[60:60:540:540],
                    mtouch_x=[172:1108:1108:172],mtouch_y=[110:110:658:658]
```
有关在系统上校准触摸的其他方法的信息，参阅[Calibrating touch](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.screen/topic/manual/mtouch_calibration.html)


**dim**
触摸坐标会根据触摸显示器的物理尺寸进行缩放和移动。
您可以使用以下以逗号分隔的参数指定坐标的缩放方式：
- width_mm=w_scale
- height_mm=h_scale

您可以使用以下参数指定坐标的移动方式，这些参数以逗号分隔：
- border_left_mm=bl_coord
- border_right_mm=br_coord
- border_top_mm=bt_coord
- border_bottom=bt_coord

例如，要根据显示器的物理尺寸进行缩放：
```
1024x600:mode=dim,width_mm=154,height_mm=90,border_left_mm=5,
                  border_right_mm=5,border_top_mm=5,border_bottom_mm=5
```



**direct**
触摸坐标与从驱动程序传入的坐标完全相同。
例如，要不执行缩放以匹配 1280x768 的显示分辨率：
```
1280x768:mode=direct
```



**rect**
如果触摸坐标位于指定的（居中）矩形之外，则忽略它们。
坐标也会在矩形的左上角重新定位。
偏移量表示矩形左上角相对于显示器左上角的偏移量。

您可以使用逗号分隔的参数指定偏移量：
- offset_x=x_offset
- offset_y=y_offset

例如，要在分辨率为 1312 x 800 的显示器上缩放到 1920x1080 触摸屏上的居中矩形：
```
1312x800:mode=rect,offset_x=304,offset_y=140
```

> 如果针对 VMware 进行扩展，则偏移量的计算方式如下：例如，offset_x = （1920 / 2） - （1312 / 2），offset_y = （1080 / 2） - （800 / 2）


**scale**
触摸坐标由Touch服务缩放到客户端显示器的分辨率。
例如，要缩放显示器上的分辨率：
```
1024x600:mode=scale
```


**spec**
触摸坐标根据触摸面板的物理规格进行缩放。
这不应与触摸屏的物理尺寸相混淆。
此参数可用于减小可用区域的大小。
例如，如果您的显示器上有挡板，或者您希望忽略触摸屏的外边缘，则您定义的坐标允许您创建一个小的触摸区域，该区域仅报告该定义区域内的数据。
您可以通过在` mode=direct` 中运行驱动程序来了解这些值。

以下是可用于定义区域坐标的参数：

- offset_bottom_tp=bottom_spec
- offset_top_tp=top_spec
- offset_left_tp=left_spec
- offset_right_tp=right_spec

例如，要根据触摸面板规格进行缩放：
```
768x1280:mode=spec,offset_bottom_tp=324,offset_top_tp=20,
                   offset_left_tp=27,offset_right_tp=27
```
