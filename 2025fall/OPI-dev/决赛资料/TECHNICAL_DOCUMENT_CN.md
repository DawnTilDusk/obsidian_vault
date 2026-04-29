# 昆仑哨兵·实验室多模态监控系统 技术文档

## 1. 概述
- 面向实验室多模态采集、智能分析与可视化的一体化方案，运行于鲲鹏平台（openEuler + openGauss）。
- 支持温度、光敏、图像三类数据采集与周期模型输出，提供实时（SSE）与历史融合展示。

## 2. 架构总览
```
┌──────────────┐       ┌──────────────┐       ┌──────────────┐
│   浏览器     │◄────►│   Flask后端  │◄────►│  openGauss   │
│ 仪表盘/DB页  │  REST │  REST + SSE  │  SQL  │  数据存储     │
└──────────────┘       └──────────────┘       └──────────────┘
         ▲                        ▲
         │                        │
         │               ┌─────────────────────┐
         │               │  模型管理器（周期） │
         │               │  stdout→UDP         │
         │               └─────────────────────┘
         │                        ▲
         ▼                        │
┌───────────────────────┐   ┌───────────────────────┐
│  采集器（C，温/光/图）│→UDP→│  中转（UDP→入库→通知） │
└───────────────────────┘   └───────────────────────┘
             ▲                           │
             └──────────SSE事件──────────┘
```

## 3. 数据流与处理
- 采集通道：
  - 采集器按固定周期（图像约 1s）输出 JSON（温度/光敏/设备信息；图像为两种路径：`image_path` 或 `frame{width,height,pixels}`）→ UDP 中转。
  - 中转补全图像：优先使用 fswebcam 生成 JPG 并上报 `image_path`；回退时将像素矩阵就地缩放并落盘为 PNG → 写入分表（`temperature_data`、`image_data`、`light_data` 或兼容的 `sensor_data`），并通知后端；后端通过 SSE 推送前端。
- 模型通道：
  - 模型脚本周期 `print(JSON)` → 模型管理器读取 stdout → UDP 中转。
  - 中转写入 `model_outputs` 并通知后端，前端展示模型卡与数据库查询。
- 图片生命周期：TTL 定时删除，防止磁盘膨胀（仅删除文件，不删除数据库记录，保留审计元数据）；空闲时段自动抓取心跳帧保证展示连续性（`IDLE_IMAGE_SEC` 控制）。

## 4. 模块职责
- 前端：
  - 仪表盘：温度趋势（ECharts）与状态心跳；SSE 实时刷新，页面可见性优化（不可见时关闭 SSE，恢复时重连；失败回退 5s 轮询）。
  - 开源社区：模型/脚本卡片展示与下载；支持示例假数据渲染。
  - 数据库页：表单化查询（温度/光敏/图像/模型输出），分页与清理控制。
- 后端：
  - REST/SSE：`/api/latest`、`/api/events`、`/api/history`。
  - 采集/中转通知：`/api/capture`、`/api/relay_notify`、`/api/ingest`。
  - 模型管理：`/api/models*`、`/api/model_output`。
  - 数据库页：`/api/db/tables`、`/api/db/query`、`/api/db/clear`。
- UDP 中转：统一 JSON 接入、图像落盘与缩放、分表（`temperature_data`/`image_data`/`light_data`）与兼容表入库、后端通知。
- 采集器：原生 C 进程采样温度/光敏/图像，按统一 JSON 通过 UDP 上报。
- 模型管理：周期运行脚本，读取 stdout 的 JSON 行，以 UDP 上报并维护状态。

## 5. 数据库设计
- 表结构（核心）：
  - `sensor_data(id BIGSERIAL, timestamp TIMESTAMPTZ, temperature REAL, image_path TEXT, light INT, created_at TIMESTAMPTZ)`
  - `model_outputs(id BIGSERIAL, name TEXT, output TEXT, created_at TIMESTAMPTZ)`
- 设计原则：幂等建表与列兼容（缺失列自动补齐），简化写入与查询；视图与索引按需扩展。
 - 后端数据库：采用 openGauss 作为生产级存储，启用 md5 认证与连接池，兼容 PostgreSQL 生态；端口默认 7654，支持 py-opengauss 优先连接、psycopg2 回退。

## 6. API 说明
- 实时与历史：
  - `GET /api/latest`：返回最新聚合状态/数据。
  - `GET /api/events`：SSE 事件流（增量推送）。
  - `GET /api/history?hours=<n>`：温度历史数据（用于趋势图）。
- 采集与中转：
  - `POST /api/capture`：触发一次采集（温度+图像）。
  - `POST /api/ingest`：外部数据接入（温度/光敏/像素矩阵）。
  - `POST /api/relay_notify`：中转通知后端刷新缓存与心跳。
- 模型管理：
  - `GET /api/models`、`POST /api/models/command`、`POST /api/models/notify`、`GET /api/models/download/<name>`。
  - `POST /api/model_output`：模型输出直传入库（中转会调用）。
- 数据库页：
  - `GET /api/db/tables`、`POST /api/db/query`、`POST /api/db/clear`。
  - `GET /api/db/table?name=<table>&hours=<n>`：分表查询（支持 `image_data`、`temperature_data`、`light_data` 等）。

## 7. 配置与环境变量
- 通用：`LAB_DIR`（默认 `/home/openEuler/lab_monitor`）、`FLASK_PORT`（默认 5000）。
- 数据库：`DB_HOST`、`DB_PORT`（默认 7654）、`DB_NAME`、`DB_USER`、`DB_PASSWORD`。
- 中转：`RELAY_HOST`（默认 127.0.0.1）、`RELAY_PORT`（默认 9999）、`IMAGE_TTL_SEC`（默认 600）、`IDLE_IMAGE_SEC`（start 默认 10；relay 默认 15，可统一通过环境变量）。
- 摄像头：`CAMERA_DEVICE`（默认 `/dev/video0` 或探测）。
- 光敏：
  - `LIGHT_GPIO`（sysfs GPIO 号）、`LIGHT_GPIO_ACTIVE_HIGH`（1/0）。
  - `LIGHT_GPIO_GROUP` 与 `LIGHT_GPIO_PIN`（配合 `gpio_operate`）。
- 模型：`MODEL_INTERVAL_SEC`（周期秒）、`MODEL_FINISH_HOLD_SEC`（最近完成展示秒）。
- 后端通知：`BACKEND_NOTIFY_URL`、`BACKEND_MODEL_URL`。

## 8. 启动与守护
- 一键脚本统一启动 openGauss / Flask 后端 / UDP 中转 / 采集器 / 模型管理，并进行端口开孔与健康探针。
- 光敏方向：如配置了 `LIGHT_GPIO_GROUP/LIGHT_GPIO_PIN`，启动前自动执行 `gpio_operate set_direction <group> <pin> 0`。
- 日志与 PID：统一位于 `opengauss_logs/`（含 `*.log` 与 `*.pid`）。
- 摄像头设备探测：默认使用 `CAMERA_DEVICE=/dev/video1`；若不存在则自动回退到 `/dev/video0`。
- 中转启动：携带 `IMAGE_TTL_SEC` 与 `IDLE_IMAGE_SEC` 环境变量，启用心跳抓拍与 TTL 删除。

## 9. 编译与构建（编译器使用步骤）
- 使用范围：仅 C 采集器需要编译（温度/光敏/图像）；Python 后端/中转/模型无需编译。
- 触发方式：启动脚本会自动执行 `make -C sensor_collectors` 完成编译；也可手动执行。
- 编译器选择：项目默认采用毕昇（BiSheng）工具链（`/opt/bisheng/bin/gcc`），可通过 `CC`/`CFLAGS` 指定优化选项；若毕昇不可用则回退到系统 GCC。
- 示例：
  - 毕昇：`make -C sensor_collectors CC=/opt/bisheng/bin/gcc CFLAGS='-O3 -march=armv8-a -mtune=tsv110 -ffast-math'`
  - GCC：`make -C sensor_collectors CC=gcc CFLAGS='-O2 -Wall'`
- Makefile 入口：`sensor_collectors/Makefile`（目标：`sensor_temp_collector`、`sensor_light_collector`、`sensor_image_collector`）。
- 验证与日志：编译输出写入 `sensor_collectors_build.log`；采集器运行日志位于 `sensor_*.log`。
- 注意事项：按板卡与系统库版本调整 `CFLAGS`；必要时启用 LTO/PGO（需确保工具链与链接器支持）。

## 10. 权限与安全
- 数据库：强制 md5 认证以提升兼容性与安全性；初始化幂等、失败回退不阻塞。
- GPIO：支持一次性 sysfs 导出（需 root）与 `gpio_operate`（可配置免密 sudo）；推荐仅对必要绝对路径授予 `NOPASSWD`。
- 接口：仅开放必要 REST/SSE；数据库清理限定特定表，避免误删。

## 11. 性能与鲲鹏优化
- 低时延：UDP 轻载 + SSE 推送，避免轮询；就地图像缩放与 TTL 清理降低 I/O 压力。
- 编译器（鲲鹏亲和）：采用毕昇（BiSheng）工具链，启用 ARMv8/NEON 优化；示例：`-O3 -march=armv8-a -mtune=tsv110 -ffast-math`，并按模块启用 LTO/PGO。
- DevKit：用于编译与系统调优（绑核/NUMA/I/O），对中转与采集器的并发与时延进行定位与优化。
- BoostKit：利用向量化与算法库对图像缩放、像素转换、统计聚合进行加速；按需引入，保持代码与依赖可替换。
- 数据库：连接池与重试回退；重启后快速恢复。

### openGauss 使用要点
- 启动与初始化：由 `start_lab_monitor.sh` 幂等配置与启动 openGauss，创建数据库与用户，强制启用 md5 认证；默认端口 `7654`。
- 连接优先级：后端优先使用 `py-opengauss`，无法加载时自动回退到 `psycopg2`；通过 `DB_HOST/PORT/NAME/USER/PASSWORD` 环境变量控制。
- 校验与排障：使用 `gsql` 测试连接；查看 `gs_ctl.log` 与后端 `flask_app.log` 以定位连接或认证问题。

### DevKit 使用建议
- 编译阶段：在 `sensor_collectors/Makefile` 的构建中选择毕昇编译器或 GCC/Clang；通过 `CC`/`CFLAGS` 指定优化选项（示例见第 9 章）。
- 运行阶段：对中转与采集器进行绑核/NUMA 调优，例如：`taskset -c 0-3 bash start_lab_monitor.sh` 或 `numactl --cpunodebind=0 --membind=0 bash start_lab_monitor.sh`。
- 诊断阶段：结合 DevKit 提供的分析工具进行火焰图/热点定位，关注 UDP 收发、图像缩放与数据库写入的耗时分布。

### BoostKit 使用建议
- 加速范围：图像缩放、像素转换、向量统计等热点；结合 BoostKit 的向量化/算法库提升吞吐与稳定性。
- 集成方式：在 C 采集器或中转的加速段引入 BoostKit 头文件与库，示例（按实际路径调整）：`CFLAGS+='-I<boostkit_include>' LDFLAGS+='-L<boostkit_lib> -lboostkit_xxx'`。
- 渐进集成：优先对稳定计算路径做替换，保留纯 C/NumPy/OpenCV 的回退实现，确保不同平台的可移植性。

## 12. 扩展规范
- 新采集器：以 JSON 通过 UDP 上报，建议字段约定 `temperature_c`、`light`、`frame{width,height,pixels}` 等，保持中转解析兼容。
- 新模型：脚本周期输出 JSON 行（至少含 `name` 与 `output`），`print(..., flush=True)`；异常捕获与稳态循环。
- 前端：可在社区页与数据库页新增板块与表单，无需复杂构建链。

## 13. 运维与故障定位
- 健康探针：脚本末尾打印 Web 与 DB 信息；`curl http://<IP>:5000/api/latest` 检查缓存与心跳。
- 关键日志：`flask_app.log`、`udp_relay.log`、`sensor_*.log`、`model_manager.log`、`script_monitor.log`。
- 常见问题：GPIO 未导出/未提权、免密 sudo 未生效、摄像设备路径不一致、数据库服务或认证异常。
- 图像清理：TTL 删除仅作用于文件系统；如需同步清理数据库记录，建议执行：
  - `DELETE FROM image_data WHERE timestamp < NOW() - INTERVAL '1 hour';`
  - `UPDATE sensor_data SET image_path = NULL WHERE timestamp < NOW() - INTERVAL '1 hour';`
- 前端卡顿：隐藏标签页时关闭 SSE，恢复时重连；错误回退到 5s 轮询，防止长时间堆积导致卡顿。

## 14. 术语与兼容性
- openGauss：PostgreSQL 兼容数据库，支持 py-opengauss/psycopg2 连接；默认端口 7654。
- openEuler：鲲鹏生态操作系统；建议开启防火墙端口与 sudoers 精确规则。
- SSE：服务端事件推送，适合低频增量刷新；与 UDP 中转形成低延时闭环。
- 毕昇编译器（BiSheng）：面向鲲鹏的亲和编译器，提供 ARMv8/NEON 优化与链路增强。
- DevKit：鲲鹏平台的开发与调优工具集，用于编译、诊断与系统性能调优。
- BoostKit：鲲鹏平台的加速库集合，提供向量化与常用算法加速能力。

## 15. 相关文件与目录
- 后端与接口：`app.py`
- 数据库初始化：`db_init.sql`
- 启动与守护脚本：`start_lab_monitor.sh`
- UDP 中转：`relay/udp_relay.py`
- 采集器（C）：
  - `sensor_collectors/sensor_temp_collector.c`
  - `sensor_collectors/sensor_light_collector.c`
  - `sensor_collectors/sensor_image_collector.c`
  - `sensor_collectors/Makefile`
- 模型管理与脚本：
  - `models/model_manager.py`
  - `models/model_XX.py`（示例：`model_01.py`）
  - `models/config.json`
- 前端页面与资源：
  - `templates/index.html`
  - `templates/open_source.html`
  - `templates/db.html`
  - `static/js/main.js`
  - `static/css/style.css`
  - `static/images/`
- 辅助脚本：
  - `scripts/script_monitor.py`
  - `scripts/light_example.py`
- 文档：`README.md`、`TECHNICAL_DOCUMENT_CN.md`
- 日志与数据目录（默认）：
  - 日志：`/home/openEuler/opengauss_logs/`
    - `flask_app.log`、`udp_relay.log`、`sensor_temp.log`、`sensor_light.log`、`sensor_image.log`、`model_manager.log`、`script_monitor.log`、`sensor_collectors_build.log`、`gs_ctl.log`
  - 数据：`/home/openEuler/opengauss_data/`

### 当前硬件配置（AHT10 + GY-30）
- AHT10 温湿度传感器：I2C-7（`/dev/i2c-7`）I2C 地址 0x38，采集温度/湿度
- GY-30 光照传感器（BH1750）：I2C 地址 0x23，单位 Lux
- 两者通过同一 I2C 总线外接至面包板，连接到引脚：1（3.3V）、3（I2C7_SDA）、5（I2C7_SCL）、6（GND）

---
*本技术文档用于工程内部与外部交付说明，可与 README 结合使用。*
