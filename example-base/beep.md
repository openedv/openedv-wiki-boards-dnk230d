---
title: '蜂鸣器实验'
sidebar_position: 2
---

# 蜂鸣器实验

## 前言

K230D内部包含64个GPIO Pin，每个Pin可配置为输入或输出，可配置上下拉，可配置驱动能力。

本章实验将介绍如何使用CanMV让Kendryte K230D控制板载的蜂鸣器发声。通过本章的学习，读者将学习到在CanMV下控制Kendryte K230D的GPIO输出高低电平。  

## FPIOA模块介绍

有关FPIOA模块的介绍，请见[跑马灯实验的FPIOA模块介绍](led.md#fpioa模块介绍)

## Pin模块介绍

有关Pin模块的介绍，请见[跑马灯实验的Pin模块介绍](led.md#pin模块介绍)

## 硬件设计

### 例程功能

1. 控制板载蜂鸣器间歇发声

### 硬件资源

1. 蜂鸣器 - IO33

### 原理图

本章实验内容，需要控制板载蜂鸣器发声，正点原子DNK230D开发板上蜂鸣器的连接原理图，如下图所示：  

![01](./img/02.png)

通过以上原理图可以看出，蜂鸣器的发声与否由IO33控制，当IO33输出低电平时，蜂鸣器不发声，当IO33输出高电平时，蜂鸣器发声。

## 实验代码

```python
from machine import Pin
from machine import FPIOA
import time

# 实例化FPIOA
fpioa = FPIOA()

# 设置Pin33为GPIO33
fpioa.set_function(33, FPIOA.GPIO33)

# 实例化Pin33为输出
beep = Pin(33, Pin.OUT, pull=Pin.PULL_NONE, drive=7)

while True:
    # 设置蜂鸣器对应的GPIO对象输出对应的高低电平
    beep.value(0)
    time.sleep_ms(1000)
    beep.value(1)
    time.sleep_ms(1000)
```

可以看到，首先通过FPIOA构造函数构造了fpioa对象，然后通过set_function函数为控制蜂鸣器的IO分配了GPIO33的功能，再通过Pin模块的构造函数构造蜂鸣器对象，配置为输出模式并配置驱动能力，最后在一个循环中轮流设置蜂鸣器的GPIO对象输出依次输出高低电平并延时一段时间，从而能听到板载的蜂鸣器间歇地发声。  

## 运行验证

将DNK230D开发板连接CanMV IDE，并点击CanMV IDE上的“开始(运行脚本)”按钮后，可以听到板载的蜂鸣器间歇地发声，这与理论推断的结果一致。
