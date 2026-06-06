## 一、文件说明

- `yolo_realtime_cuda.py`：使用 YOLOv5 + PyTorch CUDA + OpenCV 的实时检测脚本。
- `yolov5-6.0/`：下载并解压的 YOLOv5 v6.0 源代码。
- `yolov5s.pt`：YOLOv5s v6.0 官方预训练权重。

## 二、使用方法

### 1. 摄像头实时检测

```bash
cd /home/gwb/Fang/Yolo
python3 yolo_realtime_cuda.py --source 0 --device cuda:0
```

窗口打开后：

- 按 `q` 退出。
- 按 `Esc` 退出。

### 2. 视频文件检测

```bash
cd /home/gwb/Fang/Yolo
python3 yolo_realtime_cuda.py --source /path/to/video.mp4 --device cuda:0
```

### 3. 图片或无窗口验证

```bash
cd /home/gwb/Fang/Yolo
python3 yolo_realtime_cuda.py --source yolov5-6.0/data/images/bus.jpg --device cuda:0 --no-window --max-frames 1
```

### 4. 常用参数

```bash
python3 yolo_realtime_cuda.py \
  --source 0 \
  --weights yolov5s.pt \
  --img-size 640 \
  --conf-thres 0.25 \
  --iou-thres 0.45 \
  --device cuda:0
```

- `--source`：输入源，摄像头编号、视频路径、图片路径或网络流。
- `--weights`：YOLOv5 `.pt` 权重路径。
- `--img-size`：推理尺寸，默认 `640`。
- `--conf-thres`：置信度阈值，默认 `0.25`。
- `--iou-thres`：NMS 阈值，默认 `0.45`。
- `--device`：默认 `cuda:0`，脚本会明确使用 CUDA。
- `--half`：在 CUDA 上启用 FP16。
- `--no-window`：不显示窗口，只在终端输出状态，适合远程或无显示环境验证。
- `--max-frames`：最多处理多少帧，`0` 表示不限制。