---
title: '固件烧录'
sidebar_position: 2 
---

# 固件烧录

## 前言

K230D BOX在出厂前默认都会在其内部的SD NAND上提前烧录好CanMV固件，用户无需烧录固件即可直接上手使用。

一般情况下，用户无需主动固件烧录，只有在需要更新固件或需要从TF卡启动等情况时，才需要进行固件烧录。

## 固件说明

用户可从K230D BOX的资料盘中获取编译好的CanMV固件，存放CanMV固件的目录路径为`资料盘/6，软件资料/软件/CanMV固件/`。

> 目录下存放的是固件的压缩包，需要解压后使用

在上述路径中，存放了两类CanMV固件，固件的描述如下：

| 固件                                                         | 描述                     |
| ------------------------------------------------------------ | ------------------------ |
| CanMV-K230D_ATK_DNK230D_micropython\_\{canmv_revision\}\_nncase\_\{nncase_version\}.img | CanMV固件（SD NAND启动） |
| CanMV-K230D_ATK_DNK230D_micropython\_\{canmv_revision\}\_nncase\_\{nncase_version\}\_tf.img | CanMV固件（TF卡启动）    |

## 固件烧录工具

本教程使用的K230固件烧录工具为**K230BurningTool**，软件是嘉楠科技官方提供的用于K230固件烧录的图形化软件，有Windows和Linux两种版本可供选择，可以通过[这里](https://kendryte-download.canaan-creative.com/developer/common/K230BurningTool-v2.0.0/)进行下载。

### 安装

> 本章以Windows环境为例，介绍使用K230BurningTool的安装

在[下载页面](https://kendryte-download.canaan-creative.com/developer/common/K230BurningTool-v2.0.0/)下载**K230BurningTool-Windows-v2.0.0-0-g0c27e7f.zip**，解压后即可直接使用，无需安装。

### 使用

嘉楠科技官方提供了K230BurningTool的使用说明，可以通过[这里](https://kendryte-download.canaan-creative.com/developer/common/K230BurningTool-v2.0.0/K230BurningTool.pdf)下载。

## 固件烧录

>  对K230进行固件烧录前，需先设置其进入BootROM模式，K230D BOX进入BootROM模式的方式如下
>
>  **确保设备没有插入TF卡 --> 设备上电 --> 按住KEY2不放 --> 单击复位按键 --> 松开KEY2**
>
>  若K230D BOX的UART0输出“**boot failed with exit code 19**”，则说明K230D BOX已经成功进入BootROM模式，否则请重试上述步骤。
>
>  若需要将固件烧录至TF卡中，则在进入BootROM模式后，在将TF卡插入K230D BOX。
>
>  > K230D BOX的UART0输出可通过板载的CH342，并借助相应的上位机软件进行观察。
>  >
>  > CH342的驱动安装教程，请见[CH342驱动安装](../set-up-development-environment/ch342-driver-install)。

### 1. 打开K230BurningTool.exe

双击K230BurningTool安装目录下的`bin/K230BurningTool.exe`文件打开烧录软件

![k230 burning tool](./img/k230-burning-tool.png)

### 2. 选择固件文件

请参考[固件说明](#固件说明)在"镜像文件"一栏中选择正确的固件文件。

> 若需要烧录固件至K230D BOX板载的SD NAND中，则请选择从SD NAND启动的固件
>
> 若需要烧录固件至外置的TF卡中，则请选择从TF卡启动的固件

**选择错误的固件可能导致设备运行异常，甚至可能导致不可逆的硬件损坏。**

### 3. 选择目标介质

在“目标介质”下拉列表中选择对应的目标介质。

> | 目标介质 | 说明                                                 |
> | -------- | ---------------------------------------------------- |
> | EMMC     | K230的MMC0连接的设备，对应K230D BOX板载的**SD NAND** |
> | SD Card  | K230的MMC1连接的设备，对应K230D BOX外置的**TF卡**    |
> | 其他     | K230D BOX未使用                                      |

### 4. 开始烧录

> **固件烧录会损坏目标介质中的数据，如有需要，请在开始烧录前，做好目标介质中数据的备份工作。**

配置好“镜像文件”和“目标介质”后，其余的配置项保持默认，点击“开始”按钮后，即可开始对K230D BOX进行固件烧录。

固件烧录时，软件底部的信息框会显示固件烧录进度

![k230 burning tool flashing](./img/k230-burning-tool-flashing.png)

固件烧录成功后，软件底部的信息框会显示固件烧录结果

![k230 burning tool flash done](./img/k230-burning-tool-flash-done.png)

### 5. 运行固件

K230D BOX复位后默认从板载的SD NAND启动，

若需要从TF卡启动，请将正确烧录好固件的TF卡插入K230D BOX，并在KEY2按下时，单击复位按键，即可从TF卡启动固件。
