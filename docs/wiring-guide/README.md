# 接线指南

Microduck 双足机器人的完整接线文档。本指南覆盖电源系统、Dynamixel 舵机总线和 IMU 传感器的所有电气连接。

> [!CAUTION]
> 本文档中的 OpenRB-150 端子丝印标注为**占位状态**。在实际接线前，请**核对板丝印**（PCB silk screen），以实物印刷为准。照片将在 Allen 拍摄实物后补充到 `assets/` 目录。

---

## 接线方案概述

Microduck 采用以下硬件组合：

| 组件 | 型号 | 职责 |
| :--- | :--- | :--- |
| 主控制器 | Raspberry Pi Zero 2W | 运行 ONNX 行走策略、读取 IMU、处理手柄/键盘输入 |
| 舵机控制板 | ROBOTIS OpenRB-150 | 通过 USB 连接 Pi，管理 Dynamixel 舵机总线通信 |
| 舵机 | Dynamixel XL330-M288-T ×14 | 3-pin 串联总线，由 OpenRB-150 供电和通信 |
| 姿态传感器 | BNO08x (BNO080/085/086) | I2C 连接到 Pi，提供实时姿态四元数 |

与 Sesame 的 ESP32 + PWM 舵机方案不同，Microduck 使用 **Dynamixel 总线协议**，接线更简洁——所有舵机通过 3-pin 线缆串联，不需要为每个舵机单独走信号线。

---

## 一、电源接线

### 电源链路总览

```
成品 6V 可充电电池
        │
   ┌────┴────┐
   │  电源开关  │
   └────┬────┘
        │
   ┌────┴──────────────────────────┐
   │                                │
   ▼                                ▼
┌──────────────────┐     ┌──────────────┐
│  OpenRB-150      │     │  5V 稳压模块  │
│  电源输入         │     │  (DC-DC 降压) │
│  (VIN / GND)     │     └──────┬───────┘
│  ⚠️ 核对板丝印    │            │
└──────────────────┘            ▼
        │                ┌──────────────┐
        ▼                │ Pi Zero 2W   │
┌──────────────────┐     │ 5V (Pin 2/4) │
│  XL330 舵机总线   │     │ GND (Pin 6)  │
│  (3-pin 供电)     │     └──────────────┘
└──────────────────┘
        │
  Pi ←──┤ USB 数据线 ──→ OpenRB-150
        │
   （两侧必须共地）
```

### 电源连接表

| 连接 | 起点 | 终点 | 线规 | 说明 |
| :--- | :--- | :--- | :--- | :--- |
| 电池正极 → 开关 | 电池 + | 开关 COM | 22AWG | 红色线 |
| 开关 → OpenRB VIN | 开关 NO | OpenRB-150 VIN（⚠️ 核对板丝印） | 22AWG | 电池经开关到 OpenRB 电源输入 |
| 电池负极 → OpenRB GND | 电池 − | OpenRB-150 GND（⚠️ 核对板丝印） | 22AWG | 黑色线 |
| 电池正极 → 稳压模块 IN+ | 电池 + (开关后) | 稳压模块输入正极 | 22AWG | 从开关后取电 |
| 电池负极 → 稳压模块 IN− | 电池 − | 稳压模块输入负极 | 22AWG | 共地 |
| 稳压模块 OUT+ → Pi 5V | 稳压输出 5V | Pi GPIO Pin 2 或 Pin 4 (5V) | 22AWG | 确认稳压输出为 5.0-5.1V |
| 稳压模块 OUT− → Pi GND | 稳压输出 GND | Pi GPIO Pin 6 (GND) | 22AWG | 共地 |
| Pi USB → OpenRB USB | Pi Micro-USB (数据口) | OpenRB-150 USB | 数据线 | USB 仅用于串口数据通信 |
| **共地** | Pi GND | OpenRB-150 GND | 22AWG | **关键！两侧必须共地** |

> [!WARNING]
> **共地是最容易遗漏的步骤。** Pi 的 GND 和 OpenRB-150 的 GND 必须物理连接。如果通过 USB 数据线的 GND 已经连通，也可以，但建议额外焊一根 GND 线作为保险。

### 上电前检查清单

- [ ] 万用表测量电池极性，确认正负极
- [ ] 稳压模块空载输出调到 5.0–5.1V
- [ ] OpenRB-150 VIN/GND 接线方向正确（⚠️ 核对板丝印）
- [ ] Pi 5V/GND 接线方向正确（Pin 2 = 5V, Pin 6 = GND）
- [ ] Pi GND 和 OpenRB-150 GND 共地
- [ ] 所有焊点已用热缩管或热熔胶绝缘
- [ ] 电源开关功能正常（ON 通电 / OFF 断电）

---

## 二、Dynamixel 舵机总线接线

### XL330 3-pin 线序

Dynamixel XL330 使用 3-pin 连接器，线序如下：

| 引脚 | 信号 | 说明 |
| :---: | :--- | :--- |
| 1 | **GND** | 地线 |
| 2 | **VDD** | 电源（由 OpenRB-150 通过总线供电，约 5-6V） |
| 3 | **DATA** | 半双工串行数据线 |

> [!TIP]
> XL330 每个舵机有**两个 3-pin 端口**（IN 和 OUT），用于菊花链串联。线缆方向不影响通信，但要确保插头方向正确，不要反插。

### 舵机链路拓扑

OpenRB-150 的 Dynamixel 端口通过分线器分出三条总线链：

```
OpenRB-150 Dynamixel Port
            │
       ┌────┴────┐
       │  分线器   │
       └─┬──┬──┬─┘
         │  │  │
         ▼  ▼  ▼
   右腿链  左腿链  头颈链
```

### 舵机 ID 映射表

每条链上的舵机按机械布线顺序串联，但 **ID 必须严格匹配下表**：

#### 右腿链 (5 个舵机)

| 链路顺序 | ID | 名称 | 位置 |
| :---: | :---: | :--- | :--- |
| 1 | 5 | `right_hip_yaw` | 右髋偏航 |
| 2 | 4 | `right_hip_roll` | 右髋横滚 |
| 3 | 3 | `right_hip_pitch` | 右髋俯仰 |
| 4 | 2 | `right_knee` | 右膝 |
| 5 | 1 | `right_ankle` | 右踝 |

#### 左腿链 (5 个舵机)

| 链路顺序 | ID | 名称 | 位置 |
| :---: | :---: | :--- | :--- |
| 1 | 10 | `left_hip_yaw` | 左髋偏航 |
| 2 | 9 | `left_hip_roll` | 左髋横滚 |
| 3 | 8 | `left_hip_pitch` | 左髋俯仰 |
| 4 | 7 | `left_knee` | 左膝 |
| 5 | 6 | `left_ankle` | 左踝 |

#### 头颈链 (4 个舵机)

| 链路顺序 | ID | 名称 | 位置 |
| :---: | :---: | :--- | :--- |
| 1 | 12 | `neck_pitch` | 颈部俯仰 |
| 2 | 11 | `head_pitch` | 头部俯仰 |
| 3 | 13 | `head_yaw` | 头部偏航 |
| 4 | 14 | `head_roll` | 头部横滚 |

> [!NOTE]
> 可选嘴部舵机使用 **ID 15**，当前行走策略不使用该舵机。

### Y 形分线线束（视频作者做法）[14:03–16:24]

> **视频参考：** 以下 Y-splice 线束和独立供电方案来自 [Microduck biped build / Dynamixel XL330](https://www.youtube.com/watch?v=Vep8AjoCnEM) [14:03–16:24]。**这是视频中演示的做法**，与上面的分线器方案二选一。

标准 Dynamixel 菊花链无法从一个端口分出两条腿的链路。视频作者的做法是制作自定义 Y 形分线线束 [14:27]：

1. 取两根 Dynamixel 3-pin 线缆，在一端剪掉连接器
2. 剥开三根线的绝缘层约 5mm
3. 将两根线缆中**同色线**对应扭绞：GND↔GND、Data↔Data、VDD↔VDD
4. 焊接并用热缩管逐根绝缘
5. 未剪的那端连接到 OpenRB-150 或上一级分线点；两个剪开端分别连接右腿链和左腿链

> [!CAUTION]
> **绝对不要交叉 Data 和 GND！** [14:32] 接反会导致通信失败并可能损坏舵机。焊接前仔细确认线色对应关系。

<!-- TODO: 将以下视频截图放入 docs/assets/video-stills/ 后取消注释
![Y 形分线线束 ~14:27](assets/video-stills/14m27_y_splice.jpg)
-->

📷 *[截图待放入：`14m27_y_splice.jpg` — Y 形分线线束焊接实拍 ~14:27]*

下图为视频中展示的 3-pin 接线正确/错误对比（GND / V / Data 必须一一对应，不得交叉）：

<!-- TODO: 将以下视频截图放入 docs/assets/video-stills/ 后取消注释
![3-pin 正确/错误接法 ~14:32](assets/video-stills/14m32_wiring_correct_incorrect.jpg)
-->

📷 *[截图待放入：`14m32_wiring_correct_incorrect.jpg` — 正确接法（上）vs 错误接法（下）~14:32]*

---

### 「视频作者做法」独立供电方案 [15:44–16:24]

> [!WARNING]
> **以下为视频中演示的供电方案，与本仓库默认框图不同。** 两种方案各有优劣，**请只选择一种**，不要混用。

视频作者将舵机电源线（VDD）从 OpenRB-150 总线中分离出来，独立接电池：

```
「视频作者做法」供电拓扑
================================

             ┌──── GND ──→ OpenRB-150 "G" 端子
舵机总线 ────┼──── Data ──→ OpenRB-150 "D" 端子
             └──── VDD ──→ 直接接电池正极 (⚠️ 不经过 OpenRB)

电池 ──→ DC-DC (6.4V→5V) ──→ Pi Zero 2W
```

> [!CAUTION]
> **15 个舵机同时运行的峰值电流可超过 OpenRB-150 板载稳压器的承载上限。** 视频作者 [15:44] 明确指出：如果将 15 个舵机的 VDD 全部经由 OpenRB-150 供电，板子可能烧毁。因此他将 VDD 直接连接电池正极，仅让 GND 和 Data 经过 OpenRB-150。
>
> **如果你使用此方案，务必注意：**
> - OpenRB-150 的 "G"/"D" 端子丝印可能因批次不同，**焊接前核对实物丝印**
> - VDD 线要用足够粗的线材（≥22AWG），焊点必须牢靠
> - 电池侧需要在 VDD 线上串联保险丝或可恢复保险（PTC）防止短路

#### 「本仓库默认 / 上游 README 框图」

本仓库其余文档中描述的默认方案是：

```
「本仓库默认」供电拓扑
================================

电池 → 开关 → OpenRB-150 VIN/GND
                  │
                  ├──→ Dynamixel 总线 (VDD/Data/GND 全部经由 OpenRB)
                  │
电池 → 开关 → 稳压模块 → Pi Zero 2W 5V/GND
```

> [!IMPORTANT]
> **两种方案只能选一种，不能混用。** 选定后在实际接线前核对你的 OpenRB-150 版本丝印和电流承载规格。如果你的舵机数量 ≤14 且电池电压在安全范围内，默认方案更简洁；如果你使用 15 个舵机（含嘴部）或电池电压偏高，视频作者的独立供电方案更安全。

### 舵机链路注意事项

- 链路中的物理线缆顺序可以按机械结构方便来布，但 **每个舵机的 ID 必须通过 Dynamixel Wizard 预先设置好**
- 如果总线出现通信错误，优先检查：
  1. OpenRB-150 是否正确枚举（USB 串口可见）
  2. 电池电压是否正常（6V 电池不应低于 5V）
  3. GND 是否连通
  4. 线序和插头方向
  5. 舵机 ID 是否重复

---

## 三、IMU 接线 (BNO08x)

### 连接表

BNO08x 模块通过 I2C 连接到 Raspberry Pi：

| IMU 模块引脚 | Pi GPIO | Pi 物理引脚 | 说明 |
| :--- | :--- | :---: | :--- |
| `VIN` 或 `3V3` | 3.3V | Pin 1 | 按模块板上标识选择。大多数分线板使用 VIN，内部降压 |
| `GND` | GND | Pin 9 (或其他 GND) | 地线 |
| `SDA` | GPIO2 (I2C1 SDA) | **Pin 3** | I2C 数据线 |
| `SCL` | GPIO3 (I2C1 SCL) | **Pin 5** | I2C 时钟线 |

### Pi Zero 2W 40-pin 相关引脚速查

| 物理引脚 | GPIO | 功能 | 本项目用途 |
| :---: | :--- | :--- | :--- |
| 1 | 3.3V | 电源输出 | IMU VIN/3V3 |
| 2 | 5V | 电源输入 | 稳压模块 → Pi 供电 |
| 3 | GPIO2 | I2C1 SDA | IMU SDA |
| 4 | 5V | 电源输入 | 稳压模块 → Pi 供电（备用） |
| 5 | GPIO3 | I2C1 SCL | IMU SCL |
| 6 | GND | 地 | 稳压模块 GND / IMU GND |
| 9 | GND | 地 | IMU GND（备用） |
| 14 | GND | 地 | 共地备用 |

### IMU 注意事项

- 软件默认使用 I2C bus **`1`**，自动检测 BNO08x 常见地址（0x4A / 0x4B）
- IMU 安装方向由代码中的 `IMU_MOUNT_QUAT` 配置，已匹配当前行走模型
- 安装位置参考上游的 [`imu_marker_trunk_views.png`](https://github.com/AI-FanGe/Microduck-build-tutorial/blob/main/mjlab_microduck/imu_marker_trunk_views.png)
- I2C 线缆保持短距离（<20cm），过长会导致信号质量下降
- 如果 IMU 地址冲突，可通过模块上的 ADR 跳线切换

> [!TIP]
> 安装 IMU 前，先在 Pi 上运行 `i2cdetect -y 1` 确认能看到 BNO08x 的地址（通常 `0x4a` 或 `0x4b`），再固定到躯干内。

---

## 四、完整接线检查清单

完成所有接线后，**上电前**逐项检查：

### 电源系统
- [ ] 电池极性正确
- [ ] 稳压模块输出 5.0–5.1V
- [ ] OpenRB-150 VIN/GND 接线正确（⚠️ 核对板丝印）
- [ ] Pi 5V/GND 接线正确
- [ ] 共地连接完成

### 舵机总线
- [ ] 所有 XL330 3-pin 线缆插接牢固，方向正确
- [ ] 右腿链 5 个舵机 ID: 5→4→3→2→1
- [ ] 左腿链 5 个舵机 ID: 10→9→8→7→6
- [ ] 头颈链 4 个舵机 ID: 12→11→13→14
- [ ] 分线器连接到 OpenRB-150 Dynamixel 端口

### IMU
- [ ] BNO08x SDA → Pi Pin 3 (GPIO2)
- [ ] BNO08x SCL → Pi Pin 5 (GPIO3)
- [ ] BNO08x VIN/3V3 → Pi Pin 1 (3.3V)
- [ ] BNO08x GND → Pi GND

### 绝缘与安全
- [ ] 所有焊点已绝缘（热缩管或热熔胶）
- [ ] 线束整理扎带固定，无裸露导体
- [ ] 无线缆被夹在机械结构中
- [ ] USB 数据线不受力

---

## 五、照片占位

> [!NOTE]
> 以下照片需要 Allen 拍摄实物后补充：

| 照片 | 文件名 | 状态 |
| :--- | :--- | :--- |
| OpenRB-150 电源端子特写（含丝印标注） | `assets/openrb-power-terminals.jpg` | 📷 待拍摄 |
| 舵机总线分线接法 | `assets/servo-bus-splitter.jpg` | 📷 待拍摄 |
| IMU 安装位置 | `assets/imu-mounting.jpg` | 📷 待拍摄 |
| Pi Zero 2W GPIO 接线 | `assets/pi-gpio-wiring.jpg` | 📷 待拍摄 |
| 完成接线全貌 | `assets/wiring-complete.jpg` | 📷 待拍摄 |
| 电源开关安装 | `assets/power-switch.jpg` | 📷 待拍摄 |
| Y 形分线线束焊接 | `assets/y-splice-harness.jpg` | 📷 待拍摄 |

---

## 参考来源

| 来源 | 时间范围 | 对应章节 |
| :--- | :--- | :--- |
| [Microduck biped build / Dynamixel XL330](https://www.youtube.com/watch?v=Vep8AjoCnEM) | 14:03–14:32 | Y 形分线线束制作 |
| 同上 | 14:32–16:24 | 「视频作者做法」独立供电方案 + 电流警告 |
