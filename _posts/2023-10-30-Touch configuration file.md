---
layout:     post
title:      Touch configuration file
# subtitle:    "\"Hello World, Hello Blog\""
date:       2023-10-30
author:     Underdog Linux
header-img: img/post-bg-2015.jpg
catalog: true
tags:
    - mtouch
---

mtouch 部分指定应用平台支持的触摸设备（也称为多点触控设备的 mtouch 设备）的配置。

touch 配置文件（默认名称为 mtouch.conf）文件是由 mtouch 服务解析的自由格式 ASCII 文本文件。
该文件可以包含额外的制表符和空行，以便进行格式设置。文件中的关键字区分大小写。
注释可以放在文件中的任何位置（引号内除外）。注释以 # 字符开头，在行尾结束。

仅当系统支持触摸设备，或者系统上有需要触摸的应用程序时，
该部分才必须以 `begin mtouch` 开头，以` end mtouch` 结尾。

配置文件中**可以有多个 mtouch 部分**。
mtouch 部分的数量必须与您的平台支持的物理显示器数量相匹配。
您可以在每个 mtouch 部分指定不同的参数（例如驱动程序）以匹配显示。


下面是目标上` /etc/system/config/mtouch.conf` 的 mtouch 部分的示例。
此文件是特定于主板的，因此是要使用的主板的主板支持包 （BSP） 的一部分。

> 如果使用 -c 选项启动 mtouch 驱动程序，则可以为配置文件指定不同的路径和不同的名称。有关详细信息，请参阅 [mtouch](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.screen/topic/manual/mtouch.html)。

```
begin mtouch
  driver = hid
  options = vid=0x0eef,did=0x7200,verbose=6,width=4096,height=4096,max_touchpoints=1
end mtouch
```

# mtouch 部分的参数
以下是可以在 mtouch 部分下配置的有效参数： 通常使用[display]()，[scaling]()，[driver]()和[options]()。

**device_name**
一个字符串值，该值指定设备的名称。

**driver**
要使用的触摸驱动程序的名称。
该名称特定于每个板。

> 译者注：下面这句话是重点

**mtouch 服务使用以下算法来确定要加载的触摸驱动程序**：`“libmtouch-” + 驱动程序名称 + “.so”`

例如，您有驱动程序名称，例如 egalax、focaltech、hid 或 lg-tsc101，它们对应于 libmtouch-egalax.so、libmtouch-focaltech.so、libmtouch-hid.so 和 libmtouch-lg-tsc101.so。

> libmtouch-hid.so 驱动器用于连接到电路板的显示器。通常，显示器使用 HDMI 或 DVI 和 USB 电缆连接。有关用于不同显示器的各种设置的示例，参阅[Example of various displays to use]()，有关使用 HID 设备进行故障排除的信息，参阅[Troubleshooting HID display issues]()。

**display**
显示标识符 （Integer） 或用于显示的连接类型 （String）。
1. 作为整数，指定与此触摸设备关联的显示器的标识符。显示 ID 在图形配置文件（例如 graphics.conf）的显示子部分中标识。
还可以使用 Screen API 从 Screen 显示对象的` SCREEN_PROPERTY_ID `属性中检索显示 ID。
2. 作为 String，可用于此参数的值为：
- composite
显示器通过复合连接进行连接。
- dvi
显示器通过DVI连接进行连接。
- hdmi
显示器通过 HDMI 连接进行连接。
- internal
显示器连接到电路板。
- rgb
显示器通过组件连接的RGB信号连接。
- rgbhv
显示器通过组件连接的 RBGHV 信号连接。
- svideo
显示器通过S-Video视频连接。
- YPbPr
显示器通过分量视频连接，特别是分量连接的YPbPr信号。

**min_event_interval**
一个无符号整数 （0 - 4,294,967,295），用于指定触摸控制器的采样率（以微秒为单位）。
此指定时间表示两个触摸样本之间的最短时间。
此配置的有效性取决于驱动程序。

**scaling**
一个字符串值，该值指定 scaling.conf 文件的位置。
您可以指定其他名称。
如果未指定此参数，则系统上的预期位置和文件名为：`/etc/system/config/scaling.conf`

**scaling_factor**
将触摸矩阵中的缩放因子覆盖到 scaling.conf 文件中指定的` mode` 参数。
此参数的值可以指定为十进制值，例如 2.5。

**options**
**选项配置取决于驱动程序**。
**配置的字符串将逐字传递到驱动程序本身**。
例如，它可用于设置矩形触摸表面的高度和宽度（以像素为单位）。在这种情况下，您将设置触摸控制器硬件的最大值 X 和 Y 值。

**此配置必须采用 option_name=value 的形式，对于多个选项，用逗号 （，） 分隔**。例如：
`vid=vendorID ,did=productID ,max_touchpoints=number_of_touchpoints ,lvds=interface`
以下是一些示例：
- height=4096
- width=4096
- vid=0x0eef
- did=0x7200
- lvds=720
- verbose=6
- max_touchpoints=2

有关详细信息，请参阅主板的 BSP 用户指南。


# 各种 HID 显示器的设置
通常，可用于`options`参数的条目特定于触摸控制器硬件。
对于 HID，以下是可用的常见通用选项：

**did**
The USB Device ID or Product ID.


**height**
触摸屏的最大 Y 坐标值。

**invert_x**
X 坐标是反转的。
设置为 0（零）可将其关闭，设置为 1 可将其设置为打开。
默认值为 off。

**invert_y**
Y 坐标是反转的。
设置为 0（零）可将其关闭，设置为 1 可将其设置为打开。
默认值为 off。

**verbose**
将日志记录消息的详细级别设置。
您可以将数字设置为 0 到 6。
数字越大，表示日志数量越多。如果未指定，则默认值为 0（零）。

**vid**
The USB Vendor ID.

**width**
触摸屏的最大 X 坐标值。


若要确定宽度和高度值，请首先在 mode：direct 中运行驱动程序，然后通过在verbose模式下运行 mtouch 服务，从事件二进制文件中获取输出，以查看系统日志中的最大 X 和 Y 值。
或者，您可以从 hidview -R（逻辑最大 X 和 Y）获取输出，
或联系供应商以获取 HID 显示的输出。
更多关于`hidview`的信息，参阅QNX Neutrino Utilities Reference中的 [hidview](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.neutrino.utilities/topic/h/hidview.html)

以下是显示器的一些设置：

- UNITEC USB Touch (Windows XP and Windows 7)
```
begin mtouch
  driver = hid
  options = vid=0x0afa,did=0x07d2,verbose=6,width=4096,height=4096,max_touchpoints=2
 end mtouch
```

- Lilliput 10.4" LCD Monitor (Egalax Controller)
```
begin mtouch
  driver = hid
  options = vid=0x0eef,did=0x0001,verbose=6,width=4096,height=4096,max_touchpoints=1
end mtouch
```

- Egalax Controller (7200)
```
begin mtouch
  driver = hid
  options = vid=0x0eef,did=0x7200,verbose=6,width=4096,height=4096,max_touchpoints=1
end mtouch
```

- Egalax Controller (7201)
```
begin mtouch
  driver = hid
  options = vid=0x0eef,did=0x7200,verbose=6,width=4096,height=4096,max_touchpoints=1
end mtouch
```

- ST2220T LG Display
```
begin mtouch
  driver = hid
  options = vid=0x1fd2,did=0x0064,verbose=6,width=1024,height=1024,max_touchpoints=2
end mtouch
```

- GECHIC Monitor
```
begin mtouch
  driver = hid
  options = vid=0x222a,did=0x001c,verbose=6,width=7539,height=4211,max_touchpoints=10
end mtouch
```

- Flatfrog (E10-TM55F-0021 HID/DFU)
```
begin mtouch
  driver = hid
  options = vid=0x25b5,did=0x0021,verbose=6,width=19354,height=10886,max_touchpoints=5
end mtouch
```

- Chalkboard Electronics (HID Multi-touch)
```
begin mtouch
  driver = hid
  options = vid=0x04d8,did=0xf724,verbose=6,width=1366,height=768,max_touchpoints=2
end mtouch
```

- N-Trig (DuoSense)
```
begin mtouch
  driver = hid
  options = vid=0x1b96,did=0x0007,verbose=6,width=9600,height=7200,max_touchpoints=1
end mtouch
```

- Zytronic Displays Limited (ZXY200 Controller)
```
begin mtouch
  driver = hid
  options = vid=0x14c8,did=0x0006,verbose=6,width=4096,height=4096,max_touchpoints=2
end mtouch
```

- TPK USA LLC Touch: Fusion 4. (Lilliput)
```
begin mtouch
  driver = hid
  options = vid=0x1391,did=0x2112,verbose=6,width=1024,height=1024,max_touchpoints=1
end mtouch
```

# HID 问题疑难解答
如果您在使用 HID 设备时遇到问题，以下是一些故障排除做法： 如果触摸屏无法与 HMI 配合使用，请确保以下几点：

- 检查 USB 控制器是否正常工作。使用 HID 触摸控制器时，必须启动 USB 控制器。要检查 USB 控制器是否显示，请运行命令 usb -vvv 命令。如果没有，请检查：
    - 您已启动正确的驱动程序并在目标板上
    - 触摸控制器已通电。例如，对于 Lilliput 显示器，如果显示器没有电源或显示器没有视频，则触摸控制器不会从 USB 出现。

- 运行 hidview 命令时，控制器会显示。
如果没有，请确保 io-hid 是使用 devh-usb.so 模块启动的，并且没有发生错误。

- 当 mtouch 启动时，请检查 slog2info 是否有任何错误。
可以通过在驱动程序的配置文件中传递 verbose 选项来增加驱动程序的详细程度。
使用 -v 选项设置 mtouch 服务的详细级别。

- Slay HMI 流程，启动Screen和 mtouch。然后运行二进制事件，查看它是否从控制器接收触摸事件。如果二进制事件接收到触摸数据，则确认触摸屏在屏幕中配置正确，问题可能出在校准中。

- 检查触摸配置文件（mtouch.conf）中的参数。
需要注意的是，宽度和高度并不指定显示分辨率，而是指定触摸分辨率（即 MaxX 和 MaxY 值）。
您可以通过`hidview -R`命令获取宽度和高度值。这些值由 X 和 Y 报告的 HID Logical Max 分隔。

- 检查scaling配置文件 （scaling.conf） 中的设置。
显示器分辨率必须有匹配的配置，并且需要根据是否使用校准来正确设置。

`mode=scale `表示，**不要与校准一起使用**，Touch 框架会根据 touch 配置文件 （mtouch.conf） 的 options 参数中指定的宽度和高度以及显示器的分辨率来计算缩放比例。

`mode mode=direct` 表示使用 calib-touch，因为框架不执行任何缩放，而是直接发送坐标，不加更改。

- 如果您使用的是电阻式触摸屏或选择运行 calib-touch，则必须在每次屏幕和 mtouch 启动时运行 calib-touch。如果存在校准文件，则 calib-touch 只需加载校准矩阵并将设置应用于Screen Graphics Subsystem（Screen）服务。

> 请务必记住，Screen 不会自行应用矩阵设置(`matrix settings`)。






