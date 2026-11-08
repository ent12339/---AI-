# 智流 - AI 视觉水循环柔性分选系统

<p align="center">
  <b>AI 视觉识别 × 多层柔性分选 × 水循环运输 × Web 触控控制台</b><br/>
  <sub>基于 K1 MUSE Pi Pro（Bianbu）的边缘端智能分选系统</sub>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Platform-K1%20MUSE%20Pi%20Pro%20%2F%20Bianbu-2f81f7?style=flat-square" alt="Platform" />
  <img src="https://img.shields.io/badge/AI-YOLO%20ONNX%20%7C%20SpaceMIT-8b5cf6?style=flat-square" alt="AI" />
  <img src="https://img.shields.io/badge/Backend-FastAPI-009688?style=flat-square" alt="Backend" />
  <img src="https://img.shields.io/badge/UI-Web%20Console-orange?style=flat-square" alt="UI" />
  <a href="https://github.com/ent12339/qianrushi"><img src="https://img.shields.io/badge/GitHub-ent12339%2Fqianrushi-181717?logo=github&style=flat-square" alt="Repository" /></a>
  <a href="https://github.com/ent12339/qianrushi/releases/download/v1.0/K1.MUSE.Pi.Pro.mp4"><img src="https://img.shields.io/badge/Demo-Video-ff69b4?style=flat-square" alt="Demo" /></a>
</p>

**智流 - AI 视觉水循环柔性分选系统**是一套面向多品类物料分选的边缘端 AI 视觉柔性分选系统：以水循环水流作为物料输送载体，以 YOLO 视觉识别作为分选判据，以多层 PWM 舵机作为执行机构，并通过 Web 控制台完成参数配置、设备绑定、实时监控与日志追溯。

> 项目原名「智能加工材料分拣机」，现更名为「智流 - AI 视觉水循环柔性分选系统」，代码结构与技术栈保持不变。

## 目录

- [项目简介](#项目简介)
- [系统组成与工作原理](#系统组成与工作原理)
- [主要功能](#主要功能)
- [项目演示视频](#项目演示视频)
- [项目结构](#项目结构)
- [核心模块说明](#核心模块说明)
- [环境依赖](#环境依赖)
- [模型文件放置](#模型文件放置)
- [安装方法](#安装方法)
- [运行方法](#运行方法)
- [访问控制台](#访问控制台)
- [Web 控制台使用流程](#web-控制台使用流程)
- [分选逻辑](#分选逻辑)
- [舵机参数说明](#舵机参数说明)
- [API 接口说明](#api-接口说明)
- [项目亮点](#项目亮点)
- [常见问题](#常见问题)
- [当前项目状态](#当前项目状态)

## 项目简介

本项目是一个基于 K1 MUSE Pi Pro / Bianbu 系统开发的多层视觉智能分选系统。系统通过 USB 摄像头采集画面，使用 YOLO ONNX 模型识别红、黄、蓝、绿等不同颜色物料，并通过 Linux PWM 控制 MG90 舵机完成分选动作；水泵由主控软件控制启停与水流速度，驱动水循环系统连续输送物料，物料穿过各分选层的检测区域完成分流。

与传统「一次性分选」方案相比，本系统强调**柔性**：

- 分选规则不写死在代码里，每层的目标颜色、摄像头、PWM 通道、触发线、置信度和舵机参数都可在 Web 控制台中独立配置；
- 非目标物料可选择 `pass`（直通下一层）或 `other`（打入 OTHER 位），从而适配多层串联筛选与双向分流等不同工艺；
- 摄像头和 PWM 通道支持自动扫描，首次绑定保存后按 UID 自动恢复，现场更换设备无需改代码；
- 水泵接入软件控制，可由主控统一控制水泵启停与水流速度，与视觉分选流程联动运行。

项目采用模块化结构，将摄像头扫描、模型推理、目标跟踪、分选层控制、PWM 舵机控制、日志记录和 Web 控制台拆分为多个模块，方便比赛展示、现场调试和后续扩展。

## 系统组成与工作原理

### 硬件组成

| 部件 | 说明 |
|---|---|
| K1 MUSE Pi Pro 开发板 | 系统主控，运行 Bianbu 系统，负责视觉推理、舵机控制与水泵控制 |
| USB 摄像头 | 每层一路（以实际接线为准），支持自动扫描与试读 |
| MG90 舵机 | 分选执行机构，通过 Linux PWM 控制 |
| 水循环运输系统 | 回流水泵、输水管路与底部集水区；水泵已接入 K1 主控，可由软件控制启停并调节水流速度 |
| MIPI 触摸屏（可选） | 现场展示用，Chromium Kiosk 全屏打开 Web 控制台 |

### 工作流程

```text
物料投放
  │
  ▼
水循环运输系统（水泵由主控软件控制启停与流速）
  │
  ▼
第一层 → 第二层 → 第三层 → 第四层        # 多层柔性分选
  │
  ▼
USB 摄像头采集 → YOLO 推理 → 目标跟踪 → 触发线判断 → PWM 舵机分流
  │
  ▼
Web 控制台实时显示画面 / 状态 / 计数 / 日志
```

系统的控制逻辑可以概括为「**水循环负责运输、视觉负责识别、主控统一控制水泵与舵机、Web 控制台负责配置与监控**」：

1. 系统开启水泵并设置合适的水流速度，建立稳定的循环水流；
2. 水泵驱动水流，带动物料在分选通道中连续前进；
3. 各层 USB 摄像头持续采集画面，模型推理识别物料颜色与位置；
4. 当同一物料被连续跟踪并跨过该层触发线时，按该层规则决定舵机动作；
5. 未分选物料随水流汇入底部集水区，再由水泵回流至顶部入口，形成闭环循环；
6. 分选结果、计数与运行日志实时推送到 Web 控制台并写入 SQLite。

> ⚠️ **说明**：水泵已接入软件控制，启动分选前请先开启水泵并确认水流稳定；软件停止不能替代物理断电急停。

## 主要功能

- 支持多层分选结构，默认包含四个分选层。
- 支持红、黄、蓝、绿四种目标颜色识别，每层可独立选择目标颜色。
- 支持每层独立绑定摄像头、PWM 通道、触发线、控制置信度和舵机参数。
- 支持摄像头自动扫描和 PWM 通道自动扫描，绑定信息按 UID 保存和恢复。
- 支持 YOLO ONNX Runtime 推理。
- 支持 SpaceMITExecutionProvider 硬件加速。
- 支持多路摄像头共享一个模型会话与单推理调度线程，避免争抢 NPU。
- 支持 IoU 多目标跟踪、连续命中帧数判断与触发线跨越检测，减少误触发与重复触发。
- 支持 `pass`（非目标直通）与 `other`（非目标分流）两种非目标动作，适配不同分选工艺。
- 支持 Web 控制台实时查看状态、日志和视频画面。
- 支持单层启动、暂停、继续、停止和清零。
- 支持全部启动、全部暂停、全部停止和全部清零。
- 支持手动目标位、手动 OTHER 位和舵机复位。
- 支持运行中手动控制舵机。
- 支持 SQLite 保存系统运行日志。
- 支持系统温度、CPU、内存等运行指标查看。
- 支持水泵软件控制：可由主控统一控制水循环水泵启停，并调节水流速度，适配不同轻质物料的输送需求。
- 支持水循环闭环运行：未分选物料随水流回流至顶部入口，可长时间连续循环分选。

## 项目演示视频

本项目演示视频已上传至 GitHub Release：

[点击查看项目演示视频](https://github.com/ent12339/qianrushi/releases/download/v1.0/K1.MUSE.Pi.Pro.mp4)

> 如果链接无法打开，请进入本仓库右侧 `Releases` 页面，下载对应的演示视频附件。

## 项目结构

```text
智流- AI 视觉水循环柔性分选系统/
├── app.py                      # FastAPI 后端入口
├── config.yaml                 # 系统配置文件
├── requirements.txt            # Python Web 端依赖
├── run.sh                      # 一键启动脚本
├── README.md                   # 项目说明文档
├── sorter/                     # 后端核心模块
│   ├── config.py               # 配置默认值、读取、校验和保存
│   ├── devices.py              # 摄像头和 PWM 设备扫描
│   ├── layer.py                # 单层分选控制逻辑（含 PWM 舵机控制）
│   ├── logstore.py             # SQLite 日志存储
│   ├── system.py               # 多层系统总管理器
│   └── vision.py               # 摄像头采集、YOLO 推理、目标跟踪与绘制
├── web/                        # Web 控制台页面
│   └── console.html            # 主控制台页面
├── models/                     # 模型目录，按实际情况放置
├── data/                       # 运行时数据目录，日志数据库会自动生成
└── deploy/                     # 部署文件，可选
```

> 注意：当前 `web/` 目录中主要页面是 `console.html`，建议运行后优先访问 `/console`。如果要访问根路径 `/`，需要保证 `web/index.html` 存在，或者在 `app.py` 中把根路径改为返回 `console.html`。

## 核心模块说明

### app.py

`app.py` 是后端服务入口，基于 FastAPI 实现，主要负责：

- 初始化 `SorterSystem`
- 挂载 Web 静态文件
- 提供 Web 页面访问
- 提供系统状态接口
- 提供设备扫描接口
- 提供分选层配置接口
- 提供单层启动、暂停、继续、停止接口
- 提供全部启动、暂停、停止接口
- 提供日志读取接口
- 提供每层视频流接口
- 提供 WebSocket 状态推送
- 提供运行中手动 PWM 舵机控制补丁
- 通过 `--config` 参数指定配置文件，通过 `SORTER_CONFIG` 环境变量传递给应用

主要访问地址：

```text
http://开发板IP:8000/console
```

### config.yaml

`config.yaml` 是系统运行配置文件，保存模型路径、摄像头参数、推理参数、每层目标颜色、摄像头绑定、PWM 绑定、触发线和舵机占空比参数。

系统主要参数示例：

```yaml
system:
  name: 智流 - AI 视觉水循环柔性分选系统
  camera_width: 640
  camera_height: 480
  camera_fps: 30
  camera_cache_size: 2
  conf_threshold: 0.15
  nms_iou: 0.45
  topk: 100
  warmup_runs: 10
  camera_scan_max_index: 64
  jpeg_quality: 80
```

其中 `system.name` 是 Web 控制台显示的系统名称，可按部署需要修改。

四层默认配置参考如下（首次部署请在控制台重新扫描设备并保存绑定，实际值以 `config.yaml` 为准）：

| 分选层 | 目标颜色 | 摄像头（参考） | PWM 芯片（参考） | PWM 通道 | 触发线 Y | 非目标动作 |
|---|---|---|---|---|---|---|
| 第一层 | red | `/dev/video20` | `/sys/class/pwm/pwmchip0` | 0 | 400 | other |
| 第二层 | yellow | `/dev/video22` | `/sys/class/pwm/pwmchip2` | 0 | 400 | other |
| 第三层 | blue | `/dev/video24` | `/sys/class/pwm/pwmchip1` | 0 | 300 | other |
| 第四层 | green | `/dev/video26` | `/sys/class/pwm/pwmchip3` | 0 | 300 | other |

每层除了颜色和绑定信息外，还包含以下可调参数：

| 参数 | 说明 |
|---|---|
| `trigger_line` | 触发线在画面中的 Y 坐标 |
| `flow_direction` | 物料流动方向，`1` 为自上向下，`-1` 为自下向上 |
| `control_conf` | 触发舵机所需的最低置信度 |
| `min_hits` | 目标需要连续命中的帧数，用于过滤抖动 |
| `track_iou` | IoU 跟踪匹配阈值 |
| `track_max_age` | 跟踪丢失后保留轨迹的最大帧数 |
| `startup_delay` | 启动后延迟多久开始触发，避免误动作 |
| `infer_interval_ms` | 该层推理最小间隔，用于控制多路负载 |
| `non_target_action` | 非目标动作：`pass` 或 `other` |

> 注意：同一个摄像头是否能同时给多层使用，需要根据实际硬件和运行情况确认。正式比赛展示前建议在 Web 控制台中重新扫描设备并保存每层绑定。

### sorter/config.py

负责系统默认配置、配置合并、路径解析、参数校验和配置保存。

该模块限制的颜色类别为：

```text
red
yellow
blue
green
```

支持的非目标动作包括：

```text
pass
other
```

其中：

- `pass` 表示非目标物料直通，舵机不动作，适合多层串联结构。
- `other` 表示非目标物料转向 OTHER 位置，适合双向分流结构。

### sorter/devices.py

负责扫描硬件设备，包括：

- `/dev/video*` 摄像头节点
- `/sys/class/pwm/pwmchip*` PWM 通道

摄像头扫描时会尝试打开设备并读取画面，用于判断该设备是否可用。PWM 扫描时会读取 `pwmchip` 和通道数量，并生成稳定 UID，方便保存和恢复设备绑定。界面中会标记不可读的摄像头节点。

### sorter/vision.py

负责视觉识别相关功能，包括：

- 摄像头采集线程
- 只保留最新帧，降低延迟
- 强制 USB 摄像头使用 V4L2
- 设置 MJPG、640x480、30fps、小缓冲
- 图像预处理
- ONNX Runtime 推理
- SpaceMITExecutionProvider 加速
- NMS 后处理
- IoU 多目标跟踪（`IoUTracker`）
- 检测框、类别和跟踪 ID 绘制
- `InferenceScheduler` 统一推理调度，多层共享一个模型会话

### sorter/layer.py

负责单个分选层的运行逻辑。每层包含：

- 摄像头实例
- PWM 舵机实例
- 推理任务提交与结果处理
- 目标颜色判断
- IoU 跟踪与触发线跨越检测
- 计数统计
- 状态管理
- 画面叠加显示
- 手动动作控制

每层的触发逻辑为：物料轨迹从触发线一侧跨到另一侧、置信度不低于 `control_conf`、且连续命中帧数不少于 `min_hits` 时，才执行一次动作；同一条轨迹只会触发一次，避免每一帧重复驱动舵机。识别到目标颜色时执行 `target` 动作；识别到非目标颜色且 `non_target_action` 为 `other` 时执行 `other` 动作，为 `pass` 时舵机不动作。

### sorter/system.py

负责整个多层分选系统的管理，包括：

- 加载配置
- 初始化日志数据库（`data/sorter.db`）
- 扫描设备
- 加载 YOLO 模型
- 创建推理调度器
- 创建多层控制器
- 管理所有分选层状态
- 汇总目标计数、非目标计数和运行层数
- 提供系统温度、CPU、内存等运行指标

### sorter/logstore.py

负责将系统运行日志写入 SQLite 数据库。默认数据库路径为：

```text
data/sorter.db
```

系统运行后会自动创建该文件。

### web/console.html

`console.html` 是 Web 控制台页面，主要功能包括：

- 查看系统在线状态
- 查看故障层数
- 选择第一层、第二层、第三层、第四层
- 选择摄像头
- 选择 PWM 通道
- 设置目标颜色
- 设置非目标动作
- 设置触发线 Y
- 设置控制置信度
- 设置目标 duty、复位 duty、OTHER duty
- 设置舵机保持时间
- 保存参数
- 启动、暂停、继续、停止、清零
- 手动目标位
- 手动 OTHER 位
- 舵机复位
- 查看右侧实时视频
- 查看目标计数、非目标计数、FPS、推理耗时
- 查看系统运行日志
- 查看水循环运行提示（启动分选前确认水泵已开启、水流稳定）

## 环境依赖

`requirements.txt` 中包含以下 Web 端依赖：

```text
fastapi
uvicorn[standard]
PyYAML
psutil
```

在 K1 / Bianbu 系统中，以下硬件相关库通常已经适配完成，不建议通过普通 `pip` 随意覆盖安装：

```text
numpy
cv2
onnxruntime
spacemit_ort
```

## 模型文件放置

当前 `config.yaml` 中模型路径为：

```text
/home/bianbu/1/ai-sdk/models/my_yolov11m/model/best_yolov11n_int8_fix.q.onnx
```

标签文件路径为：

```text
/home/bianbu/1/ai-sdk/models/my_yolov11m/data/label.txt
```

建议模型目录结构如下：

```text
models/
└── my_yolov11m/
    ├── model/
    │   └── best_yolov11n_int8_fix.q.onnx
    └── data/
        └── label.txt
```

`label.txt` 建议内容如下：

```text
red
yellow
blue
green
```

标签顺序必须与模型训练时的类别顺序保持一致。

## 安装方法

进入项目目录（目录名以实际为准）：

```bash
cd "智流- AI 视觉水循环柔性分选系统"
```

> 如果代码已直接放在仓库根目录，直接在仓库根目录执行后续命令即可。

安装 Web 端依赖：

```bash
python3 -m pip install -r requirements.txt
```

如果在开发板上已经配置好 OpenCV、ONNX Runtime 和 `spacemit_ort`，不要重新覆盖这些硬件相关库。

## 运行方法

推荐使用启动脚本：

```bash
bash run.sh
```

也可以直接运行：

```bash
python3 app.py --host 0.0.0.0 --port 8000
```

如果 PWM 没有普通用户写权限，可以使用：

```bash
sudo -E python3 app.py --host 0.0.0.0 --port 8000
```

如果需要指定配置文件：

```bash
python3 app.py --config config.yaml --host 0.0.0.0 --port 8000
```

### 触摸屏 Kiosk 展示（可选）

在 MIPI 触摸屏上全屏展示控制台：

```bash
chromium --kiosk --noerrdialogs --disable-infobars http://127.0.0.1:8000/console
```

不同系统上 Chromium 命令名称可能不同（`chromium` 或 `chromium-browser`），可按实际情况调整，也可写入 `deploy/` 目录做开机自启。

## 访问控制台

开发板本机访问：

```text
http://127.0.0.1:8000/console
```

同一局域网电脑或手机访问：

```text
http://开发板IP:8000/console
```

例如：

```text
http://192.168.1.100:8000/console
```

## Web 控制台使用流程

1. 启动水泵并设置合适的水流速度（水泵已接入软件控制），确认水循环水流稳定。
2. 启动程序。
3. 打开浏览器访问 `/console`。
4. 点击“扫描设备”。
5. 分别进入第一层、第二层、第三层、第四层。
6. 给每层选择摄像头和 PWM 通道。
7. 给每层选择目标颜色。
8. 设置非目标动作：
   - `pass`：非目标直通。
   - `other`：非目标打到 OTHER 位。
9. 设置触发线 Y、控制置信度和舵机 duty 参数。
10. 点击“保存参数”。
11. 点击“手动目标位 / 手动 OTHER 位 / 舵机复位”检查机械方向。
12. 点击“启动”或“全部启动”开始分选。
13. 在右侧查看实时画面、FPS、推理耗时和计数结果。

## 分选逻辑

每层都有自己的目标颜色、触发线、跟踪参数和舵机动作参数。

当摄像头识别到物料后：

1. 模型输出检测框与颜色类别，经 NMS 后进入 IoU 跟踪，得到稳定的轨迹 ID。
2. 当轨迹从触发线一侧跨越到另一侧（方向由 `flow_direction` 决定），且置信度不低于 `control_conf`、连续命中帧数不少于 `min_hits` 时，才认为该物料“过线”。
3. 如果识别颜色等于当前层目标颜色，则舵机执行 `target` 动作。
4. 如果识别颜色不是当前层目标颜色，并且 `non_target_action` 为 `other`，则舵机执行 `other` 动作。
5. 如果识别颜色不是当前层目标颜色，并且 `non_target_action` 为 `pass`，则舵机不动作，物料继续流向下一层。
6. 系统更新目标计数、非目标计数、运行状态和日志；同一条轨迹只触发一次动作。
7. 页面实时刷新显示当前层状态和画面。

## 舵机参数说明

常用 PWM 参数如下：

| 参数 | 说明 |
|---|---|
| `period_ns` | PWM 周期 |
| `min_duty_ns` | 舵机最小安全占空比 |
| `max_duty_ns` | 舵机最大安全占空比 |
| `init_duty_ns` | 初始化占空比 |
| `reset_duty_ns` | 复位位置占空比 |
| `target_duty_ns` | 目标颜色动作占空比 |
| `other_duty_ns` | 非目标 OTHER 动作占空比 |
| `hold_seconds` | 舵机动作保持时间 |
| `smooth_steps` | 平滑移动步数 |
| `smooth_delay` | 平滑移动每步延迟 |
| `queue_size` | 舵机动作队列长度 |

当前主要舵机参数：

```yaml
period_ns: 5000000
reset_duty_ns: 1500000
target_duty_ns: 1630000
other_duty_ns: 1370000
hold_seconds: 2
```

> ⚠️ 舵机建议使用独立电源并与开发板共地；首次调试请先小角度验证方向，再逐步加大动作幅度。

## API 接口说明

| 接口 | 方法 | 功能 |
|---|---|---|
| `/` | GET | 旧首页，需要 `web/index.html` |
| `/console` | GET | Web 控制台 |
| `/api/status` | GET | 获取系统状态 |
| `/api/devices` | GET | 获取摄像头和 PWM 设备 |
| `/api/devices/rescan` | POST | 重新扫描设备 |
| `/api/layers` | GET | 获取所有分选层状态 |
| `/api/layers/{layer_id}/config` | PUT | 更新指定分选层配置 |
| `/api/layers/{layer_id}/target-color` | POST | 修改指定分选层目标颜色 |
| `/api/layers/{layer_id}/start` | POST | 启动指定分选层 |
| `/api/layers/{layer_id}/pause` | POST | 暂停指定分选层 |
| `/api/layers/{layer_id}/resume` | POST | 继续指定分选层 |
| `/api/layers/{layer_id}/stop` | POST | 停止指定分选层 |
| `/api/layers/{layer_id}/reset-counts` | POST | 清零指定分选层计数 |
| `/api/layers/{layer_id}/manual/{action}` | POST | 手动控制舵机动作 |
| `/api/system/start` | POST | 启动全部分选层 |
| `/api/system/pause` | POST | 暂停全部分选层 |
| `/api/system/stop` | POST | 停止全部分选层 |
| `/api/system/reset-counts` | POST | 清零全部计数 |
| `/api/logs` | GET | 获取系统日志 |
| `/api/layers/{layer_id}/stream` | GET | 获取指定分选层实时视频流 |
| `/ws/status` | WebSocket | 实时推送系统状态 |

`manual/{action}` 中的 `action` 可使用：

```text
target
other
reset
```

## 项目亮点

- 以水循环水流为输送载体，实现物料的连续输送与循环流转，输送过程更柔性。
- 使用 K1 MUSE Pi Pro 完成边缘端视觉识别与控制，无需上位机。
- 使用 YOLO ONNX 模型识别不同颜色物料。
- 使用 SpaceMITExecutionProvider 提升推理效率。
- 使用多层结构提高分选能力，每层规则可独立配置。
- 使用 IoU 跟踪 + 触发线跨越判断 + 最小命中帧数，减少误触发和重复动作。
- 使用 Linux PWM 直接控制舵机，结构简单、成本低。
- 使用 Web 控制台进行触摸屏式交互，支持参数在线保存。
- 使用模块化代码结构，便于展示、维护和扩展。
- 支持实时视频、实时日志和实时统计数据。
- 水泵接入主控软件控制：可调节水泵启停与水流速度，水循环与视觉分选联动，可连续闭环运行。

## 常见问题

### 1. 网页打不开 `/`

当前 `app.py` 中 `/` 默认返回 `web/index.html`，如果项目里没有 `index.html`，访问 `/` 会失败。建议直接访问：

```text
http://开发板IP:8000/console
```

或者把 `app.py` 中的根路径改成返回 `console.html`。

### 2. 水泵 / 水循环相关

水泵已接入主控，可由软件控制启停与水流速度。启动分选前请先确认水泵已开启、水流稳定。如果水泵不转或水流异常，可以检查：

1. 水泵供电是否正常，驱动/继电器接线是否可靠。
2. 运行程序的用户是否具有对应 GPIO / PWM 的控制权限（必要时使用 `sudo -E` 启动）。
3. 水位是否充足，回流水路是否堵塞或存在漏气。
4. 水泵转向与管路连接是否符合回流水路设计。
5. 调速信号（如 PWM / 驱动模块）接线是否正确，调速范围与水泵参数是否匹配。

> 不同版本的水泵控制方式（GPIO 继电器、驱动模块等）可能不同，以实际接线和程序实现为准。

### 3. 摄像头打不开

可以先检查设备：

```bash
ls /dev/video*
```

然后在控制台点击“扫描设备”。如果有多个同型号 USB 摄像头，当前程序会使用 sysfs 路径和 `/dev/videoX` 生成 UID，减少 UID 冲突。

### 4. 舵机不动

检查：

1. PWM 通道是否选对。
2. 是否点击了“保存参数”。
3. 程序是否有写 `/sys/class/pwm` 的权限。
4. 是否需要使用 `sudo -E` 启动。
5. 舵机是否使用独立供电，GND 是否与开发板共地。

### 5. 识别不到物料或频繁误触发

可以依次检查：

1. `control_conf` 是否过高或过低。
2. `min_hits`、`track_iou`、`track_max_age` 是否适合当前水流速度。
3. `trigger_line` 与 `flow_direction` 是否与实际物料流动方向一致。
4. `label.txt` 顺序是否与模型训练类别顺序一致。
5. 摄像头曝光、光照和水面反光是否影响识别。

### 6. SpaceMITExecutionProvider 不可用

需要确认 `spacemit_ort` 和适配版 ONNX Runtime 已正确安装。不要随意用普通 `pip` 覆盖开发板原本适配好的推理环境。

## 当前项目状态

当前项目已经完成：

- FastAPI 后端服务
- Web 控制台
- 多层分选配置
- 摄像头和 PWM 扫描
- YOLO ONNX 推理
- SpaceMIT 硬件加速调用
- IoU 多目标跟踪与触发线跨越判断
- 实时视频流
- SQLite 日志
- 舵机手动控制
- 目标颜色分选逻辑
- 水泵软件控制（启停与流速调节）与水循环闭环运行
- 系统运行指标展示（温度、CPU、内存）

项目仍在持续完善中，欢迎在实战环境中按实际硬件调整触发线、层内参数和舵机参数。
