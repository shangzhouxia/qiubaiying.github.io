---
layout:     post
title:      Touch filters
# subtitle:    "\"Hello World, Hello Blog\""
date:       2023-10-30
author:     Underdog Linux
header-img: img/post-bg-2015.jpg
catalog: true
tags:
    - mtouch
---

可以通过指定与每个触摸驱动程序相关的筛选器来指定进一步的配置。

# Touch Filters

在 mtouch 部分中，以 begin filter 开始一个子部分，以 end filter 结束以指定一个过滤器。例如：

    begin mtouch
        driver = focaltech
        options = generic_j6=1,interrupt=1002,i2c_slave_addr=0x38,i2c_devname=/dev/i2c0,protocol_ver=2

        begin filter
    ...    type
    ...    options
        end filter
    end mtouch

# Touch filter parameters

Touch filters提供了一种去除不需要的坐标或坐标噪点的方法，具体取决于您使用的filter类型。
某些filter会根据过滤器的用途调整坐标数据或禁止显示坐标数据。
以下是可用的触摸筛选器参数：

**type**

String
应用于触摸坐标的一系列filters。
可能的取值

*   ballistic
    最大限度地减少通常在遵循低速弹道轨迹的连续触摸事件中看到的噪音。
    弹道运动（例如，静止手指）的速度越低，对运动速度施加的增益降低就越大。
    通过这样做，结果是没有噪音的坚固触感。

*   compression
    压缩发送到 Screen 的事件数。
    **将删除重复事件**，并压缩指定阈值内的事件。

*   edge\_swipe\_detect
    检测手指是否经过触摸表面的边缘。这使您可以从屏幕外部开始滑动手势。

*   kalman
    通过使用卡尔曼滤波算法，将移动手指在给定触摸表面上的跟踪噪声降至最低。

**options**

String

特定于筛选器的选项。可用的过滤器选项取决于过滤器类型。配置筛选器选项的格式为：

`options = filter_option=值`

其中` filter_option` 是“Touch Filter options”下列出的选项之一，value 是要将其设置为的关联值。
如果要指定多个选项，则用逗号分隔每个选项值对：

`options = filter_option1=值，filter_option2=值`

根据您的过滤器类型，请参阅下文了解有效的过滤器选项。

# Touch Filter options

有效的配置过滤器选项取决于过滤器类型。
请参阅下面的 mtouch\_filter 部分（从mtouch\_filter开始）的 options 参数中可接受的筛选器选项。

*   **Ballistic filter (e.g., type = ballistic)**

| Filter option | Description         | Default |
| ------------- | ------------------- | ------- |
| scale         | FP scale factor     | 256     |
| min\_gain     | Minimal gain        | 8       |
| low\_speed    | Low speed threshold | 32      |

*   **Compression filter (e.g., type = compression)**

| Filter option            | Description               | Default |
| ------------------------ | ------------------------- | ------- |
| threshold                | 阈值是坐标在发送事件之前从其起点可以漂移的最大距离 | 10      |
| min\_event\_interval     | 最小事件间隔（以微秒为单位）            | -1      |
| num\_coords\_to\_average | 事件坐标数为平均值                 | 0       |

*   **Edge-swipe filter (e.g., type = edge\_swipe\_detect)**

    这些是过滤器顺序的选项：

    | Filter option | Description | Default |
    | :------------ | :---------- | :------ |
    | filter\_order | 过滤顺序        | 2       |

这些是左边缘的选项（例如，当 x 触摸坐标为 0 时）：

| Filter option                 | Description | Default |
| :---------------------------- | :---------- | :------ |
| xl\_enable                    | 应用检测        | 1       |
| xl\_bezel                     | 边框宽度        | 50      |
| xl\_speed                     | 速度阈值        | 25      |
| xl\_jitter                    | 抖动控制        | 0       |
| xl\_*reject\_*physical\_bezel | 拒绝物理边框      | 0       |

这些是顶部边缘的选项（例如，当 y 触摸坐标为 0 时）：

| Filter option                 | Description | Default |
| :---------------------------- | :---------- | :------ |
| yl\_enable                    | 应用检测        | 1       |
| yl\_bezel                     | 边框宽度        | 80      |
| yl\_speed                     | 速度阈值        | 25      |
| yl\_jitter                    | 抖动控制        | 0       |
| yl\_*reject\_*physical\_bezel | 拒绝物理边框      | 0       |

这些是底部边缘的选项（例如，当 y 触摸坐标处于最大可能值时）：

| Filter option                 | Description | Default |
| :---------------------------- | :---------- | :------ |
| yh\_enable                    | 应用检测        | 1       |
| yh\_bezel                     | 边框宽度        | 50      |
| yh\_speed                     | 速度阈值        | 25      |
| yh\_jitter                    | 抖动控制        | 0       |
| yh\_*reject\_*physical\_bezel | 拒绝物理边框      | 0       |

> 边缘滑动过滤器选项的命名法是：
>
> *   xl x-低边界，x =0 附近；
> *   xh x-高边框，x接近 = 最大X触摸坐标；
> *   yl y-下边界，y接近 = 0；
> *   yh y-高边框，y接近 = 最大 Y 触摸坐标；



*   **Kalman filter (e.g., type = kalman)**

    这些是噪声方差的选项：

    | Filter option         | Description	 | Default |
    | :-------------------- | :----------- | :------ |
    | proc*\_noise\_x\_var* | 过程噪声方差 (X)   | 32      |
    | proc*\_noise\_y\_var* | 过程噪声方差 (Y)   | 32      |
    | mean*\_noise\_x\_var* | 测量噪声方差(X)    | 100     |
    | mean*\_noise\_y\_var* | 测量噪声方差(Y)    | 100     |

这些是自适应速度阈值的选项：

| Filter option        | Description | Default |
| :------------------- | :---------- | :------ |
| slot\_*threshold\_*1 | 自适应速度阈值1    | 30      |
| slot\_*threshold\_*2 | 自适应速度阈值2    | 20      |
| slot\_*threshold\_*3 | 自适应速度阈值3    | 10      |
| slot\_*threshold\_*4 | 自适应速度阈值4    | 5       |











