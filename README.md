# CP100 像素现实指南针

CP100 是一个基于 ESP32-S3、GNSS、三轴加速度计/磁力计和 WS2812B 像素灯的 Minecraft 风格现实指南针。

## 目录结构

- `docs/`：产品设计、硬件设计、定位模块和 ESP32 天线选型文档。
- `design/easyeda/`：EasyEDA 基线工程包；当前设计仍以 EasyEDA 云工程为主。
- `outputs/pcb/`：当前 PCB 回读数据和顶/底层预览。
- `outputs/schematic/`：当前原理图预览。
- `verification/`：最后一轮 PCB 和原理图检查结果。
- `assets/`：项目灵感图和参考图片。

## 当前设计状态

- 保留现有像素指南针异形板框，四层板布局已完成。
- Type-C 位于底边几何中心；S1/S2 分别位于左右侧边并朝外。
- 底层临时 GND/VDD stub 已清理；只保留顶层灯板的 99 段既有电源走线。
- 新增电路尚未完成 PCB 布线、铺铜和生产级 DRC，当前输出用于继续设计和审核。

## EasyEDA 工程

- 项目：`MC指南针`
- 项目 UUID：`1be4f1c5e4144fea950b4497cac80aa8`
- PCB 文档 UUID：`aac2d96fd1c54254a72fc9bfe6c8b2be`
- 原理图文档 UUID：`85ca39e46377c6d2`

`MC指南针_before_design.epro2` 是本地保留的基线工程包，不代表当前云端设计的最终源文件；最终云端设计需要在 EasyEDA 中打开对应项目继续编辑。
