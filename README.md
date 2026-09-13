# Microduck DIY 装机文档
![License](https://img.shields.io/badge/License-Apache_2.0-blue)
![Platform](https://img.shields.io/badge/Platform-Raspberry_Pi_Zero_2W-red)
![Servos](https://img.shields.io/badge/Servos-Dynamixel_XL330-orange)
![Language](https://img.shields.io/badge/语言-中文-green)

> **English** — This repository is a Chinese-first DIY build guide for the [Microduck](https://github.com/AI-FanGe/Microduck-build-tutorial) biped robot. It covers the complete bill of materials, 3D printing, wiring, phased assembly, servo/IMU commissioning, and first-walk instructions. Software and firmware live upstream — this repo is **docs only**.

---

Microduck 是一台紧凑型双足行走机器人，使用 Dynamixel XL330 舵机、Raspberry Pi Zero 2W 和强化学习 ONNX 策略实现自主行走。本仓库提供**从零装机到首次行走**的完整中文文档，帮助国内 Maker 按照清晰的步骤完成搭建。

软件/固件/训练代码请参考上游仓库：

- 上游装机教程与镜像：[AI-FanGe/Microduck-build-tutorial](https://github.com/AI-FanGe/Microduck-build-tutorial)
- 机械结构致谢：[microban](https://github.com/Rhoban/microban) · [pollen-robotics/microduck](https://github.com/pollen-robotics/microduck)

> [!CAUTION]
> 本文档中的接线照片和 OpenRB-150 端子丝印标注为**占位**状态，需要 Allen 实拍电路板后替换。在实际接线前，请务必**核对板丝印**，以实物为准。

---

## 快速开始（5 步）

按照下面的顺序完成你的第一台 Microduck：

### 1️⃣ 采购物料

查看 **[物料清单 (BOM)](hardware/bom/README.md)** 获取完整的电子元器件、电源、紧固件和工具列表，包含淘宝/京东/Amazon 参考搜索链接。

### 2️⃣ 3D 打印

阅读 **[打印指南](hardware/printing/README.md)** 了解如何从上游 `.3mf` 文件提取 STL、打印参数和部件清单。

### 3️⃣ 组装与接线

跟随 **[组装指南](docs/build-guide/README.md)** 的 6 个阶段逐步完成机械装配，同时参考 **[接线指南](docs/wiring-guide/README.md)** 完成电源、舵机总线和 IMU 的布线。

### 4️⃣ 调试舵机与 IMU

进入 **[调试指南](docs/commissioning/README.md)** 使用 Dynamixel Wizard 设置舵机 ID 和波特率，运行 `i2cdetect` 验证 IMU，执行首次冒烟测试。

### 5️⃣ 刷写镜像并运行

1. 下载预构建镜像 [`microduck.img.xz`](https://github.com/AI-FanGe/Microduck-build-tutorial/releases/download/image.v1/microduck.img.xz)
2. 使用 Raspberry Pi Imager 或 `dd` 刷写到 microSD 卡
3. 配置 Wi-Fi（编辑 `bootfs/network-config` 或使用 `nmcli`）
4. SSH 登录：
   ```bash
   ssh user@microduck.local   # 默认密码: password
   ```
5. 首次运行（手扶机器人！）：
   ```bash
   cd ~/microduck
   PYTHONPATH=src .venv/bin/python src/main.py
   ```

完整的镜像刷写和 Wi-Fi 配置流程请参考 [上游 README](https://github.com/AI-FanGe/Microduck-build-tutorial#使用-microduckimgxz)。

---

## 文档目录

```
microduck-diy/
├── README.md                        ← 你在这里
├── LICENSE
├── docs/
│   ├── README.md                    ← 文档总览
│   ├── build-guide/
│   │   ├── README.md                ← 分阶段组装指南
│   │   └── assets/                  ← 组装照片（待拍摄）
│   ├── wiring-guide/
│   │   ├── README.md                ← 详细接线指南
│   │   └── assets/                  ← 接线照片（待拍摄）
│   └── commissioning/
│       └── README.md                ← 舵机/IMU 调试
├── hardware/
│   ├── README.md                    ← 硬件总览
│   ├── bom/
│   │   └── README.md                ← 完整物料清单
│   ├── printing/
│   │   └── README.md                ← 3D 打印指南
│   └── cad/
│       └── README.md                ← CAD 参考
```

---

## 致谢

本项目的机械结构、部署思路、仿真训练和实机行走流程均受益于以下开源工作：

- [microban](https://github.com/Rhoban/microban) — 原始机械设计
- [pollen-robotics/microduck](https://github.com/pollen-robotics/microduck) — Microduck 原始项目
- [AI-FanGe/Microduck-build-tutorial](https://github.com/AI-FanGe/Microduck-build-tutorial) — 装机教程、镜像和训练代码

---

## 更新日志

- **2026-09** — 基于 [YouTube 装机视频](https://www.youtube.com/watch?v=Vep8AjoCnEM) 大幅充实文档：组装指南阶段 3–4（舵机预配置 / 零位对齐 / 关节结构 / 逐步装配）、调试指南（OpenRB USB 替代路径 + Center 零位步骤）、接线指南（Y 形分线线束 + 双供电方案对比警告）、BOM（视频淘宝采购快照店铺/标题表）、打印指南（颜色分组 + 脚部修正版说明）、视频截图素材框架。

## 许可证

本仓库文档以 [Apache License 2.0](LICENSE) 发布。机械结构设计版权归 microban / pollen-robotics / AI-FanGe 原作者所有，请参阅各上游仓库的许可协议。
