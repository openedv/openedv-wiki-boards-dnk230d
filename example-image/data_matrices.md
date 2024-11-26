---
title: 'DM码识别实验'
sidebar_position: 16
---

# DM码识别实验

## 前言

在上个章节中，已经了解了如何在CanMV下使用image模块实现AprilTags码识别的方法，本章将通过Data Matrix二维码（简称DM码，下同）识别实验，介绍如何使用CanMV的find_datamatrices()方法实现DM码识别功能。在本实验中，我们将摄像头捕获的图像进行处理，查找图像中所有的数据矩阵，并将结果绘制并显示到显示器上。通过本章的学习，读者将学习到如何在CanMV下使用find_datamatrices()方法实现DM码识别的功能。

Data Matrix二维码是一种由黑色、白色的色块以正方形或长方形组成的二维码，其发展构想是希望在较小的标签上存储更多的信息量。DM码的最小尺寸是目前所有条码中最小的，特别适合于小零件的的标识，直接印刷在实体上，被广泛应用在电路、药品等小件物品以及制造业的流水线生产过程。

## Image模块介绍

### 概述

`Image`类是机器视觉处理中的基础对象。此类支持从Micropython GC、MMZ、系统堆、VB区域等内存区域创建图像对象。此外，还可以通过引用外部内存直接创建图像（ALLOC_REF）。未使用的图像对象会在垃圾回收时自动释放，也可以手动释放内存。

支持的图像格式如下：

- BINARY
- GRAYSCALE
- RGB565
- BAYER
- YUV422
- JPEG
- PNG
- ARGB8888（新增）
- RGB888（新增）
- RGBP888（新增）
- YUV420（新增）

支持的内存分配区域：

- **ALLOC_MPGC**：Micropython管理的内存
- **ALLOC_HEAP**：系统堆内存
- **ALLOC_MMZ**：多媒体内存
- **ALLOC_VB**：视频缓冲区
- **ALLOC_REF**：使用引用对象的内存，不分配新内存

### API描述

‌Python中的Image模块是一个强大的图像处理工具，它提供了一系列函数和方法，可以用于图像元素绘制、图像滤波、图像特征检测、色块追踪、图像对比和码识别等。由于image模块功能强大，需要介绍的内容也比较多，因此本章仅介绍image模块中find_datamatrices()方法的使用。

#### find_datamatrices

```python
image.find_datamatrices([roi[, effort=200]])
```

该函数查找指定ROI内的所有数据矩阵，并返回一个包含`image.datamatrix`对象的列表。有关更多信息，请参考`image.datamatrix`对象的相关文档。

为了确保该方法成功运行，图像上的矩形码需尽量平展。可以通过使用sensor.set_windowing()函数在镜头中心放大、使用image.lens_corr()函数消除镜头的桶形畸变，或更换视野较窄的镜头，获得不受镜头畸变影响的平展矩形码。部分机器视觉镜头不产生桶形失真，但其成本高于OpenMV提供的标准镜头，这些镜头为无畸变镜头。

【参数】

- roi：是用于指定感兴趣区域的矩形元组(x, y, w, h)。若未指定，ROI默认为整个图像。操作范围仅限于该区域内的像素。
- effort：控制查找矩形码匹配所需的计算时间。默认值为200，适用于所有用例。然而，您可以在降低检测率的情况下提高帧速率，或在降低帧速率的情况下提高检测率。请注意，当effort设置低于约160时，无法进行任何检测；相反，您可以将其设置为任何更高的值，但若设置高于240，检测率将不会进一步提高。

注意：不支持压缩图像和Bayer格式图像。

更多用法请阅读官方API手册：

https://developer.canaan-creative.com/k230_canmv/dev/zh/api/openmv/image.html#image

## 硬件设计

### 例程功能

1. 系统会获取摄像头输出的图像，并使用image模块中的find_datamatrices()方法查找图像中所有的DM码。当识别到DM码时，系统会在DM码周围绘制一个矩形框，并在图像的左上角显示DM码的识别结果。最后，处理后的图像将显示在LCD屏幕上。

### 硬件资源

1. 本章实验内容，主要讲解image模块的使用，无需关注硬件资源。  


### 原理图

本章实验内容，主要讲解image模块的使用，无需关注原理图。  

## 实验代码

``` python
import time, math, os, gc
from media.sensor import *  # 导入sensor模块，使用摄像头相关接口
from media.display import * # 导入display模块，使用display相关接口
from media.media import *   # 导入media模块，使用meida相关接口

try:
    sensor = Sensor(width=1280, height=960) # 构建摄像头对象
    sensor.reset() # 复位和初始化摄像头
    sensor.set_framesize(Sensor.VGA)    # 设置帧大小VGA(640x480)，默认通道0
    sensor.set_pixformat(Sensor.RGB565) # 设置输出图像格式，默认通道0

    # 初始化LCD显示器，同时IDE缓冲区输出图像,显示的数据来自于sensor通道0。
    Display.init(Display.ST7701, width=640, height=480, fps=90, to_ide=True)
    MediaManager.init() # 初始化media资源管理器
    sensor.run() # 启动sensor
    clock = time.clock() # 构造clock对象

    while True:
        os.exitpoint() # 检测IDE中断
        clock.tick()  # 记录开始时间（ms）
        img = sensor.snapshot() # 从通道0捕获一张图

        # 遍历图像中的二维码
        for matrix in img.find_datamatrices():
            # 绘制识别到的 Data Matrix 码的矩形框
            img.draw_rectangle([v for v in matrix.rect()], color=(255, 0, 0))
            img.draw_string_advanced(0, 0, 30, matrix.payload(), color=(255, 255, 255)) # 图像显示DM码信息
            print_args = (matrix.rows(), matrix.columns(), matrix.payload(), (180 * matrix.rotation()) / math.pi, clock.fps())
            print("矩阵 [%d:%d], 内容 \"%s\", 旋转 %f (度), FPS %f" % print_args)

        # 显示图片
        Display.show_image(img)

# IDE中断释放资源代码
except KeyboardInterrupt as e:
    print("user stop: ", e)
except BaseException as e:
    print(f"Exception {e}")
finally:
    # sensor stop run
    if isinstance(sensor, Sensor):
        sensor.stop()
    # deinit display
    Display.deinit()
    os.exitpoint(os.EXITPOINT_ENABLE_SLEEP)
    time.sleep_ms(100)
    # release media buffer
    MediaManager.deinit()
```

可以看到一开始是先初始化了LCD和摄像头。接着在一个循环中不断地获取摄像头输出的图像，因为获取到的图像就是Image对象，因此可以直接调用image模块为Image对象提供的各种方法，然后就是对图像中的DM码进行检测和识别，并在LCD左上角绘制识别到的DM码信息，最后在 LCD 显示图像。

## 运行验证

实物原图如下图所示：

![01](./img/29.png)

将K230D BOX开发板连接CanMV IDE，并点击CanMV IDE上的“开始(运行脚本)”按钮后，可以看到LCD上实时地显示这摄像头采集到的画面，如下图所示：

![01](./img/30.png)

点击左下角“串行终端”，可以看到“串行终端”窗口中输出了一系列信息，如下图所示：

![01](./img/31.png)
