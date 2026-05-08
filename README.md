# Smart-Trash-Can

一个以「智能垃圾桶」为主题的综合项目仓库，包含了多个阶段的实现：
- Jetson Nano / Python 端的视觉识别 + 语音交互 + 串口控制；
- 桌面端（Mac）语音服务与联调代码；
- STM32 嵌入式控制代码；
- 数据集与标注文件。

> 说明：该仓库包含多个历史版本与实验目录，代码风格和入口不完全一致。建议优先参考 `Smart_Home_nano/` 与 `Intelligent_voice_trash_can_Beta/`。

---

## 1. 仓库结构概览

- `Smart_Home_nano/`：较完整的 Nano 端主流程（UI + 视觉 + 语音 + 串口）。
- `Intelligent_voice_trash_can_Beta/`：另一套 Beta 实现，包含 ONNX 检测模型、UI 和语音助手。
- `Intelligent_voice_trash_can/`：更早版本实现。
- `nanoBeta/`、`nanoNew/`、`Smart_Home_nano_old/`、`Smart_Home_mac_old/`：历史分支/备份版本。
- `Smart_Home_mac/`：Mac 端语音与网络联调脚本。
- `Contest_garbage_collect/`：STM32 固件工程（电机、串口、控制等硬件模块）。
- `datasets/smart_can/`、`img/`：训练图片与标注数据。
- `requirement.txt`：Python 依赖（包含 OpenCV、PyQt5、PyTorch、onnxruntime、pyserial 等）。

---

## 2. 项目目标

核心目标是实现一个可落地的「智能垃圾分类系统」：
1. 摄像头检测到投放动作；
2. 模型识别垃圾类别；
3. 通过串口向下位机发送开盖/转盘/压缩等控制指令；
4. 通过 UI 实时展示状态；
5. 语音模块支持唤醒、播报与交互。

常见分类类别（代码中可见）：
- 厨余垃圾
- 可回收垃圾
- 有害垃圾
- 其他垃圾

---

## 3. 关键流程（以 `Smart_Home_nano` 为例）

`Smart_Home_nano/main.py` 中主程序会：
1. 初始化共享数据结构（满载状态、当前类别、计数、UI 文案等）；
2. 初始化网络客户端、分类模型、视觉处理对象、语音处理对象；
3. 启动 UI 子进程；
4. 主循环中并发运行“视觉触发”和“语音触发”，谁先触发谁进入对应处理流程。

视觉链路（`vision_processing.py`）大致为：
- 采集背景帧；
- 连续读相机帧做差分检测是否有新物体；
- 调用 ONNX 分类模型得到垃圾类别；
- 向嵌入式端发送控制码，更新 UI 与服务端状态。

模型链路（`AI_module.py`）大致为：
- 图像缩放/补黑到模型输入大小；
- 通过 `onnxruntime` 推理；
- softmax + argmax 得到类别并映射到四分类。

---

## 4. 关键流程（以 `Intelligent_voice_trash_can_Beta` 为例）

`Intelligent_voice_trash_can_Beta/main.py` 主要逻辑：
- 启动 UI 进程；
- 加载 ONNX 模型；
- 启动视觉模块 `Vision_Module`；
- 可选启动语音助手（iFlytek / SparkApi 相关接口）。

`Intelligent_voice_trash_can_Beta/cv_module.py`：
- 通过边缘计数判断是否有投放动作；
- 调用 AI 模块识别垃圾；
- 维护投放统计、满载状态与串口通信；
- 通过消息队列向 UI/语音模块同步数据。

`Intelligent_voice_trash_can_Beta/AI_module.py`：
- 使用 ONNX 检测输出框与类别；
- 将细粒度类别映射到四大垃圾分类。

---

## 5. 运行环境建议

- 系统：Ubuntu（Jetson Nano）或 macOS（联调脚本）
- Python：建议 3.8（部分代码为该版本编译缓存）
- 硬件：
  - USB 摄像头（或 CSI 摄像头）
  - 串口设备（默认代码里常见 `/dev/ttyUSB0`）
  - 下位机（STM32 控制板）

安装依赖：

```bash
pip install -r requirement.txt
```

> 注意：`requirement.txt` 中部分包版本较旧，首次部署可能需要按平台调整（如 `PyAudio`、`onnxruntime`、`torch`）。

---

## 6. 快速开始（推荐路径）

### 方案 A：运行 `Smart_Home_nano`

```bash
cd Smart_Home_nano
python main.py
```

运行前请检查：
- `resnet50.onnx` 文件存在；
- 摄像头索引是否正确（代码默认 `0`）；
- 串口地址是否正确（mac 常见 `/dev/tty.Bluetooth-Incoming-Port`，Linux 常见 `/dev/ttyUSB0`）；
- `socket_client.py` 的服务器 IP/端口是否可达。

### 方案 B：运行 `Intelligent_voice_trash_can_Beta`

```bash
cd Intelligent_voice_trash_can_Beta
python main.py
```

运行前请检查：
- `main.py` 中 `load_path` 对应 ONNX 文件存在；
- `cv_module.py` 串口参数与本机一致；
- 语音接口账号/密钥（若启用 iFlytek/Spark）。

---
