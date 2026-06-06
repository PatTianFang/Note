# YoloJetsonCpp

这是一个最小版 TensorRT + OpenCV YOLOv5 图片识别工程。

第一版只做三件事：

1. 加载 TensorRT engine。
2. 读取一张图片并执行推理。
3. 把检测框画到图片上并保存结果。

## 工程结构

```text
07/
  CMakeLists.txt
  include/
    Detection.h
    TensorRTEngine.h
    YoloDetector.h
    Preprocess.h
    Postprocess.h
  src/
    main.cpp
    TensorRTEngine.cpp
    YoloDetector.cpp
    Preprocess.cpp
    Postprocess.cpp
  models/
    best_fp16_640.engine
    labels.txt
  test_images/
    demo.jpg
  output/
```

## 编译

JetPack 4.x 常见 CMake 版本是 3.10，使用下面这种兼容写法：

```bash
cd 07
mkdir -p build
cd build
cmake ..
make -j$(nproc)
```

编译成功后会生成：

```bash
07/build/yolo_detect
```

## 准备文件

运行前需要放好三个文件：

```text
07/models/best_fp16_640.engine
07/models/labels.txt
07/test_images/demo.jpg
```

`labels.txt` 必须和训练时类别顺序一致，一行一个类别名。

## 运行

在 `07` 目录下执行：

```bash
./build/yolo_detect \
  --engine models/best_fp16_640.engine \
  --labels models/labels.txt \
  --image test_images/demo.jpg \
  --output output/result.jpg
```

如果暂时没有 `labels.txt`，可以直接指定类别数量：

```bash
./build/yolo_detect \
  --engine models/best_fp16_640.engine \
  --classes 1 \
  --image test_images/demo.jpg \
  --output output/result.jpg
```

## 参数说明

```text
--engine PATH       TensorRT engine 路径
--labels PATH       labels.txt 路径
--image PATH        输入图片路径
--output PATH       输出图片路径
--input-size N      模型输入尺寸，默认 640
--input-w N         模型输入宽度
--input-h N         模型输入高度
--classes N         不使用 labels.txt 时手动指定类别数量
--conf N            置信度阈值，默认 0.25
--nms N             NMS 阈值，默认 0.45
--help              查看帮助
```

## 当前版本限制

当前版本按第一版目标实现，假设：

1. batch = 1。
2. 一个输入 binding。
3. 一个输出 binding。
4. 输入、输出 binding 都是 FP32。
5. YOLOv5 输出格式是 `1 x N x (5 + 类别数)`。

图片版本跑通后，再扩展视频、摄像头、多路输入和更复杂的动态 shape。
