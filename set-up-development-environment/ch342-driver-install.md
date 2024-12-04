---
title: 'CH342驱动安装'
sidebar_position: 1
---

# CH342驱动安装

## 概述

[CH342](https://www.wch.cn/products/CH342.html)是一个USB总线的转接芯片，实现USB转两个异步串口，可以供K230的两个核心同时使用。

CH342的驱动程序可在其官网的[资料下载](https://www.wch.cn/search?q=CH342&t=downloads)页面进行下载，也可以通过[这里](https://www.wch.cn/downloads/CH343SER_EXE.html)下载最新的适用于Windows的一键式安装驱动程序。

## 安装

> 本章以Windows环境为例，介绍使用CH342的一键式安装驱动程序在Windows下安装CH342的驱动程序

双击打开下载好的CH342一键式安装驱动程序。

![ch342 driver installer](./img/ch342-driver-installer.png)

直接点击安装程序中的“安装”按钮，安装会自动安装CH342的驱动程序，并在安装成功后，进行弹窗提示。

![ch342 driver install done](./img/ch342-driver-install-done.png)

CH342驱动程序安装成功后，若正常连接了CH342设备，则能够在Windows的“设备管理器”中看到对应的两个端口设备。

![check ch342 com ports](./img/check-ch342-com-ports.png)

> USB-Enhanced-SERIAL-A CH342连接至K230D BOX的UART0
>
> USB-Enhanced-SERIAL-B CH342连接至K230D BOX的UART4

至此，CH342驱动程序安装完毕。