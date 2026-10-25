# CP100 像素现实指南针

CP100 是一个基于 ESP32-S3、GNSS、三轴加速度计/磁力计和 WS2812B 像素灯的 Minecraft 风格现实指南针。

## 目录结构

- `docs/`：产品设计、硬件设计、选型文档，以及 [EasyEDA 云端开发流程](docs/EasyEDA开发流程.md)。
- `design/`：预留给可公开的设计资料；立创/EasyEDA 工程文件不上传到本仓库。
- `outputs/pcb/`：当前 PCB 回读数据和顶/底层预览。
- `outputs/schematic/`：当前原理图预览。
- `verification/`：最后一轮 PCB 和原理图检查结果。
- `assets/`：项目灵感图和参考图片。

## 当前设计状态

- CP100 原理图已拆成三页：保留灯板 P1，新增 `主控电源_IMU` 和 `GNSS_RF` 两张 A4 页。
- 两张新页已完成器件放置、网络标签、地线干线、NC 标记、图签和分区框。
- P2/P3 已完成保存、重载、回读；最终 schematic gate 均通过，官方 DRC 无 fatal/error。
- PCB 仍处于布局完成、未开始正式布线的阶段；异形板框和灯板布局保持不变。

## EasyEDA 工程

- 项目：`MC指南针`
- 项目 UUID：`1be4f1c5e4144fea950b4497cac80aa8`
- PCB 文档 UUID：`f0fcd51a86da5d3d`
- CP100 原理图 UUID：`2cbdd638d241b2f9`

EasyEDA 工程文件只保留在立创 EDA 云端，不上传到 GitHub；仓库保存产品、硬件和验证文档。
