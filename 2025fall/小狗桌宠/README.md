# SmartDeskPet

  

一个桌面智能小宠物，采用非阻塞状态机架构，同时驱动多路舵机、OLED表情显示与超声波测距，实现“靠近开心、过近害怕、远离空闲并眨眼”的互动效果。

  

- 代码主文件： [SmartDeskPet.ino](file:///c:/Users/Notebook/Desktop/SmartDeskPet/scripts/SmartDeskPet.ino)

- 步态测试： [legs_move.ino](file:///c:/Users/Notebook/Desktop/SmartDeskPet/scripts/legs_move.ino)

- 参考库： [references/Robotics-eyes-on-OLED-display.../README.md](file:///c:/Users/Notebook/Desktop/SmartDeskPet/references/Robotics-eyes-on-OLED-display---Ardiuno-(参考表情开源库）/README.md)

  

## 特性

- 非阻塞（基于状态机与时间片）舵机控制，支持平滑速度与并行动作。

- OLED显示表情（眨眼、左右看、睡觉、醒来、开心、害怕）。

- 超声波传感器测距，基于阈值切换宠物行为（SCARED/HAPPY/IDLE）。

- 启动动画与自动眨眼、随机左右看行为。

  

## 仓库结构

```

SmartDeskPet/

├─ scripts/                # 主控制与步态测试

│  ├─ SmartDeskPet.ino     # 主程序（非阻塞）

│  └─ legs_move.ino        # 简易步态演示（阻塞延时）

├─ references/             # 参考的OLED表情开源库

├─ 3D-models/              # 机身/外壳等3D模型（STL/SLDPRT）

├─ media/                  # 演示视频/图片（建议忽略）

└─ 项目报告_SmartDeskPet_钟煜晨.docx

```

  

## 硬件与连线

- 开发板：Arduino Nano（或同等）

- 舵机：5路（4条腿 + 尾巴）

- OLED：SSD1306 128x64 I2C（地址 0x3C）

- 超声波：HC-SR04

  

引脚映射（详见 [SmartDeskPet.ino: Pin Definitions](file:///c:/Users/Notebook/Desktop/SmartDeskPet/scripts/SmartDeskPet.ino#L27-L36)）：

- Legs：D5(RF)、D6(LF)、D7(RB)、D8(LB)

- Tail：D4

- HC-SR04：Trig -> D9、Echo -> D10

- I2C：SDA -> A4、SCL -> A5（Nano）

  

舵机中位校准（详见 [MID_POS](file:///c:/Users/Notebook/Desktop/SmartDeskPet/scripts/SmartDeskPet.ino#L38-L40)）：

```

LF: 85, RF: 80, LB: 105, RB: 100, Tail: 90

```

根据安装姿态不同，可在代码中调整以获得“站立中位”。

  

## 软件依赖与安装

- Arduino IDE 或 Arduino CLI

- 库：

  - Adafruit GFX Library（通过库管理器搜索安装）

  - Adafruit SSD1306（通过库管理器搜索安装）

  - Servo（IDE自带）

  - Wire（IDE自带）

  

SSD1306地址默认 0x3C（见 [SCREEN_ADDRESS](file:///c:/Users/Notebook/Desktop/SmartDeskPet/scripts/SmartDeskPet.ino#L22-L26)），如屏幕不显示，请检查模块丝印地址是否为0x3C或0x3D。

  

## 运行与烧录

1. 打开 Arduino IDE，加载 `scripts/SmartDeskPet.ino`。

2. 选择开发板与端口（工具 -> 开发板 -> Arduino Nano）。

3. 库安装完成后，点击“编译”，再点击“上传”。

4. 上电：

   - 前2秒显示唤醒动画（仅更新OLED）。

   - 根据距离自动切换状态：

     - 距离 < 6cm：SCARED（下蹲、尾巴快速摆动）。

     - 6–30cm：HAPPY（站立略高、尾巴快速摇摆）。

     - ≥30cm：IDLE（正常站立、慢速摇尾）。

  

单独步态测试：打开 `scripts/legs_move.ino` 上传，可演示左右摆动式步态（该脚本使用阻塞延时）。

  

## 代码架构

- AsyncServo（[SmartDeskPet.ino:48–103](file:///c:/Users/Notebook/Desktop/SmartDeskPet/scripts/SmartDeskPet.ino#L48-L103)）

  - 非阻塞、定时步进控制舵机角度；可设定目标角与速度。

- EyeController（[SmartDeskPet.ino:106–466](file:///c:/Users/Notebook/Desktop/SmartDeskPet/scripts/SmartDeskPet.ino#L106-L466)）

  - OLED绘制与动画状态机：IDLE/BLINK/SLEEPING/WAKEUP/HAPPY/SCARED/LOOK_LEFT/LOOK_RIGHT。

  - 平滑左右注视偏移（简易P控制），随机事件触发（定时眨眼/左右看）。

- SonarSensor（[SmartDeskPet.ino:468–516](file:///c:/Users/Notebook/Desktop/SmartDeskPet/scripts/SmartDeskPet.ino#L468-L516)）

  - 200ms周期测距，pulseIn超时防阻塞；返回“超出量程”时置大值。

- 主循环与行为（[SmartDeskPet.ino:518–638](file:///c:/Users/Notebook/Desktop/SmartDeskPet/scripts/SmartDeskPet.ino#L518-L638)）

  - 启动阶段2秒只更新OLED，保证动画清晰。

  - 根据距离阈值切换状态并触发舵机动作与尾巴摆动策略。

  

## 参数与校准

- 中位角：`MID_POS[]`（腿部/尾巴）根据机械实际安装调整。

- 速度与步幅：`AsyncServo.move(target, speed)` 中的 `speed`（越小越快），步进大小在类内 `_stepSize` 可按需修改。

- 眨眼/注视频率：EyeController中各`_lastUpdate`间隔与随机事件区间可微调。

- HC-SR04量程与超时：`pulseIn(..., 6000)`约对应~1m；可按需要放宽或收紧。

  

## 常见问题

- OLED无显示：检查I2C接线、供电与地址是否为0x3C；确保库版本匹配。

- 舵机抖动/重启：请使用独立5V舵机供电并共地；避免USB直接供电多个舵机。

- 超声测距为0：可能为角度/材质导致回波弱；适当增大超时或调整安装角度。

- 机械干涉：调节中位与动作幅度，避免卡死与超过舵机行程。

  

## 扩展建议

- 增加更多表情或眼睛动画（例如生气、困惑）。

- 引入红外或光线传感器，丰富互动逻辑。

- 将步态控制改为更复杂的相位节拍器或逆解算法。

  

## 参考与致谢

- OLED表情开源参考： [references](file:///c:/Users/Notebook/Desktop/SmartDeskPet/references/Robotics-eyes-on-OLED-display---Ardiuno-(参考表情开源库）)

  

## 许可

- 若无特殊说明，默认以MIT或Apache-2.0开源；可按项目需求调整。