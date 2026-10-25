# CP100 EasyEDA 云端接入与开发流程

## 1. 工程边界

CP100 的立创 EDA 工程以 EasyEDA 云端项目为唯一设计源。GitHub 仓库只保存设计文档、最终预览、回读数据和检查结果，不保存 `.epro2`、`.epru` 或 EasyEDA 本地工程缓存。

当前云端项目：

- 项目名：`MC指南针`
- 项目 UUID：`1be4f1c5e4144fea950b4497cac80aa8`
- PCB 文档：`CP100`，UUID `aac2d96fd1c54254a72fc9bfe6c8b2be`
- 主原理图页：UUID `85ca39e46377c6d2`
- 灯板页：UUID `fcd2a01486bd4ccf8278cd70504526a1`

2026-10-25 的云端回读基线是：主原理图 44 个 BOM 器件、36 个网络；页面另有网络端口/电源标记等语义图元；PCB 100 个器件，原始 17 点异形板框。PCB 当前完成布局，新增电路仍未进入整板布线阶段。

## 2. 开始工作前

先确认 EasyEDA Pro、连接器和 CLI 版本一致，并确认工程确实在云端可读：

```powershell
easyeda health
easyeda doc ls --project 'MC指南针'
```

`easyeda health` 必须显示 daemon、connector 和 EasyEDA 宿主均为可用状态。项目有多个 EasyEDA 窗口时，所有命令都用 `--project 'MC指南针'`，并进一步用 `--doc <UUID>` 锁定目标文档；不要依赖当前前台标签页。

## 3. 读取真实设计

原理图一次性读取器件、引脚、网络和几何检查：

```powershell
easyeda sch read `
  --project 'MC指南针' `
  --doc 85ca39e46377c6d2
```

PCB 读取器件包络和焊盘：

```powershell
easyeda pcb list `
  --project 'MC指南针' `
  --doc aac2d96fd1c54254a72fc9bfe6c8b2be `
  --include-bbox --include-pads
```

需要检查板框、铜线、过孔和铺铜时，读取完整 PCB 快照：

```powershell
easyeda pcb dump `
  --project 'MC指南针' `
  --doc aac2d96fd1c54254a72fc9bfe6c8b2be `
  --include-copper
```

每一次修改前都保留一份只读快照。快照是证据，不是新的设计源；不要直接编辑 JSON、SVG 或回读文件来“修复”设计。

## 4. 原理图开发流程

1. 先读取当前主原理图，确认真实器件、引脚和网络。
2. 明确供电、接口、机械和模块边界，再生成修改计划。
3. 通过 typed EasyEDA 命令修改，例如 `sch place`、`sch modify`、`sch autoconnect`、`sch connect`；参数以当前 `--help` 为准。
4. 任何批量操作先 dry-run，确认器件 ID、网络名和坐标，再串行执行。
5. 修改后运行：

```powershell
easyeda sch check --project 'MC指南针' --doc 85ca39e46377c6d2
easyeda sch bridge-check --project 'MC指南针' --doc 85ca39e46377c6d2
easyeda sch drc --project 'MC指南针' --doc 85ca39e46377c6d2
easyeda sch layout-lint --project 'MC指南针' --doc 85ca39e46377c6d2
```

6. 确认检查结果后保存、重载，再做一次新鲜回读：

```powershell
easyeda sch save --project 'MC指南针' --doc 85ca39e46377c6d2
easyeda doc reload --project 'MC指南针' --doc 85ca39e46377c6d2
easyeda sch read --project 'MC指南针' --doc 85ca39e46377c6d2
```

保存必须返回 `saved:true`。自动保存、导出图片和离线报告不能代替保存证据。

## 5. PCB 开发流程

### 布局阶段

先锁定原始异形板框和机械边界，再放连接器、天线座、ESP32、GNSS、传感器和电源模块。移动器件时区分 anchor 与 bbox center；需要按包络中心定位时使用 `pcb modify --center`。旋转和移动分两次执行，旋转后重新读取 bbox。

当前 CP100 的机械约束：

- Type-C 位于底边几何中心，开口朝外。
- S1 位于左侧直边、S2 位于右侧直边，侧推方向朝外。
- J2、J3 需要结合实际外壳和连接器插拔方向复核。
- 56 颗顶层像素灯和原始异形板框不能被改成矩形。

布局阶段只允许规则范围内的局部电源连接；不要在用户确认布局前开始整板布线。

### PCB 修改和验证

布局修改使用 typed 命令，例如：

```powershell
easyeda pcb modify --project 'MC指南针' --doc aac2d96fd1c54254a72fc9bfe6c8b2be --id <primitiveId> --center --x <mil> --y <mil>
```

每轮修改后依次检查：

```powershell
easyeda pcb check --project 'MC指南针' --doc aac2d96fd1c54254a72fc9bfe6c8b2be
easyeda pcb layout-lint --project 'MC指南针' --doc aac2d96fd1c54254a72fc9bfe6c8b2be --json
easyeda pcb drc --project 'MC指南针' --doc aac2d96fd1c54254a72fc9bfe6c8b2be --json
```

检查覆盖范围必须分开记录：

- `pcb check`：DFM、悬空线、净距、丝印和连接器机械启发式检查。
- `layout-lint`：封装重叠、越界、紧间距、ratsnest 和可布线性。
- 原生 `pcb drc`：EasyEDA 官方 DRC；`passed:false` 就是未通过。

保存并重载是 PCB 验收的一部分：

```powershell
easyeda pcb save --project 'MC指南针' --doc aac2d96fd1c54254a72fc9bfe6c8b2be
easyeda doc reload --project 'MC指南针' --doc aac2d96fd1c54254a72fc9bfe6c8b2be
easyeda pcb dump --project 'MC指南针' --doc aac2d96fd1c54254a72fc9bfe6c8b2be --include-copper
```

只有保存成功、重载成功、身份一致且新回读通过，才更新仓库中的 `outputs/` 和 `verification/` 文件。

## 6. 预览图和验证结果

预览图是辅助检查，不是 EasyEDA 原生截图，也不能证明生产 DRC 通过：

```powershell
easyeda pcb layout render --project 'MC指南针' --doc aac2d96fd1c54254a72fc9bfe6c8b2be --actual --side top --board <fresh-dump.json> --out outputs/pcb/CP100_top_final_verified.svg
easyeda pcb layout render --project 'MC指南针' --doc aac2d96fd1c54254a72fc9bfe6c8b2be --actual --side bottom --board <fresh-dump.json> --out outputs/pcb/CP100_bottom_final_verified.svg
```

仓库中的 `verification/` 只保留最后一轮验证结果。中间调试输出不应重新放回仓库。

## 7. GitHub 协作边界

提交到 GitHub 的内容：

- 产品和硬件设计文档；
- PCB、原理图最终预览和回读数据；
- 最后一轮 DFM、布局检查和原理图检查结果；
- 必要的灵感图和参考素材。

不提交的内容：

- `.epro2`、`.epru`、EasyEDA 本地工程和登录态数据；
- `.easyeda/` 缓存；
- 中间快照、尝试脚本、重复渲染图和临时日志。

GitHub 是文档和验证结果的协作副本，EasyEDA 云端项目才是原理图和 PCB 的设计源。完成一轮云端修改后，先保存并回读，再更新仓库输出：

```powershell
git add docs outputs verification README.md
git commit -m "Update CP100 EasyEDA design evidence"
git push origin main
```
