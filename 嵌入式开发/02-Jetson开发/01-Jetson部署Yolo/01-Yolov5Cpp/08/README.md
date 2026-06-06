# 08 摄像头视频流示例说明

本目录包含两个 C++ 示例工程，按排错顺序分开实现。

## 01_camera_preview

作用：只打开摄像头、视频流或视频文件，并显示画面。

这个工程不加载模型，也不做 TensorRT 推理。它用于先确认输入源是否正常，避免把摄像头、GStreamer、模型推理和后处理问题混在一起排查。

支持的输入源格式：

```text
usb:0                 USB 摄像头，等价于 /dev/video0
csi                   Jetson CSI 摄像头
video:/path/demo.mp4  本地视频文件
rtsp://...            RTSP 网络摄像头
gst:...               自定义 GStreamer pipeline
```

编译：

```bash
cd 08/01_camera_preview
mkdir -p build
cd build
cmake ..
make -j$(nproc)
```

运行 USB 摄像头：

```bash
./camera_preview --source usb:0 --width 1280 --height 720 --fps 30
```

运行 CSI 摄像头：

```bash
./camera_preview --source csi --width 1280 --height 720 --fps 30
```

按 `Esc` 或 `q` 退出。

## 02_camera_yolo_detect

作用：打开摄像头或视频流，逐帧调用 YOLOv5 TensorRT 模型检测，并显示带检测框的画面。

这个工程复用 `07` 图片识别工程中的核心模块：

```text
TensorRTEngine
YoloDetector
Preprocess
Postprocess
Detection
```

本工程只新增视频流读取、循环检测、画框显示和 FPS 显示逻辑。

编译：

```bash
cd 08/02_camera_yolo_detect
mkdir -p build
cd build
cmake ..
make -j$(nproc)
```

运行 USB 摄像头检测：

```bash
./yolo_camera_detect \
  --source usb:0 \
  --engine ../models/best_fp16_640.engine \
  --labels ../models/labels.txt
```

如果暂时没有 `labels.txt`，可以指定类别数量：

```bash
./yolo_camera_detect \
  --source usb:0 \
  --engine ../models/best_fp16_640.engine \
  --classes 1
```

按 `Esc` 或 `q` 退出。

## 推荐排错顺序

1. 先运行 `01_camera_preview`，确认摄像头画面可以稳定显示。
2. 再运行 `02_camera_yolo_detect`，接入 TensorRT engine。
3. 如果检测框偏移，优先检查 07 中的 letterbox 预处理和坐标还原。
4. 如果视频卡顿，先降低输入分辨率或关闭显示窗口，再考虑多线程。
