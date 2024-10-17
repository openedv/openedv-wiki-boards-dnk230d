---
title: '色块计数实验'
sidebar_position: 10
---

# 色块计数实验

## 前言

在上一章节中，已经了解了如何在CanMV下使用image模块实现多颜色识别的方法，本章将通过色块计数实验，介绍如何使用CanMV的find_blobs()方法实现色块计数功能。在本实验中，我们将摄像头捕获的图像进行处理，查找图像中所有符合目标的色块，并进行计数，然后将结果绘制并显示到显示器上。通过本章的学习，读者将学习到如何在CanMV下使用find_blobs()方法结合其返回的色块对象实现色块计数的功能。

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

‌Python中的Image模块是一个强大的图像处理工具，它提供了一系列函数和方法，可以用于图像元素绘制、图像滤波、图像特征检测、色块追踪、图像对比和码识别等。由于image模块功能强大，需要介绍的内容也比较多，因此本章仅介绍image模块中find_blobs()方法以及色块对象的使用。

有关find_blobs()方法的介绍，请见[单颜色识别实验的find_blobs()方法介绍](single_color.md#api描述)

更多用法请阅读官方API手册：

https://developer.canaan-creative.com/k230_canmv/dev/zh/api/openmv/image.html

## 硬件设计

### 例程功能

1. 获取摄像头输出的图像，并使用image模块的find_blobs()方法查找图像上所有的目标色块并标记，然后将色块数量也绘制到图像上，最后将图像显示在LCD上。

### 硬件资源

1. 本章实验内容，主要讲解image模块的使用，无需关注硬件资源。


### 原理图

本章实验内容，主要讲解image模块的使用，无需关注原理图。

## 实验代码

``` python
import time, os, sys
from media.sensor import *  #导入sensor模块，使用摄像头相关接口
from media.display import * #导入display模块，使用display相关接口
from media.media import *   #导入media模块，使用meida相关接口

# 颜色识别阈值 (l_lo, l_hi, a_lo, a_hi, b_lo, b_hi) 即LAB模型，对应 LAB 色彩空间中的 L、A 和 B 通道的最小和最大值
# 下面的阈值元组是用来识别 红、绿、蓝三种颜色，你可以根据使用场景调整提高识别效果。
thresholds = [(18, 72, -13, 31, 18, 83)] #识别物体的颜色阈值，默认黄色色块

try:
    sensor = Sensor() #构建摄像头对象
    sensor.reset() #复位和初始化摄像头
    sensor.set_framesize(Sensor.VGA)    #设置帧大小QVGA(320x240)，默认通道0
    sensor.set_pixformat(Sensor.RGB565) #设置输出图像格式，默认通道0

    # 初始化LCD显示器，同时IDE缓冲区输出图像,显示的数据来自于sensor通道0。
    Display.init(Display.ST7701, width = 800, height = 480, fps=60, to_ide = True)
    MediaManager.init() #初始化media资源管理器
    sensor.run() #启动sensor
    clock = time.clock() # 构造clock对象

    while True:
        os.exitpoint() #检测IDE中断
        clock.tick()  #记录开始时间（ms）
        img = sensor.snapshot() #从通道0捕获一张图
        blobs = img.find_blobs([thresholds[0]], pixels_threshold= 100) # 0,1,2分别表示红，绿，蓝色。
        for blob in blobs:
            img.draw_rectangle(blob[0], blob[1], blob[2], blob[3], color = (255, 255, 255),  thickness = 4)
            img.draw_cross(blob[5], blob[6], color = (255, 255, 255),  thickness = 4)

        #显示计算信息
        img.draw_string_advanced(0, 0, 30, 'Num: '+str(len(blobs)), color = (255, 255, 255))
        # 显示图片
        Display.show_image(img, x=round((800-sensor.width())/2),y=round((480-sensor.height())/2))
        print(clock.fps()) #打印FPS

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

可以看到一开始是先初始化了LCD和摄像头。接着在一个循环中不断地获取摄像头输出的图像，因为获取到的图像就是Image对象，因此可以直接调用image模块为Image对象提供的各种方法，find_blobs()方法每查找到一个符合颜色值的色块就会返回一个色块对象的列表，因此，我们只需要统计色块对象的列表数量，就能获取图像中包含目标色块的数量，然后将图像里所有符合目标的色块画框标记并进行计数，最后在LCD显示处理好后的图像。

## 运行验证

实验原图如下所示：

![01](./img/17.png)

将DNK230D开发板连接CanMV IDE，并点击CanMV IDE上的“开始(运行脚本)”按钮后，可以看到LCD上实时地显示这摄像头采集到的画面，如下图所示：

![01](./img/18.png)

也可以在CanMV IDE看到摄像头采集的画面，如下图所示：

![01](./img/18.png)

