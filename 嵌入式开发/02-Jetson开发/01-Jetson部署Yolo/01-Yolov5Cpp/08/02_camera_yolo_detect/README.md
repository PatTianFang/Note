# 摄像头 YOLOv5 TensorRT 检测程序

## 运行方法

先编译：

```bash
cd 08/02_camera_yolo_detect
mkdir -p build
cd build
cmake ..
make -j$(nproc)
```

准备模型和类别文件：

```text
08/02_camera_yolo_detect/models/best_fp16_640.engine
08/02_camera_yolo_detect/models/labels.txt
```

运行 USB 摄像头检测：

```bash
./yolo_camera_detect \
  --source usb:0 \
  --engine ../models/best_fp16_640.engine \
  --labels ../models/labels.txt \
  --width 1280 \
  --height 720 \
  --fps 30
```

运行 Jetson CSI 摄像头检测：

```bash
./yolo_camera_detect \
  --source csi \
  --engine ../models/best_fp16_640.engine \
  --labels ../models/labels.txt \
  --width 1280 \
  --height 720 \
  --fps 30
```

运行本地视频检测：

```bash
./yolo_camera_detect \
  --source video:/home/nvidia/demo.mp4 \
  --engine ../models/best_fp16_640.engine \
  --labels ../models/labels.txt
```

运行 RTSP 网络摄像头检测：

```bash
./yolo_camera_detect \
  --source rtsp://user:password@192.168.1.10:554/stream \
  --engine ../models/best_fp16_640.engine \
  --labels ../models/labels.txt
```

如果暂时没有 `labels.txt`，可以手动指定类别数量：

```bash
./yolo_camera_detect \
  --source usb:0 \
  --engine ../models/best_fp16_640.engine \
  --classes 1
```

程序运行后会显示检测画面，按 `Esc` 或 `q` 退出。

查看所有参数：

```bash
./yolo_camera_detect --help
```

## 项目结构思路

这个项目是在 `07` 图片识别工程跑通之后，把输入源从单张图片扩展为连续视频帧。

结构设计思路：

```text
摄像头或视频流
  -> cv::VideoCapture 读取 frame
  -> YoloDetector.detect(frame)
       -> Preprocess: letterbox、BGR 转 RGB、归一化、NCHW
       -> TensorRTEngine: H2D、TensorRT enqueue、D2H
       -> Postprocess: 置信度过滤、坐标还原、NMS
  -> drawDetections 画框
  -> cv::imshow 显示结果
  -> Esc 或 q 退出
```

本工程不重复发明模型推理核心，而是复用 `07` 中已经实现的图片检测模块。这样 07 负责“单帧怎么检测”，08 只负责“连续从摄像头取帧并展示”。

## 文件作用

```text
02_camera_yolo_detect/
  CMakeLists.txt
  README.md
  models/
    best_fp16_640.engine
    labels.txt
  src/
    main.cpp
```

| 文件 | 作用 |
| --- | --- |
| `CMakeLists.txt` | 配置 OpenCV、CUDA、TensorRT，并链接 `07` 的检测核心源码 |
| `src/main.cpp` | 解析参数，加载 labels，初始化检测器，打开视频源，循环检测并显示 |
| `models/` | 放置 TensorRT engine 和 `labels.txt` |
| `README.md` | 说明运行方法、项目结构、参数、模型要求和排错方法 |

`src/main.cpp` 内部主要函数：

| 函数 | 作用 |
| --- | --- |
| `parseOptions` | 解析输入源、engine、labels、阈值、分辨率等参数 |
| `loadLabels` | 读取 `labels.txt`，一行一个类别名 |
| `buildCsiPipeline` | 生成 Jetson CSI 摄像头 GStreamer pipeline |
| `openCapture` | 打开 USB、CSI、视频文件、RTSP 或自定义 GStreamer |
| `drawDetections` | 把 `Detection` 结果画到画面上 |
| `main` | 初始化模型和视频源，执行读帧、检测、画框、显示主循环 |

复用的 `07` 文件：

| 文件 | 作用 |
| --- | --- |
| `../../07/include/Detection.h` | 检测结果数据结构 |
| `../../07/include/Preprocess.h` / `../../07/src/Preprocess.cpp` | YOLOv5 输入预处理 |
| `../../07/include/TensorRTEngine.h` / `../../07/src/TensorRTEngine.cpp` | TensorRT engine 加载、显存管理和推理 |
| `../../07/include/Postprocess.h` / `../../07/src/Postprocess.cpp` | YOLOv5 输出后处理、坐标还原和 NMS |
| `../../07/include/YoloDetector.h` / `../../07/src/YoloDetector.cpp` | 串联预处理、推理和后处理 |

## 项目作用

这个项目实现摄像头或视频流的实时 YOLOv5 TensorRT 检测。

流程是：

```text
打开输入源
  -> 读取一帧 frame
  -> YOLOv5 预处理
  -> TensorRT 推理
  -> YOLOv5 后处理和 NMS
  -> 在画面上画检测框
  -> 显示 FPS 和检测数量
```

这个工程复用 `07` 图片识别项目中的核心检测代码：

```text
../../07/include/Detection.h
../../07/include/TensorRTEngine.h
../../07/include/YoloDetector.h
../../07/include/Preprocess.h
../../07/include/Postprocess.h

../../07/src/TensorRTEngine.cpp
../../07/src/YoloDetector.cpp
../../07/src/Preprocess.cpp
../../07/src/Postprocess.cpp
```

本目录只新增摄像头读取、显示循环、画框显示和 FPS 统计。

## 支持的输入源

```text
usb:0                 USB 摄像头，打开 /dev/video0
usb:1                 USB 摄像头，打开 /dev/video1
csi                   Jetson CSI 摄像头
video:/path/demo.mp4  本地视频文件
rtsp://...            RTSP 网络摄像头
gst:...               自定义 GStreamer pipeline
```

## 参数说明

```text
--source VALUE      输入源，默认 usb:0
--engine PATH       TensorRT engine 路径，默认 models/best_fp16_640.engine
--labels PATH       labels.txt 路径，默认 models/labels.txt
--classes N         不使用 labels.txt 时手动指定类别数量
--width N           摄像头采集宽度，默认 1280
--height N          摄像头采集高度，默认 720
--fps N             摄像头采集帧率，默认 30
--input-size N      模型输入尺寸，默认 640
--input-w N         模型输入宽度
--input-h N         模型输入高度
--conf N            置信度阈值，默认 0.25
--nms N             NMS 阈值，默认 0.45
--window NAME       显示窗口名称
--help              查看帮助
```

## 模型文件要求

当前版本假设：

1. TensorRT engine 是 YOLOv5 模型导出的 engine。
2. batch = 1。
3. 一个输入 binding。
4. 一个输出 binding。
5. 输入和输出 binding 都是 FP32。
6. 输出格式是 `1 x N x (5 + 类别数)`。

`labels.txt` 必须和训练时类别顺序一致，例如：

```text
person
car
helmet
```

如果 `labels.txt` 行数和模型类别数不一致，后处理会解析错误，表现为类别名错误、检测数量异常或输出尺寸无法整除。

## 推荐使用顺序

先运行纯摄像头预览工程：

```bash
cd 08/01_camera_preview/build
./camera_preview --source usb:0
```

确认画面能稳定显示后，再运行本检测程序。不要在摄像头还没跑通时直接排查 TensorRT 模型。

## 常见问题

如果程序打不开摄像头，先回到 `01_camera_preview` 排查输入源。

如果程序提示找不到 engine，确认 `--engine` 路径是否正确。

如果程序提示没有 labels，确认 `models/labels.txt` 是否存在，或者使用 `--classes N`。

如果检测框整体偏移、变形或大小不对，优先检查 `07` 中的 letterbox 预处理和坐标映射。

如果 FPS 很低，先尝试：

1. 降低摄像头采集分辨率。
2. 降低显示窗口开销。
3. 确认 engine 是否为 FP16。
4. 使用 `tegrastats` 观察 GPU、CPU 和内存占用。

## 验收标准

这个项目跑通的标准是：

1. 能成功打开摄像头或视频流。
2. 能成功加载 TensorRT engine。
3. 画面中能显示检测框。
4. 窗口左上角能显示 FPS 和检测数量。
5. 按 `Esc` 或 `q` 能正常退出。
