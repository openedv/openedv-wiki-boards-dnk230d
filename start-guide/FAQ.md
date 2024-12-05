---
title: '常见问题汇总（FAQ）'
sidebar_position: 4
---

# 常见问题汇总（FAQ）

1. K230D BOX上电后背面的电源指示灯亮度很低，是否正常？
   - 是正常的，因为电源指示灯在K230D BOX上电后是常亮的，且与摄像头都位于K230D BOX的背面，为了避免电源指示灯对摄像头成像的影响，所以将电源指示灯的亮度调低。

2. CanMV IDE无法连接K230D BOX?
   - 请确保K230D BOX的USB线连接正常，K230D BOX只有侧边的USB口能通与CanMV IDE进行连接。

3. 通过USB口无法给K230D BOX供电？
   - K230D BOX具有侧边和背面两个USB口，侧边的USB口用于供电和连接CanMV IDE，而背面的USB口仅用于与上位机进行串口通讯的调试作用，背面的USB口无供电功能。

4. K230D BOX运行时，整体温度较高，是否正常？
   - 是正常的，因为K230D BOX上搭载的Kendryte K230D芯片，其采用内部集成了两个RISC-V高能效计算核心和嘉楠的第三代自研KPU，同时因为K230D BOX小巧的体积，散热条件有限，所以整体温度较高。
   - 当然，可以将产品包装盒里附带的散热片贴在K230D芯片上，以帮助K230D散热。
   ![k230d box with heat sink](./img/k230d-box-with-heat-sink.png)