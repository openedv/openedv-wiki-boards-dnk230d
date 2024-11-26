---
title: '线段检测实验'
sidebar_position: 4
---

# 线段检测实验

## 前言

在上一章节中，已经了解了如何在CanMV下使用image模块实现直线检测的方法，本章将通过线段检测实验，介绍如何使用CanMV的find_line_segments()方法实现线段检测功能。在本实验中，我们将摄像头捕获的图像进行处理，查找图像中所有物体的线段，并将结果绘制并显示到显示器上。通过本章的学习，读者将学习到如何在CanMV下使用find_line_segments()方法实现线段检测的功能。

## Image模块介绍

### 概述

`Image` 类是机器视觉处理中的基础对象。此类支持从Micropython GC、MMZ、系统堆、VB区域等内存区域创建图像对象。此外，还可以通过引用外部内存直接创建图像（ALLOC_REF）。未使用的图像对象会在垃圾回收时自动释放，也可以手动释放内存。

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

‌Python中的Image模块是一个强大的图像处理工具，它提供了一系列函数和方法，可以用于图像元素绘制、图像滤波、图像特征检测、色块追踪、图像对比和码识别等。由于image模块功能强大，需要介绍的内容也比较多，因此本章仅介绍image模块中find_line_segments函数的使用。

#### find_line_segments

```python
image.find_line_segments([roi[, merge_distance=0[, max_theta_difference=15]]])
```

该函数使用霍夫变换查找图像中的线段，并返回一个image.line对象的列表。

【参数】

- roi：为感兴趣区域的矩形元组(x, y, w, h)。若未指定，ROI默认为整个图像的矩形。操作仅限于该区域内的像素。
- merge_distance：指定两条线段之间的最大像素距离，若小于该值则合并为一条线段。
- max_theta_difference：为需合并的两条线段之间的最大角度差。

该方法使用LSD库（OpenCV亦采用）来查找图像中的线段。虽然速度较慢，但准确性高，且线段不会出现跳跃现象。

注意：此功能不支持压缩图像和Bayer图像。

更多用法请阅读官方API手册：

https://developer.canaan-creative.com/k230_canmv/dev/zh/api/openmv/image.html

## 硬件设计

### 例程功能

1. 获取摄像头输出的图像，并使用image模块的find_line_segments()方法查找图像上所有线段并绘制出来，最后将图像显示在LCD上。

### 硬件资源

1. 本章实验内容，主要讲解image模块的使用，无需关注硬件资源。


### 原理图

本章实验内容，主要讲解image模块的使用，无需关注原理图。

## 实验代码

``` python
import time, os, sys
from media.sensor import *  # 导入sensor模块，使用摄像头相关接口
from media.display import * # 导入display模块，使用display相关接口
from media.media import *   # 导入media模块，使用meida相关接口

try:
    sensor = Sensor(width=1280, height=960) #构建摄像头对象
    sensor.reset()  # 复位和初始化摄像头
    sensor.set_framesize(Sensor.QVGA)   # 设置帧大小QVGA(320x240)，默认通道0
    sensor.set_pixformat(Sensor.RGB565) # 设置输出图像格式，默认通道0

    # 初始化LCD显示器，同时IDE缓冲区输出图像,显示的数据来自于sensor通道0。
    Display.init(Display.ST7701, width=640, height=480, fps=90, to_ide=True)
    MediaManager.init() # 初始化media资源管理器
    sensor.run() # 启动sensor
    clock = time.clock() # 构造clock对象

    while True:
        os.exitpoint() # 检测IDE中断
        clock.tick()   # 记录开始时间（ms）
        img = sensor.snapshot() # 从通道0捕获一张图

        for l in img.find_line_segments(merge_distance=2, max_theta_diff=15):
            img.draw_line(l.line(), color=(255, 0, 0), thickness=2)
            print(l)

        Display.show_image(img, x=round((640 - sensor.width()) / 2), y=round((480 - sensor.height()) / 2))
        print(clock.fps()) # 打印FPS

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

可以看到一开始是先初始化了LCD和摄像头。接着在一个循环中不断地获取摄像头输出的图像，因为获取到的图像就是Image对象，因此可以直接调用image模块为Image对象提供的各种方法，因为该方法使用的是LSD库，所以速度比较慢，我们需要降低图像分辨率以提高识别速度，然后将图像里所有的线段绘制出来，最后在LCD显示处理好后的图像。

## 运行验证

将K230D BOX开发板连接CanMV IDE，并点击CanMV IDE上的“开始(运行脚本)”按钮后，可以看到LCD上实时地显示这摄像头采集到的画面，如下图所示：

![01](./img/05.png)

