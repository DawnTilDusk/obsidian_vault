# 昆仑哨兵·实验室多模态监控系统

## 项目概述

昆仑哨兵是一款专为实验室环境设计的多模态监控系统，部署于Orange Pi Kunpeng Pro开发板。系统集成了温度监测、图像采集、气泡检测和数据可视化功能，为实验室提供智能化的环境监测解决方案。

## 系统特性

### 🔧 硬件集成
- **温度采集**：DS18B20数字温度传感器，GPIO2_15接口
- **图像采集**：标准UVC USB摄像头，支持实时图像捕获
- **数据存储**：openGauss 5.0.0数据库，本地化数据持久化

### 🌐 Web界面
- **实时监控**：温度、气泡计数、图像展示
- **数据可视化**：Plotly.js图表，24小时历史趋势
- **开源社区**：静态展示页面，模型共享平台
- **响应式设计**：适配多种显示设备

### 🔧 技术栈
- **后端**：Flask 2.3 + Python 3.9
- **数据库**：openGauss 5.0.0 (PostgreSQL兼容)
- **前端**：HTML5 + CSS3 + JavaScript + Plotly.js
- **图像处理**：OpenCV-Python 4.8
- **传感器**：DS18B20 + UVC摄像头

### PIN
Pin1 有 3.3V 电源功能
Pin2 有 5V 电源功能
Pin3 有 I²C7_SDA / GPIO2_12 / GPIO#76 功能
Pin4 有 5V 电源功能
Pin5 有 I²C7_SCL / GPIO2_11 / GPIO#75 功能
Pin6 有 GND 功能
Pin7 有 UART7_TX / GPIO7_02 / GPIO#226 功能
Pin8 有 UART0_TX / GPIO0_14 / GPIO#14 功能
Pin9 有 GND 功能
Pin10 有 UART0_RX / GPIO0_15 / GPIO#15 功能
Pin11 有 CAN_RX3 / URXD2 / GPIO2_18 / GPIO#82 功能
Pin13 有 GPIO1_06 / GPIO#38 功能
Pin15 有 GPIO2_15 / GPIO#79 功能
Pin16 有 GPIO2_16 / GPIO#80 功能
Pin17 有 3.3V 电源功能
Pin18 有 GPIO0_25 / GPIO#25 功能
Pin19 有 SPI0_MOSI / GPIO2_27 / GPIO#91 功能
Pin20 有 GND 功能
Pin21 有 SPI0_MISO / GPIO2_28 / GPIO#92 功能
Pin22 有 GPIO0_02 / GPIO#2 功能
Pin23 有 SPI0_SCLK / GPIO2_25 / GPIO#89 功能
Pin24 有 SPI0_CS / GPIO2_26 / GPIO#90 功能
Pin25 有 GND 功能
Pin26 有 GPIO2_19 / GPIO#83 功能
Pin27 有 I²C6_SDA 功能（无 GPIO 复用，电压 1.8V）
Pin28 有 I²C6_SCL 功能（无 GPIO 复用，电压 1.8V）
Pin29 有 URXD7 / GPIO7_07 / GPIO#231 功能
Pin30 有 GND 功能
Pin31 有 GPIO2_20 / GPIO#84 功能
Pin32 有 PWM3 / GPIO1_01 / GPIO#33 功能
Pin33 有 GPIO4_00 / GPIO#128 功能
Pin34 有 GND 功能
Pin35 有 GPIO7_04 / GPIO#228 功能
Pin36 有 UTXD2 / CAN_TX3 / GPIO2_17 / GPIO#81 功能
Pin37 有 GPIO0_03 / GPIO#3 功能
Pin38 有 GPIO7_06 / GPIO#230 功能
Pin39 有 GND 功能
Pin40 有 GPIO7_05 / GPIO#229 功能
 📌 此外，Pin8/Pin10 为调试串口 UART0（TX/RX），已与 Micro USB 调试口复用，不建议另作他用。

⚠️ 补充说明：

Pin11/Pin36 默认为 UART2（URXD2/UTXD2），但也可通过设备树切换为 CAN3（CAN_RX3/CAN_TX3）；二者不可同时启用。
Pin27/Pin28 仅有 I²C 功能，不可作 GPIO 使用，且为 1.8V 电平，与其余 3.3V GPIO 不同。
所有标 GPIO 的引脚默认为 3.3V 电平。
全板共 26 个 GPIO 引脚（含复用为 UART/SPI/I²C/PWM 的 pin）。

## 系统架构

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Web浏览器     │    │   Flask应用     │    │   传感器硬件    │
│                 │◄──►│                 │◄──►│                 │
│ - 实时监控      │    │ - API接口       │    │ - DS18B20温度   │
│ - 数据可视化    │    │ - 数据采集      │    │ - UVC摄像头     │
│ - 开源社区      │    │ - 数据库访问    │    │ - Ascend NPU   │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                                │
                                ▼
                       ┌─────────────────┐
                       │  openGauss数据库 │
                       │                 │
                       │ - 传感器数据    │
                       │ - 系统状态      │
                       │ - 模型信息      │
                       └─────────────────┘
```

## 文件结构

```
/home/openEuler/lab_monitor/
├── app.py                  # Flask主应用
├── db_init.sql            # 数据库初始化脚本
├── static/                # 静态文件目录
│   ├── css/
│   │   └── style.css      # 样式文件
│   ├── js/
│   │   └── main.js        # 前端JavaScript
│   └── images/            # 图像存储目录
├── templates/             # HTML模板
│   ├── index.html         # 监控首页
│   └── community.html     # 开源社区页面
├── model/                 # AI模型目录
│   └── bubble_detector.om # NPU模型文件
└── README.md              # 项目说明文档
```

## 快速开始

### 1. 环境准备

确保系统已安装以下组件：
```bash
# Python包
pip install flask opencv-python numpy psycopg2

# 系统服务
systemctl status gaussdb  # openGauss数据库
ls /dev/video0          # 摄像头设备
ls /sys/bus/w1/devices/ # DS18B20设备
```

### 2. 数据库初始化

```bash
# 使用labuser用户连接数据库（openGauss）
gsql -h localhost -p 7654 -U labuser -d lab_monitor

# 执行初始化脚本
\i /home/openEuler/lab_monitor/db_init.sql
```

### 3. 启动应用

```bash
# 进入项目目录
cd /home/openEuler/lab_monitor

# 启动Flask应用
python app.py

# 应用将在 http://localhost:5000 启动
```

### 4. 访问系统

- **监控首页**：http://localhost:5000/

## 模拟数据生成器（C）

为方便在没有真实硬件时进行联调，本仓库新增了 `sensor_data_generator.c`，用于生成模拟温度/光敏/摄像头像素矩阵数据，并按固定间隔通过 HTTP POST 发送到后端 `/api/ingest`。

### 编译（ARM64）

- 原生ARM64 Linux（例如 openEuler / Ubuntu aarch64）：
  - `gcc -O2 -std=c11 sensor_data_generator.c -o sensor_data_generator -lm`
- macOS Apple Silicon (arm64)：
  - `clang -O2 -std=c11 sensor_data_generator.c -o sensor_data_generator -lm`
- 交叉编译（在x86编译ARM64）：
  - `aarch64-linux-gnu-gcc -O2 -std=c11 sensor_data_generator.c -o sensor_data_generator -lm`

### 运行示例

```bash
./sensor_data_generator -h 127.0.0.1 -p 5000 -d device-001 -s 8 -i 1000
```

- 参数说明：
  - `-h` 后端地址（Flask服务地址）
  - `-p` 后端端口（默认 `5000`）
  - `-d` 设备ID（任意字符串）
  - `-s` 像素矩阵尺寸（支持 `8` 或 `16`）
  - `-i` 发送间隔，单位毫秒（默认 `1000ms`）

### 后端接收接口

- 路由：`POST /api/ingest`
- 内容类型：`application/json`
- JSON示例（省略部分pixels）：

```json
{
  "device_id": "device-001",
  "timestamp_ms": 1699999999999,
  "temperature_c": 25.3,
  "light": 512,
  "frame": {
    "width": 8,
    "height": 8,
    "pixels": [[{"r":123,"g":20,"b":200}, ... ], ... ]
  },
  "checksum_frame": "1a2b3c4d"
}
```

- 处理流程：
  - 验证并重建图像，保存到 `static/images/ingest_*.png`
  - 计算并校验 `checksum_frame`（CRC32，覆盖原始像素字节）
  - 使用现有模拟 `detect_bubbles()` 生成 `bubble_count`
  - 写入 `sensor_data(temperature, bubble_count, image_path)`

### 前端联动

- `/api/latest` 与 `/api/history` 会读取 `sensor_data` 表的最新/历史记录；
- 当生成器持续发送数据时，前端温度/气泡趋势会实时更新；摄像头状态按设备探测显示，入库的图片路径可用于后续扩展展示。

- **开源社区**：http://localhost:5000/community

## API接口

### 获取最新数据
```http
GET /api/latest
```

### 获取历史数据
```http
GET /api/history?hours=24
```

### 触发数据采集
```http
POST /api/capture
```

## 硬件配置

### DS18B20温度传感器
- **接口**：GPIO2_15 (40-pin第15号引脚)
- **上拉电阻**：4.7kΩ到3.3V
- **数据读取**：`/sys/bus/w1/devices/28-*/w1_slave`

### UVC摄像头
- **设备路径**：`/dev/video0`
- **分辨率**：640x480
- **格式**：BGR

### NPU模型
- **芯片**：Ascend 310P3
- **模型格式**：OM (Ascend Model)
- **输入形状**：(1, 3, 480, 640)
- **输出**：气泡计数

## 部署指南

### Orange Pi Kunpeng Pro部署

1. **系统环境**
   ```bash
   # 操作系统：openEuler 22.03 LTS SP4
   # 默认用户：openEuler / 密码：openEuler
   ```

2. **目录结构**
   ```bash
   sudo mkdir -p /home/openEuler/lab_monitor
   sudo chown openEuler:openEuler /home/openEuler/lab_monitor
   ```

3. **依赖安装**
   ```bash
   # 安装Python包
   pip3 install flask opencv-python numpy psycopg2
   
   # 确保摄像头权限
   sudo chmod 666 /dev/video0
   
   # 启用DS18B20
   sudo modprobe w1-gpio
   sudo modprobe w1-therm
   ```

4. **数据库配置**
   ```bash
   # 启动openGauss
   sudo systemctl start gaussdb
   
  # 创建数据库和用户（首次运行）
  gsql -d postgres -U openEuler -p 7654
  # 默认密码已调整为更易区分的示例值，亦可通过环境变量覆盖
  CREATE USER labuser WITH PASSWORD 'LabUser@12345';
  CREATE DATABASE lab_monitor OWNER labuser;
   ```

5. **自启动配置**
   ```bash
   # 创建systemd服务
   sudo tee /etc/systemd/system/kunlun-sentinel.service > /dev/null <<EOF
   [Unit]
   Description=Kunlun Sentinel Lab Monitor
   After=network.target gaussdb.service
   
   [Service]
   Type=simple
   User=openEuler
   WorkingDirectory=/home/openEuler/lab_monitor
   ExecStart=/usr/bin/python3 /home/openEuler/lab_monitor/app.py
   Restart=always
   RestartSec=10
   
   [Install]
   WantedBy=multi-user.target
   EOF
   
   # 启用服务
   sudo systemctl enable kunlun-sentinel
   sudo systemctl start kunlun-sentinel
   ```

## 使用说明

### 数据采集
1. **自动采集**：系统每30秒自动更新数据
2. **手动采集**：点击"采集数据"按钮立即采集
3. **数据保存**：所有数据自动保存到openGauss数据库

### 数据可视化
- **温度趋势**：24小时温度变化曲线
- **气泡趋势**：24小时气泡计数变化
- **实时显示**：当前温度和气泡计数

### 开源社区
- **模型展示**：查看社区贡献的AI模型
- **模型信息**：名称、描述、作者、许可证
- **下载功能**：演示下载按钮（实际跳转到GitHub）

## 故障排除

### 常见问题

1. **DS18B20传感器离线**
   ```bash
   # 检查设备
   ls /sys/bus/w1/devices/28-*
   
   # 检查GPIO
   gpio readall
   ```

2. **摄像头无法访问**
   ```bash
   # 检查设备
   ls -la /dev/video0
   
   # 测试摄像头
   fswebcam test.jpg
   ```

3. **数据库连接失败**
   ```bash
   # 检查服务状态
   systemctl status gaussdb
   
   # 测试连接
   psql -h localhost -p 15400 -U labuser -d lab_monitor
   ```

4. **NPU模型加载失败**
   ```bash
   # 检查模型文件
   ls -la /home/openEuler/lab_monitor/model/bubble_detector.om
   
   # 当前版本使用随机数模拟，实际部署需要真实模型
   ```

### 错误处理

系统采用健壮的错误处理机制：
- 传感器离线返回：`{"error": "xxx offline"}`
- 数据库错误返回：`{"error": "database error"}`
- 摄像头错误返回：`{"error": "camera offline"}`

## 开发指南

### 扩展传感器
在`app.py`中添加新的传感器读取函数：
```python
def read_new_sensor():
    try:
        # 传感器读取逻辑
        return {"sensor_value": value}
    except Exception as e:
        return {"error": "sensor offline"}
```

### 添加新模型
1. 训练AI模型
2. 使用ATC工具转换为OM格式
3. 替换`/home/openEuler/lab_monitor/model/`目录下的模型文件
4. 修改`detect_bubbles()`函数调用ACL推理接口

### 前端定制
- 修改`static/css/style.css`调整样式
- 修改`templates/index.html`调整页面布局
- 修改`static/js/main.js`添加交互功能

## 开源协议

本项目采用Apache-2.0协议开源：
- 允许商业使用
- 允许修改和分发
- 需要保留版权声明

## 社区贡献

欢迎提交您的实验室专用模型！

- **GitHub**：github.com/kunlunsentinel/lab-models
- **邮件**：contact@kunlunsentinel.com
- **论坛**：community.kunlunsentinel.com

## 技术支持

如遇到问题，请通过以下方式获取支持：

1. **查看日志**：`journalctl -u kunlun-sentinel -f`
2. **检查配置**：确认所有路径和权限设置
3. **社区求助**：在GitHub提交Issue
4. **文档查阅**：参考项目Wiki文档

---

**昆仑哨兵** - 让实验室监控更智能！

*Built with ❤️ for the openEuler community*