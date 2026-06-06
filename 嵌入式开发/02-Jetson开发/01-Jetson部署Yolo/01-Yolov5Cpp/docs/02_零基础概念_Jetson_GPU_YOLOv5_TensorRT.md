# 02 零基础概念：Jetson、GPU、YOLOv5、TensorRT

这一篇只讲概念。先把名词搞清楚，后面看命令和代码才不会乱。

## Jetson AGX Xavier 是什么

Jetson AGX Xavier 是 NVIDIA 的嵌入式 AI 计算平台。你可以把它理解成一台小型 Linux 电脑，但它不是普通电脑主板，它里面集成了 ARM CPU、NVIDIA GPU、内存、视频编解码器和各种外设接口。

它适合做：

1. 摄像头实时识别。
2. 边缘 AI 推理。
3. 工业视觉。
4. 机器人视觉。
5. 无人设备感知。

它不适合新手一开始就做：

1. 大规模模型训练。
2. 复杂深度学习环境折腾。
3. 同时接很多高分辨率摄像头并追求极限 FPS。

## Ubuntu 18.04 和 JetPack 的关系

Jetson 的系统不是普通 PC 版 Ubuntu 直接装上去就完事。NVIDIA 会提供 JetPack，里面包含：

1. Jetson Linux，也叫 L4T。
2. Ubuntu 用户空间。
3. CUDA。
4. cuDNN。
5. TensorRT。
6. OpenCV 等常用库。
7. 摄像头、显示、硬件编解码相关驱动。

所以在 Jetson 上，最重要的不是单独问“我是什么 Ubuntu”，而是问：

```bash
cat /etc/nv_tegra_release
dpkg -l | grep nvidia-jetpack
```

你要优先相信 JetPack 版本，而不是随便跟着网上教程升级 CUDA 或 TensorRT。

## CPU 和 GPU 的区别

CPU 像一个通用工人，什么都能做，但大量重复计算不一定最快。

GPU 像一大批并行工人，特别适合做矩阵乘法、卷积、图像处理这类重复计算。YOLOv5 的神经网络推理主要是大量卷积运算，所以应该交给 GPU。

在本项目里：

1. CPU 负责读摄像头、组织流程、执行后处理、显示结果。
2. GPU 负责神经网络推理。
3. CUDA 是让程序使用 NVIDIA GPU 的基础技术。
4. TensorRT 是 NVIDIA 专门做深度学习推理优化的库。

## YOLOv5 是什么

YOLO 是目标检测算法。目标检测不是只判断图片里有没有某个东西，而是要输出：

1. 物体类别，比如人、车、安全帽、缺陷。
2. 物体位置，也就是矩形框。
3. 置信度，也就是模型有多确定。

YOLOv5 是 Ultralytics 维护的一套 YOLO 实现，常见模型大小包括：

| 模型 | 特点 | 新手建议 |
| --- | --- | --- |
| YOLOv5n | 最小最快，精度较低 | 极限低功耗时用 |
| YOLOv5s | 速度和精度均衡 | 第一版推荐 |
| YOLOv5m | 精度更好，速度更慢 | 第二阶段尝试 |
| YOLOv5l/x | 更大更慢 | 不建议一开始在 Xavier 上用 |

第一阶段建议用 YOLOv5s，输入尺寸先用 640。

## `.pt`、`.onnx`、`.engine` 分别是什么

### `.pt`

`.pt` 是 PyTorch 权重文件。训练 YOLOv5 后通常会得到：

```text
best.pt
last.pt
```

`best.pt` 表示验证效果最好的模型，通常用于部署。

### `.onnx`

ONNX 是一种通用模型格式。它像一个中间文件，可以把 PyTorch 模型交给 TensorRT、OpenVINO、ONNX Runtime 等工具。

在本路线中：

```text
best.pt -> best.onnx
```

ONNX 的作用是过渡。

### `.engine`

`.engine` 是 TensorRT 为目标机器优化后的推理引擎。它不是通用文件，和下面这些东西强相关：

1. Jetson 的 GPU 架构。
2. TensorRT 版本。
3. CUDA 版本。
4. 输入尺寸。
5. FP32、FP16、INT8 精度设置。

所以不要在 PC 上生成 `.engine` 后拷到 Jetson 用。正确做法是：

```text
在 PC 或云端训练 best.pt
在 PC 或云端导出 best.onnx
把 best.onnx 拷到 Jetson
在 Jetson 本机生成 best.engine
```

## TensorRT 做了什么

TensorRT 不是一个新模型，它是推理优化器和运行时。它会做这些事情：

1. 解析 ONNX 网络结构。
2. 合并一些可以合并的计算层。
3. 选择更快的 GPU kernel。
4. 使用 FP16 或 INT8 降低计算量。
5. 为目标 GPU 生成高效 engine。

对你来说，TensorRT 的意义就是：同样的 YOLOv5 模型，在 Jetson 上比普通 PyTorch 推理更快，更适合 C++ 部署。

## FP32、FP16、INT8 是什么

| 精度 | 含义 | 优点 | 缺点 | 新手建议 |
| --- | --- | --- | --- | --- |
| FP32 | 32 位浮点 | 精度稳 | 速度较慢 | 只用于对照 |
| FP16 | 16 位浮点 | 速度快，通常精度损失小 | 少数模型需验证 | 第一版推荐 |
| INT8 | 8 位整数 | 更快，显存更少 | 需要校准，精度可能掉 | 第二阶段再做 |

Jetson AGX Xavier 支持 FP16，第一阶段建议直接用 FP16。

## 一个识别程序的完整流水线

真实运行时，程序每一帧大致做这些事：

```text
读取图像
  -> 缩放和补边到 640x640
  -> BGR 转 RGB
  -> 像素归一化到 0.0 到 1.0
  -> HWC 转 NCHW
  -> 拷贝到 GPU
  -> TensorRT 推理
  -> 拿到输出框
  -> 置信度过滤
  -> NMS 去重
  -> 坐标映射回原图
  -> 画框或输出 JSON
```

这里最容易出错的是预处理和后处理，不是 TensorRT 本身。比如：

1. 图片颜色顺序错了，BGR/RGB 混了。
2. 忘记除以 255。
3. resize 没有保持比例，导致框偏移。
4. 后处理没有乘 `objectness * class_confidence`。
5. NMS 阈值设置不合适。

## 什么叫实时

实时不是一个固定数字，要看业务场景：

| 场景 | 可能够用的 FPS |
| --- | --- |
| 拍照检测 | 1 FPS 也可能够 |
| 工业传送带慢速检测 | 5 到 15 FPS |
| 普通监控 | 15 到 25 FPS |
| 机器人避障 | 20 到 30 FPS 以上 |

第一阶段不要一上来追求 60 FPS。先跑通，再优化。

