# CAD 参考

本页汇总 Microduck 的 CAD 资源。

---

## 上游 CAD 文件

Microduck 的 CAD 文件分布在上游仓库的不同位置：

| 资源 | 位置 | 说明 |
| :--- | :--- | :--- |
| 3D 打印模型 | [`microduck3D打印.3mf`](https://github.com/AI-FanGe/Microduck-build-tutorial) | 包含所有可打印部件的 3MF 文件（含打印方向预设） |
| 爆炸视图 GIF | [`docs/assets/cad-exploded.gif`](https://github.com/AI-FanGe/Microduck-build-tutorial/blob/main/docs/assets/cad-exploded.gif) | 展示各部件装配关系的动画 |
| IMU 安装视图 | [`mjlab_microduck/imu_marker_trunk_views.png`](https://github.com/AI-FanGe/Microduck-build-tutorial/blob/main/mjlab_microduck/imu_marker_trunk_views.png) | IMU 在躯干上的安装位置和方向参考 |

> [!NOTE]
> 上游仓库中暂未发现独立的 `microduck/cad/` 目录存放原始 CAD 源文件（如 STEP / Fusion 360 / SolidWorks 格式）。如需修改设计，可能需要从 `.3mf` 反向导入或联系上游作者获取源文件。

---

## 本仓库 CAD 目录

本仓库为文档仓库，不存放 CAD 二进制文件。如果后续获取到 STEP 或其他可编辑格式的 CAD 文件，将存放在此目录下。

---

## 结构概览

Microduck 是一台 14 自由度（+1 可选嘴部）双足机器人，主要结构部件包括：

- **躯干 (trunk)** — 容纳 Pi、OpenRB-150、电池和 IMU 的主体框架
- **头颈** — 4 个舵机驱动（俯仰、偏航、横滚、颈部俯仰）
- **双腿** — 每条腿 5 个舵机（髋偏航、髋横滚、髋俯仰、膝、踝）
- **脚部** — 提供接地支撑面

爆炸视图动画可在 [上游仓库](https://github.com/AI-FanGe/Microduck-build-tutorial/blob/main/docs/assets/cad-exploded.gif) 查看。
