---
title: 'PWM实验'
sidebar_position: 6
---

# PWM实验

## 前言

本章将介绍machine模块中的PWM类。通过本章的学习，读者将学习到machine模块中PWM类的使用。

## PWM模块介绍

### 概述

K230D内部包含两个PWM硬件模块，每个模块有3个输出通道，模块输出频率可调，但3通道共用，通道占空比独立可调。因此通道0、1、2共用频率，通道3、4、5共用频率。通道输出IO配置参考IOMUX模块。

### API描述

PWM类位于machine模块下

#### 构造函数

```
pwm = PWM(channel, freq, duty=50, enable=False)
```

【参数】

- channel: PWM通道号，取值:[0,5]
- freq: PWM通道输出频率
- duty: PWM通道输出占空比，指高电平占整个周期的百分比，取值:[0,100]，可选参数，默认50
- enable: PWM通道输出立即使能，可选参数，默认False

### duty

```
PWM.duty([duty])
```

获取或设置PWM通道输出占空比

【参数】

- duty: PWM通道输出占空比，可选参数，如果不传参数则返回当前占空比

【返回值】

返回空或当前PWM通道输出占空比

更多用法请阅读官方API手册：

https://developer.canaan-creative.com/k230_canmv/dev/zh/api/canmv_spec.html

## 硬件设计

### 例程功能

1. 创建一个PWM对象，并将PWM通道5与红色LED灯的IO引脚绑定
2. 按下KEY0按键后增加PWM对象输出PWM的占空比
3. 按下KEY1按键后减少PWM对象输出PWM的占空比

### 硬件资源

1. 双色LED

   ​	LEDR - IO59

2. 独立按键

   ​	KEY0按键 - IO2
   
   ​	KEY1按键 - IO5

### 原理图

本章实验内容，主要讲解PWM模块的使用，无需关注原理图。

##  实验代码

``` python
from machine import Pin, PWM
from machine import FPIOA
import time

# 实例化FPIOA
fpioa = FPIOA()

# 为IO分配相应的硬件功能
fpioa.set_function(2, FPIOA.GPIO2)
fpioa.set_function(5, FPIOA.GPIO5)
fpioa.set_function(59,FPIOA.PWM5)

# 构造GPIO对象
key0 = Pin(2, Pin.IN, pull=Pin.PULL_UP, drive=7)
key1 = Pin(5, Pin.IN, pull=Pin.PULL_UP, drive=7)

# 构造PWM对象
pwm0 = PWM(5, 200, duty=50, enable=True)
duty = 50
while True:
    if key0.value() == 0:
        time.sleep_ms(20)
        if key0.value() == 0:
            duty = duty + 10
            while key0.value() == 0:
                pass
    elif key1.value() == 0:
        time.sleep_ms(20)
        if key1.value() == 0:
            duty = duty - 10
            while key1.value() == 0:
                pass
    if duty == 0:
        duty = 10
    elif duty == 110:
        duty = 100
    # 修改PWM占空比
    if pwm0.duty() != duty:
        pwm0.duty(duty)
    time.sleep_ms(10)
```

可以看到，首先是初始化使用到独立按键的IO，并将PWM5与红色LED灯的IO硬件绑定。

接下来构造了一个PWM对象，PWM对象的配置为通道5、输出频率为200Hz、占空比为50%的PWM，并立即使能PWM通道输出。

最后就是在一个循环中读取按键的状态，当读取到KEY0按键被按下，则增加PWM输出的占空比，具体应表现为红色LED的亮度减少，当读取到KEY1按键被按下，则减少PWM输出的占空比，具体应表现为红色LED的亮度增加。

## 运行验证

将DNK230D开发板连接CanMV IDE，并点击CanMV IDE上的“开始(运行脚本)”按钮后，此时，便可看到红色LED处于半亮状态，若按下KEY0按键，则可以看到红色LED的亮度减小，这是因为PWM输出的占空比增加导致的，若按下KEY1按键，则可以看到红色LED的亮度增加，这是因为PWM输出的占空比减小导致的。

