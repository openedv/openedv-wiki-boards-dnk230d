---
title: '呼吸灯实验'
sidebar_position: 6
---

# 呼吸灯实验

## 前言

本章将介绍machine模块中的PWM类。通过本章的学习，读者将学习到machine模块中PWM类的使用。

## PWM模块介绍

有关PWM模块的介绍，请见[蜂鸣器实验的PWM模块介绍](beep.md#PWM模块介绍)

## 硬件设计

### 例程功能

1. 创建一个PWM对象，并将PWM通道5与蓝色LED灯的IO引脚绑定
2. 创建一个GPIO对象，控制红色LED灯关闭
3. 按下KEY0按键后增加PWM对象输出PWM的占空比
4. 按下KEY1按键后减少PWM对象输出PWM的占空比

### 硬件资源

1. 双色LED

   ​	LEDB - IO59

   ​	LEDR - IO61

2. 独立按键

   ​	KEY0按键 - IO34
   
   ​	KEY1按键 - IO35

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
fpioa.set_function(34, FPIOA.GPIO34)
fpioa.set_function(35, FPIOA.GPIO35)
fpioa.set_function(59,FPIOA.PWM5)
fpioa.set_function(61,FPIOA.GPIO61)

# 构造GPIO对象
key0 = Pin(34, Pin.IN, pull=Pin.PULL_UP, drive=7)
key1 = Pin(35, Pin.IN, pull=Pin.PULL_UP, drive=7)
ledr = Pin(61, Pin.OUT, pull=Pin.PULL_NONE, drive=7)

# 构造PWM对象
pwm0 = PWM(5, 200, duty=50, enable=True)
duty = 50
ledr.value(1) # 关闭红色LED灯，防止干扰

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

可以看到，首先是初始化使用到独立按键和红色LED灯的IO，并将PWM5与蓝色LED灯的IO硬件绑定。

初始化红色LED灯的引脚主要是为了控制红色LED灯输出高电平，让其熄灭，防止影响观察蓝色LED灯的变化。

接下来构造了一个PWM对象，PWM对象的配置为通道5、输出频率为200Hz、占空比为50%的PWM，并立即使能PWM通道输出。

最后就是在一个循环中读取按键的状态，当读取到KEY0按键被按下，则增加PWM输出的占空比，具体应表现为蓝色LED的亮度减少，当读取到KEY1按键被按下，则减少PWM输出的占空比，具体应表现为蓝色LED的亮度增加。

## 运行验证

将K230D BOX开发板连接CanMV IDE，并点击CanMV IDE上的“开始(运行脚本)”按钮后，此时，便可看到蓝色LED处于半亮状态，若按下KEY0按键，则可以看到蓝色LED的亮度减小，这是因为PWM输出的占空比增加导致的，若按下KEY1按键，则可以看到蓝色LED的亮度增加，这是因为PWM输出的占空比减小导致的，这样能就完成了一个手动控制的呼吸灯的程序，我们也可以稍微修改下实现一个自动的方式，感兴趣的同学可以自己尝试下。

