---
layout:     post
title:      Calibrating touch
# subtitle:    "\"Hello World, Hello Blog\""
date:       2023-10-30
author:     Underdog Linux
header-img: img/post-bg-2015.jpg
catalog: true
tags:
    - mtouch
---

> 译者注：当前项目，没有用到

您可以使用几种不同的方法校准屏幕。

在scaling配置文件中设置 mode 参数可确定配置类型。
根据您的触摸屏，您需要将模式参数设置为`direct`或`scaled`。
scaling配置文件是位于 /etc/system/config 的` scaling.conf `文件，有关更多信息，请参阅[ Scaling configuration file ](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.screen/topic/manual/mtouch_scaling_config.html)。

以下是校准触摸屏的不同方法：
1. manual
在缩放配置文件中使用 calib 作为模式。
更多信息参考[calib](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.screen/topic/manual/mtouch_scaling_config.html#mtouch_scaling_config__caliboption)选项
2. direct mapping
此方法用于**电阻式**触摸显示器
更多信息参考[ Direct mapping](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.screen/topic/manual/mtouch_calibration.html#mtouch_calibration__directcalibrated)

3. scaled
此方法用于电容式触摸显示器
更多信息参阅[ Scaled calibration](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.screen/topic/manual/mtouch_calibration.html#mtouch_calibration__scaledcalibration)


# Direct mapping
这种校准方法适用于电阻式触摸显示器或可以在硬件上缩放值的显示器（例如，Atmel 控制器）。
要使用此方法，请执行以下操作：
1. 在伸缩配置文件中将mode参数设置为direct。
2. 触摸驱动程序和屏幕服务成功运行后，启动` calib-touch `实用程序。
Screen 完成运行后，将创建一个校准文件并将其加载到 Screen 的转换矩阵中。
有关 calib-touch 实用程序的更多信息，参阅[calib-touch](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.screen/topic/manual/calib-touch.html)。

# Scaled calibration
这种校准方法适用于电容式触摸显示器。
一旦挂载，这些值是静态的，不会随时间而偏斜。

Scaling模式将触摸区域映射到显示分辨率。
要使用此方法，您需要找到触摸屏的显示分辨率和矩阵大小。

## Display resolution
首先，将缩放模式设置为scale并包含显示器的分辨率。
分辨率是从 `graphics.conf` 中获取的，需要在缩放配置文件中设置。
例如，`graphics.conf `可能像这样：
```
begin display 1
  cursor=on
  video-mode = 800 x 480 @ 60          
  formats = rgb565 rgba8888 rgb888 yuy2
end display       
```

在此示例中，分辨率为 800x480。更新 scaling.conf 文件中的缩放模式，并将其设置为 scale：
```
800x480:mode=scale                 
```

## Touch display matrix size
接下来，将 width 和 height 属性设置为触摸区域的最大 X 和 Y 值。
将这些值放在位于 /etc/system/config 的 touch.conf 文件中。您的配置可能如下所示：
```
begin mtouch
  driver = hid
  options = width=4095,height=4095,your_other_options
end mtouch
```

> width和height选项**不是指您的显示分辨率**。它们是指触摸区域的最大 X 和 Y 值。下面介绍如何查找这些值。

## How to find the touch display matrix size
- USB/HID based touch controllers:

使用` hidview -R `的输出搜索 Usage（X/Y） 和关联的 Logical Max（value）。
在某些系统上，Logical Max(value)称为Physical Max(value)。
在以下输出示例中，X 和 Y 值均为 4095：
```
...                
Logical Max(4095)
Report Size(16)
Report Count(1)
PUSH
Unit Exponent(14)
Unit(51)
Usage(X)
...
Logical Max(4095)
Usage(Y)
...
```

- Other Controllers:
请参阅您的控制器文档。


## Finding the maximum values directly from the touch display
使用触摸屏查找最大值是不精确的。
如果使用上述说明无法找到最大 X 和 Y，请执行以下操作：

1. 将 scaling.conf 中的缩放模式设置为 direct：
```
800x480:mode=direct                     
```

2. 启动`mtouch`
```
mtouch &
```

3. 运行screeninfo
```
screeninfo -w -input /dev/screen/input                        
```

4. 触摸屏幕的四个角，并记下 screeninfo 的输出。
其中一个值应该是最大 X，Y 值的合理近似值。
由于此方法不精确，因此可以四舍五入。
例如，如果获得 X 的值 4034，则最大 X 很可能是 4095。

5. 在`touch.conf`中将width和height属性设置为最大的X、Y值
```
 begin mtouch
  driver = hid
  options = width=4095,height=4095,your_other_options
end mtouch    
```

6. 将 scaling.conf 中的缩放模式更改回 scale：
```
800x480:mode=scale                     
```

值得一提的是，此方法不会纠正未对齐的触摸显示器，但如果原点不在显示器的左上角，则可以使用invert_x和invert_y选项来反转坐标的方向。
更多信息参阅mtouch部分的[ Settings for various HID displays](https://www.qnx.com/developers/docs/7.1/com.qnx.doc.screen/topic/manual/mtouch.html#mtouch_config__hidexamples)
