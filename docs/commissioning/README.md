# 调试指南

Microduck 的舵机 ID 设置、IMU 验证、首次冒烟测试和安全检查流程。

> [!IMPORTANT]
> 本指南应在 [组装指南](../build-guide/README.md) 阶段 5 执行，即所有电气连接完成、外壳尚未安装的状态下。

---

## 一、Dynamixel 舵机 ID 设置

### 工具准备

| 工具 | 说明 |
| :--- | :--- |
| ROBOTIS U2D2 | USB 转 Dynamixel 接口 |
| U2D2 Power Hub Board | 为舵机提供外部供电 |
| Dynamixel Wizard 2.0 | ROBOTIS 官方软件，[下载页面](https://emanual.robotis.com/docs/en/software/dynamixel/dynamixel_wizard2/) |
| 6V 电源 | 可使用电池或台式电源 |

### 设置步骤

1. **连接硬件：** U2D2 连电脑 USB → U2D2 连 Power Hub → Power Hub 接 6V 电源 → **单个** XL330 舵机连 Power Hub

> [!CAUTION]
> **一次只连接一个舵机进行 ID 设置。** 出厂时所有 XL330 的 ID 都是 1，如果同时连接多个会导致总线冲突。

2. **打开 Dynamixel Wizard 2.0：**
   - 设置扫描参数：
     - Protocol: **`2.0`**
     - Baud Rate: **`57600`**（出厂默认波特率）
     - 端口: 选择 U2D2 对应的串口
   - 点击 **Scan**，应该能找到 ID=1 的舵机

3. **修改舵机参数：**

   按下表逐个设置每个舵机的 ID 和波特率：

   | 参数 | 出厂值 | 目标值 |
   | :--- | :--- | :--- |
   | **ID** | 1 | 按下表分配 |
   | **Baud Rate** | 57600 | **1,000,000 (1Mbps)** |
   | **Protocol** | 2.0 | 2.0（保持不变） |
   | **Return Delay Time** | 250 | **0** |
   | **PWM Slope** | (默认) | **255** |
   | **Shutdown** | (默认) | **去掉输入电压错误触发项** |

   > 去掉 Shutdown 中的输入电压错误触发，是为了避免电池电压波动时舵机误停。

4. **ID 分配表：**

   | ID | 名称 | 位置 | 所属链路 |
   | :---: | :--- | :--- | :--- |
   | 1 | `right_ankle` | 右踝 | 右腿链 |
   | 2 | `right_knee` | 右膝 | 右腿链 |
   | 3 | `right_hip_pitch` | 右髋俯仰 | 右腿链 |
   | 4 | `right_hip_roll` | 右髋横滚 | 右腿链 |
   | 5 | `right_hip_yaw` | 右髋偏航 | 右腿链 |
   | 6 | `left_ankle` | 左踝 | 左腿链 |
   | 7 | `left_knee` | 左膝 | 左腿链 |
   | 8 | `left_hip_pitch` | 左髋俯仰 | 左腿链 |
   | 9 | `left_hip_roll` | 左髋横滚 | 左腿链 |
   | 10 | `left_hip_yaw` | 左髋偏航 | 左腿链 |
   | 11 | `head_pitch` | 头部俯仰 | 头颈链 |
   | 12 | `neck_pitch` | 颈部俯仰 | 头颈链 |
   | 13 | `head_yaw` | 头部偏航 | 头颈链 |
   | 14 | `head_roll` | 头部横滚 | 头颈链 |
   | 15 | `mouth` | 嘴部（可选） | 头颈链 |

5. **设置完成后验证：**
   - 修改 Dynamixel Wizard 的扫描波特率为 **1Mbps**
   - 将所有已设置好 ID 的舵机连到同一条总线上
   - 扫描应该能找到所有 14 个舵机（ID 1–14）
   - 在 Wizard 中逐个选中舵机，手动拨动轴，确认位置读数变化

> [!TIP]
> 建议在每个舵机壳体上贴标签标注 ID 和位置名称（如 "ID3 右髋俯仰"），避免装配时搞混。

---

## 二、OpenRB-150 串口检查

舵机 ID 设置好并装入机体后，需要通过 OpenRB-150 验证总线通信。

### 在 Pi 上检查

1. SSH 登录到 Pi：
   ```bash
   ssh user@microduck.local
   ```

2. 检查 OpenRB-150 USB 串口是否被识别：
   ```bash
   ls /dev/serial/by-id/
   ```
   应该能看到包含 `ROBOTIS_OpenRB-150` 的设备。如果没有：
   ```bash
   ls /dev/ttyACM*
   ```
   检查是否有 `ttyACM0` 或类似设备。

3. 如果串口设备都看不到：
   - 检查 USB 数据线是否支持数据传输（某些 USB 线仅支持充电）
   - 尝试更换 USB 线或 USB 口
   - 检查 OpenRB-150 电源是否正常

### 通过代码验证

上游代码会按以下顺序自动查找串口：
1. `/dev/serial/by-id/usb-ROBOTIS_OpenRB-150_*`
2. `/dev/ttyACM0`
3. `/dev/serial0`
4. `/dev/ttyAMA0`
5. `/dev/ttyS0`

---

## 三、IMU (BNO08x) 检测

### I2C 总线检测

1. SSH 登录 Pi 后运行：
   ```bash
   sudo apt-get install -y i2c-tools   # 如果未安装
   i2cdetect -y 1
   ```

2. 预期输出中应该在地址 **`0x4a`** 或 **`0x4b`** 处显示设备：
   ```
        0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
   00:                         -- -- -- -- -- -- -- --
   10: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
   20: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
   30: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
   40: -- -- -- -- -- -- -- -- -- -- 4a -- -- -- -- --
   50: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
   60: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
   70: -- -- -- -- -- -- -- --
   ```

3. 如果看不到 BNO08x 地址：
   - 检查 I2C 是否启用：`sudo raspi-config` → Interface Options → I2C → Enable
   - 检查接线：SDA→GPIO2(Pin3)、SCL→GPIO3(Pin5)
   - 检查供电：VIN/3V3 是否接到 Pi 3.3V
   - 检查 GND 是否连接
   - 线缆是否过长（I2C 线缆应 <20cm）

### IMU 数据验证

上游代码提供了 IMU 数据查看命令：

```bash
cd ~/microduck
make imu
```

应该能看到实时的姿态数据输出。轻轻倾斜机器人，确认数值变化正常。

---

## 四、首次冒烟测试

> [!CAUTION]
> 冒烟测试时**必须手扶机器人**，并在平稳桌面上进行。第一次运行不要在桌边或高处。

### 步骤

1. 确认电池电量充足
2. 打开电源开关
3. 等待 Pi 启动（1–3 分钟）
4. SSH 登录：
   ```bash
   ssh user@microduck.local
   ```
5. 手扶机器人，启动控制程序：
   ```bash
   cd ~/microduck
   PYTHONPATH=src .venv/bin/python src/main.py
   ```
6. 观察启动日志：
   - ✅ 舵机扭矩开启
   - ✅ 平滑回到 neutral pose
   - ✅ IMU 数据正常读取
   - ✅ 键盘输入响应

7. 测试基本功能：
   - 按 `i` 查看 IMU 信息
   - 按 `v` 开启行走（手扶！）
   - 按方向键测试前后左右
   - 按 `x` 速度归零
   - 按 `q` 停止

8. 停止后安全关机：
   ```bash
   sudo shutdown -h now
   ```
   等待 10–15 秒后再关闭电源开关。

### 常见问题

| 现象 | 可能原因 | 排查方法 |
| :--- | :--- | :--- |
| 舵机不动 | 电池电压不足 / 总线通信错误 | 检查电池电压、ID 设置、线序 |
| 部分舵机不响应 | ID 设置错误 / 3-pin 线缆松动 | 在 Dynamixel Wizard 中逐个验证 |
| 机器人启动即倒 | 左右腿 ID 装反 / IMU 方向错误 | 检查 ID 对应的物理位置；检查 IMU 安装方向 |
| SSH 连不上 | Wi-Fi 未配置 / Pi 未启动 | 检查路由器分配的 IP；等待充分启动 |
| 通信错误频繁 | 波特率不匹配 / GND 未共地 | 确认所有舵机波特率为 1Mbps；检查共地 |
| IMU 无数据 | I2C 未启用 / 接线错误 | `i2cdetect` 验证；检查 SDA/SCL 线序 |

---

## 五、安全须知

- **第一次运行必须手扶机器人**
- 不要在桌边或高处测试
- 不要在电池低电压时长时间运行
- **不要直接断电** — 先执行 `sudo shutdown -h now` 或手柄安全关机
- 充电和上电前检查短路、极性和焊点绝缘
- 不要在有电状态下焊接或拆焊
- 舵机过热时立即停止运行，检查是否有机械卡死

---

## 六、舵机电压监控

日常使用中可以通过以下命令监控舵机供电电压：

```bash
cd ~/microduck
make voltage          # 查看所有舵机电压
make voltage ID=2     # 查看指定舵机电压
```

当电压低于 5V 时应及时充电，避免低电压导致舵机异常停止或通信错误。
