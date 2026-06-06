# 03 硬件系统准备：Jetson AGX Xavier + Ubuntu 18

这一篇的目标是确认板子处于可开发状态。很多问题不是代码问题，而是系统版本、电源、散热、摄像头驱动没有准备好。

## 你需要准备的东西

基础硬件：

1. Jetson AGX Xavier 开发板或基于 Xavier 的工业主机。
2. 原装或足功率电源。
3. 散热器和风扇。
4. 键盘、鼠标、显示器，或可用的 SSH 网络连接。
5. 足够大的存储空间，建议 64GB 以上。
6. USB 摄像头、CSI 摄像头或网络摄像头。

开发辅助：

1. 一台普通电脑，用来训练模型、整理数据、远程登录 Jetson。
2. 网线或稳定 Wi-Fi。
3. U 盘或 SCP 工具，用来传输 ONNX、engine、测试图片。

## 先确认系统版本

在 Jetson 终端运行：

```bash
cat /etc/nv_tegra_release
uname -a
dpkg -l | grep nvidia-jetpack
```

你要记录这些信息：

```text
L4T 版本：
JetPack 版本：
Ubuntu 版本：
CUDA 版本：
TensorRT 版本：
OpenCV 版本：
```

查询 CUDA：

```bash
nvcc --version
```

如果没有 `nvcc`，说明 CUDA 开发工具可能没装全，但也可能只是环境变量没配好。先不要乱装 CUDA，应该先检查 JetPack 包。

查询 TensorRT：

```bash
dpkg -l | grep -E "nvinfer|tensorrt"
```

查询 OpenCV：

```bash
pkg-config --modversion opencv4 || pkg-config --modversion opencv
```

## 推荐版本基线

如果你明确是 Ubuntu 18.04，那么建议基线是 JetPack 4.6.x。

常见组合：

| 项目 | 推荐基线 |
| --- | --- |
| Jetson 硬件 | Jetson AGX Xavier |
| 系统 | Ubuntu 18.04，来自 JetPack |
| JetPack | 4.6.x，优先 4.6.4 |
| CUDA | 10.2 系列 |
| TensorRT | 8.2.x 系列 |
| cuDNN | 8.2.x 系列 |
| OpenCV | JetPack 自带版本，通常为 4.x |

注意：不同 JetPack 小版本会有差异，最终以板子上 `dpkg` 查询结果为准。

## 不要随便做的事情

### 不要随便升级 Ubuntu 大版本

Jetson 的驱动、CUDA、TensorRT、内核、摄像头栈是绑定的。普通 PC 上的 Ubuntu 升级经验不能直接套用到 Jetson。

不要随意执行：

```bash
sudo do-release-upgrade
```

### 不要随便安装 PC 版 CUDA

Jetson 是 ARM 架构，不是 x86 PC。网上很多 CUDA 安装教程是给台式机显卡用的，不适合 Jetson。

### 不要混用多个 TensorRT

如果你从源码、pip、deb、tar 包混装 TensorRT，很容易导致头文件和库文件版本不一致。C++ 编译能过，但运行时崩溃。

## 电源和散热

YOLOv5 + TensorRT 推理时，GPU 会持续工作。如果电源不足或散热不好，会出现：

1. FPS 忽高忽低。
2. 板子降频。
3. 程序运行一段时间后变慢。
4. 摄像头不稳定。
5. 系统重启。

建议：

1. 使用可靠电源。
2. 确认风扇工作。
3. 长时间测试时监控温度和频率。

监控命令：

```bash
tegrastats
```

这个命令可以看到 CPU、GPU、内存、温度、功耗等状态。

![Alt text](<2026-06-04 10-16-26屏幕截图.png>)

## 性能模式

Jetson 有电源模式管理。不同模式会限制 CPU、GPU 频率和功耗。开发测试时应先确认当前模式：

```bash
sudo nvpmodel -q
sudo nvpmodel -q --verbose
```

如果你的板子模式列表里有 `MAXN` 或最大性能模式，可以切到对应模式。不同载板和系统镜像的编号可能不同，所以先查询再设置，不要盲目照抄：

```bash
sudo nvpmodel -m <模式编号>
sudo jetson_clocks
```

如果需要风扇辅助，可以查看你系统里的 `jetson_clocks` 是否支持风扇参数：

```bash
jetson_clocks --help
```

## 摄像头预检查

### USB 摄像头

插入 USB 摄像头后运行：

```bash
ls /dev/video*
v4l2-ctl --list-devices
```

如果没有 `v4l2-ctl`：

```bash
sudo apt install -y v4l-utils
```

### CSI 摄像头

CSI 摄像头依赖 Jetson 的 Argus/GStreamer 栈。先测试：

```bash
gst-launch-1.0 nvarguscamerasrc ! nvoverlaysink
```

如果这个命令都不行，C++ 程序也很可能打不开 CSI 摄像头。应先解决摄像头驱动和排线问题。

### 网络摄像头

确认 RTSP 地址可访问：

```bash
gst-launch-1.0 rtspsrc location=rtsp://用户名:密码@地址/路径 latency=100 ! fakesink
```

## 文件传输建议

从电脑传文件到 Jetson，推荐用 SCP：

```bash
scp best.onnx nvidia@<jetson_ip>:/home/nvidia/models/
scp labels.txt nvidia@<jetson_ip>:/home/nvidia/models/
```

从 Jetson 传日志回电脑：

```bash
scp nvidia@<jetson_ip>:/home/nvidia/logs/run.log .
```

## 本篇验收标准

完成本篇后，你应该能回答：

1. 我的 JetPack 版本是什么。
2. 我的 CUDA 和 TensorRT 版本是什么。
3. `tegrastats` 能否正常运行。
4. 摄像头能否被系统识别。
5. 当前电源模式是什么。

如果这些问题答不上来，不建议进入 C++ 编程阶段。

## 官方资料

- NVIDIA JetPack 4.6.4：<https://developer.nvidia.com/jetpack-sdk-464>
- NVIDIA JetPack 4.6 安装文档：<https://docs.nvidia.com/jetson/jetpack/4.6/install-jetpack/index.html>
- Jetson Xavier 电源管理：<https://docs.nvidia.com/jetson/archives/l4t-archived/l4t-3275/Tegra%20Linux%20Driver%20Package%20Development%20Guide/power_management_jetson_xavier.html>

