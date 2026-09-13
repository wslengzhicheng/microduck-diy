# 文档总览

本目录包含 Microduck 双足机器人从接线到行走的完整操作文档。

## 文档结构

| 文档 | 内容 | 前置要求 |
| :--- | :--- | :--- |
| [组装指南](build-guide/README.md) | 6 阶段分步组装流程，每阶段含时间估算和检查清单 | 已完成物料采购和 3D 打印 |
| [接线指南](wiring-guide/README.md) | 电源、舵机总线、IMU 的详细接线表格和引脚映射 | 了解基本焊接技能 |
| [调试指南](commissioning/README.md) | 舵机 ID 设置、IMU 检测、冒烟测试和安全检查 | 接线完成，准备上电 |

## 硬件文档

硬件相关文档（BOM、打印、CAD）在 [`hardware/`](../hardware/README.md) 目录下。

## 阅读顺序

建议按照以下顺序阅读：

1. [`hardware/bom`](../hardware/bom/README.md) — 确认物料齐全
2. [`hardware/printing`](../hardware/printing/README.md) — 打印所有结构件
3. [`docs/build-guide`](build-guide/README.md) — 按阶段组装
4. [`docs/wiring-guide`](wiring-guide/README.md) — 完成电气连接
5. [`docs/commissioning`](commissioning/README.md) — 调试验证
6. 上游 README — 刷写镜像并运行

## 照片状态

> [!NOTE]
> `assets/` 目录下的照片位为占位。Allen 拍摄实物后将逐步替换。如果你在装机过程中拍摄了清晰的过程照片，欢迎提交 PR。
