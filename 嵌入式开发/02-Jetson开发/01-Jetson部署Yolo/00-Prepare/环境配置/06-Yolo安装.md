# YOLO 安装


## 1. 下载 YOLOv5 源代码

输入命令：

```bash
git clone --branch v6.0 --depth 1 https://github.com/ultralytics/yolov5.git
```

终端返回出现错误，表明`git clone` 超时。说明GitHub网络连接慢或不稳定。

```text
command timed out after 120015 milliseconds
正克隆到 'yolov5'...
```

git无法顺利拉取，尝试使用wget，输入命令：

```bash
wget -O yolov5-v6.0.zip https://github.com/ultralytics/yolov5/archive/refs/tags/v6.0.zip
```

最终终端返回已保存，表明源码压缩包下载完成：

```text
2026-06-02 07:47:12 (801 KB/s) - “yolov5-v6.0.zip” 已保存 [863360]
```

输入命令进行解压：

```bash
unzip yolov5-v6.0.zip
```

## 2. 下载 YOLOv5s 权重

同样使用wget拉取：

```bash
wget -O yolov5s.pt https://github.com/ultralytics/yolov5/releases/download/v6.0/yolov5s.pt
```

权重最终完整保存。

## 3. 检查文件与基础库

源码目录为 `yolov5-6.0`（刚刚解压的路径）。

输入命令：

```bash
python3 -c "import cv2; print(cv2.__version__); print('has imshow', hasattr(cv2, 'imshow'))"
```

终端返回：

```text
4.6.0
has imshow True
```
表明`OpenCV`可用，并且有 `imshow` 接口，可以显示检测窗口。

输入命令：

```bash
python3 -c "import numpy as np; print(np.__version__)"
```

终端返回：

```text
1.19.4
```
表明 NumPy 可用。
