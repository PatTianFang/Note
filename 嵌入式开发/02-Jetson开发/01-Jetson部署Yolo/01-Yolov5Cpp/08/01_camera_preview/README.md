# 摄像头画面预览程序

## 运行方法

先编译：

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

运行 Jetson CSI 摄像头：

```bash
./camera_preview --source csi --width 1280 --height 720 --fps 30
```

运行本地视频：

```bash
./camera_preview --source video:/home/nvidia/demo.mp4
```

运行 RTSP 网络摄像头：

```bash
./camera_preview --source rtsp://user:password@192.168.1.10:554/stream
```

使用自定义 GStreamer pipeline：

```bash
./camera_preview --source "gst:nvarguscamerasrc ! video/x-raw(memory:NVMM),width=1280,height=720,framerate=30/1 ! nvvidconv ! video/x-raw,format=BGRx ! videoconvert ! video/x-raw,format=BGR ! appsink drop=1 sync=false"
```

程序运行后会弹出显示窗口，按 `Esc` 或 `q` 退出。

查看所有参数：

```bash
./camera_preview --help
```

## 项目结构思路

这个项目故意保持很小，只验证一件事：OpenCV 能不能稳定从输入源读到画面。

结构设计思路：

```text
摄像头或视频流
  -> cv::VideoCapture 打开输入源
  -> cap.read(frame) 读取画面
  -> cv::imshow 显示画面
  -> Esc 或 q 退出
```

这里不引入 TensorRT、不引入 YOLOv5、不做图像预处理和后处理。这样如果画面打不开，问题范围就只在摄像头、视频流、GStreamer 或 OpenCV VideoCapture。

## 文件作用

```text
01_camera_preview/
  CMakeLists.txt
  README.md
  src/
    main.cpp
```

| 文件 | 作用 |
| --- | --- |
| `CMakeLists.txt` | 配置 OpenCV 依赖，生成 `camera_preview` 可执行程序 |
| `src/main.cpp` | 解析命令行参数，打开输入源，读取 frame，显示画面 |
| `README.md` | 说明运行方法、项目结构、参数和排错方法 |

`src/main.cpp` 内部主要函数：

| 函数 | 作用 |
| --- | --- |
| `parseOptions` | 解析 `--source`、`--width`、`--height`、`--fps` 等参数 |
| `buildCsiPipeline` | 根据宽、高、帧率生成 Jetson CSI 摄像头 GStreamer pipeline |
| `openCapture` | 根据输入源类型打开 USB、CSI、视频文件、RTSP 或自定义 GStreamer |
| `main` | 主循环，读取画面并调用 `cv::imshow` 显示 |

## 项目作用

这个项目只负责打开输入源并显示画面，不加载 YOLOv5 模型，也不做 TensorRT 推理。

它的目的很明确：先确认摄像头或视频流本身是否可用。只有这个程序能稳定显示画面后，再去运行模型检测程序。这样可以避免把摄像头、GStreamer、OpenCV、TensorRT 和 YOLO 后处理问题混在一起排查。

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
--source VALUE   输入源，默认 usb:0
--width N        采集宽度，默认 1280
--height N       采集高度，默认 720
--fps N          采集帧率，默认 30
--window NAME    显示窗口名称
--help           查看帮助
```

## USB 摄像头排查

先确认设备存在：

```bash
ls /dev/video*
```

查看摄像头支持的格式：

```bash
v4l2-ctl --list-devices
v4l2-ctl -d /dev/video0 --list-formats-ext
```

如果设置了 `1280x720 30fps` 但实际打不开，通常是摄像头不支持这个格式。按 `v4l2-ctl` 输出改成摄像头真实支持的分辨率和帧率。

## CSI 摄像头排查

先用 GStreamer 命令确认 CSI 摄像头能工作：

```bash
gst-launch-1.0 nvarguscamerasrc ! nvoverlaysink
```

如果这一步失败，先处理 CSI 驱动、排线、权限或 Jetson 摄像头配置问题，不要直接排查 C++ 代码。

## RTSP 排查

RTSP 打不开时先检查：

1. Jetson 是否能 ping 通摄像头 IP。
2. 用户名、密码和 RTSP 地址是否正确。
3. 摄像头输出是 H.264 还是 H.265。
4. 摄像头是否需要主码流或子码流路径。
5. 网络是否丢包或延迟过高。

也可以先用 GStreamer 测试：

```bash
gst-launch-1.0 rtspsrc location=rtsp://user:password@192.168.1.10/stream latency=100 ! fakesink
```

## 验收标准

这个项目跑通的标准是：

1. 程序能成功打开输入源。
2. 画面能连续显示。
3. 按 `Esc` 或 `q` 能正常退出。
4. 连续运行一段时间没有卡死、断流或内存上涨。
