# YOLOv5 训练计划与 ONNX 导出方案

本文用于规划一个自定义目标检测项目：从任务定义、数据准备、训练评估，到最终导出 ONNX 并接入部署环境。它不是某个固定数据集的快速命令清单，而是一份可以按阶段执行的项目指南。

适用场景：

- 使用本仓库训练自定义 YOLOv5 检测模型。
- 数据集尚未完全准备好，需要先规划采集、标注和划分。
- 最终目标是得到可部署的 `best.onnx`。

---

## 1. 总体流程

建议按下面顺序推进：

```text
任务定义 -> 数据采集 -> 数据标注 -> 数据划分 -> 环境准备
        -> 训练 -> 评估 -> 导出 ONNX -> 推理验证 -> 部署优化
```

单人小项目通常需要 2-4 周。耗时最多的部分通常不是训练，而是数据采集、标注和错误样本回收。

---

## 2. 任务定义

训练前先把问题定义清楚。否则后续采集的数据、模型尺寸和部署方案都可能返工。

| 问题 | 需要明确的内容 | 示例 |
|------|----------------|------|
| 检测目标 | 模型要识别什么对象 | 行人、安全帽、零件缺陷 |
| 类别列表 | 类别名称和顺序 | `0: person`, `1: helmet` |
| 应用场景 | 室内、室外、产线、监控、移动端 | 工厂产线缺陷检测 |
| 输入来源 | 图片、视频、工业相机、摄像头流 | 固定工业相机 |
| 部署硬件 | CPU、NVIDIA GPU、Jetson、NPU | Windows + NVIDIA GPU |
| 速度要求 | 单张延迟或 FPS | 15 FPS 以上 |
| 精度目标 | mAP、召回率、误检率 | mAP@0.5 不低于 0.85 |
| 输出格式 | 框、类别、置信度、后处理要求 | `[x1,y1,x2,y2,score,class]` |

本阶段产出一份任务定义表。类别名称和顺序一旦进入标注阶段，不要随意改动。

---

## 3. 数据准备

### 3.1 数据规模建议

| 类别数量 | 推荐训练集 | 推荐验证集 | 推荐测试集 |
|----------|------------|------------|------------|
| 1 类 | 500-2000 张 | 100 张以上 | 100 张以上 |
| 2-5 类 | 每类 300-1500 张 | 每类 50-100 张 | 每类 50-100 张 |
| 5 类以上 | 每类 200-1000 张 | 每类 50 张以上 | 每类 50 张以上 |

数据质量优先于数量。500 张清晰、场景覆盖好的图片，通常比 5000 张重复、模糊或标注不一致的图片更有价值。

### 3.2 采集原则

- 覆盖真实部署场景，包括光照、角度、距离、背景和遮挡。
- 使用尽量接近最终部署的相机或数据源。
- 保留难例，例如反光、模糊、小目标、密集目标和部分遮挡。
- 控制类别平衡，避免某一类数量远超其他类。
- 单独保留测试集，不要参与训练和调参。

### 3.3 标注规范

YOLOv5 检测任务要求每张图片对应一个同名 `.txt` 标签文件。每行格式如下：

```text
class_id x_center y_center width height
```

要求：

- 坐标必须归一化到 0-1。
- `class_id` 从 0 开始，顺序必须与数据集 yaml 中的 `names` 一致。
- 框应贴合可见目标，不要包含过多背景。
- 遮挡严重、可见区域过少的目标应按项目规则统一处理。
- 标注标准必须前后一致，尤其是缺陷检测和小目标检测。

常用标注工具：

| 工具 | 适用场景 |
|------|----------|
| LabelImg | 本地轻量标注，支持 YOLO 格式 |
| Roboflow | Web 标注、团队协作、格式转换 |
| X-AnyLabeling | AI 辅助标注，适合批量提效 |
| Labelme | 多边形标注，适合先做分割再转检测 |

---

## 4. 数据集目录结构

YOLOv5 推荐的数据目录如下：

```text
datasets/my_dataset/
  images/
    train/
    val/
    test/
  labels/
    train/
    val/
    test/
  my_dataset.yaml
```

要求：

- `images/train` 中的 `abc.jpg` 应对应 `labels/train/abc.txt`。
- `val` 和 `test` 同理。
- `test` 可选，但正式项目建议保留。

示例数据集配置：

```yaml
path: C:/Users/PatTi/Desktop/yolov5/datasets/my_dataset
train: images/train
val: images/val
test: images/test

nc: 3
names:
  0: cat
  1: dog
  2: bird
```

建议使用绝对路径，减少工作目录变化导致的路径错误。

---

## 5. 环境准备

推荐环境：

| 项目 | 建议 |
|------|------|
| Python | 3.10 或 3.11 |
| PyTorch | 与本机 CUDA 驱动匹配的稳定版本 |
| GPU | 8GB 显存以上更稳，6GB 可用较小模型 |
| 系统 | Windows 11、WSL2 或 Ubuntu |

安装依赖：

```bash
cd C:/Users/PatTi/Desktop/yolov5
pip install -r requirements.txt
pip install onnx onnx-simplifier onnxruntime-gpu
```

验证 PyTorch：

```bash
python -c "import torch; print(torch.__version__, torch.cuda.is_available())"
```

如果 `torch.cuda.is_available()` 为 `False`，先解决 CUDA / PyTorch 安装问题，再开始正式训练。

---

## 6. 模型选择

YOLOv5 常用模型尺寸如下：

| 模型 | 特点 | 适用场景 |
|------|------|----------|
| `yolov5n` | 最小、最快、精度较低 | CPU、边缘设备、快速验证 |
| `yolov5s` | 速度和精度平衡 | 多数项目的起步选择 |
| `yolov5m` | 精度更好、资源占用更高 | 中等显存 GPU |
| `yolov5l` | 高精度 | 服务器 GPU |
| `yolov5x` | 最大、最慢、精度最高 | 精度优先且算力充足 |

推荐策略：

- 不确定时先用 `yolov5s`。
- 显存紧张或只想跑通流程时用 `yolov5n`。
- 小目标、遮挡多、类别多时，再尝试 `yolov5m/l/x`。

多数项目不需要修改模型结构。先使用官方 `yolov5*.yaml` 和对应预训练权重完成基线训练。

---

## 7. 开始训练

基础训练命令：

```bash
python train.py \
  --data datasets/my_dataset/my_dataset.yaml \
  --cfg yolov5s.yaml \
  --weights yolov5s.pt \
  --epochs 100 \
  --batch-size 16 \
  --img-size 640 \
  --device 0 \
  --project runs/train \
  --name my_model
```

关键参数：

| 参数 | 说明 | 建议 |
|------|------|------|
| `--data` | 数据集 yaml | 使用项目自己的配置 |
| `--cfg` | 模型结构 | 与模型尺寸一致 |
| `--weights` | 初始权重 | 优先使用预训练权重 |
| `--epochs` | 训练轮数 | 100-300 |
| `--batch-size` | 批大小 | 在不 OOM 的前提下尽量大 |
| `--img-size` | 输入尺寸 | 常用 640，小目标可尝试 960/1280 |
| `--device` | 训练设备 | GPU 用 `0`，CPU 用 `cpu` |
| `--workers` | 数据加载进程数 | Windows 可从 2 开始 |
| `--hyp` | 超参文件 | 先用默认，必要时再调 |

显存不足时按顺序处理：

1. 降低 `--batch-size`，例如 16 改 8 或 4。
2. 降低 `--img-size`，例如 640 改 512。
3. 换用更小模型，例如 `yolov5s` 改 `yolov5n`。
4. 在 Windows 上减少 `--workers`，必要时改为 `--workers 0`。
5. 避免在内存较小的机器上使用 `--cache ram`。

---

## 8. 训练监控

启动 TensorBoard：

```bash
tensorboard --logdir runs/train
```

浏览器打开：

```text
http://localhost:6006
```

重点观察：

- `train/box_loss`、`train/obj_loss`、`train/cls_loss` 是否下降。
- `val/box_loss` 是否在后期反弹。
- `metrics/mAP_0.5` 和 `metrics/mAP_0.5:0.95` 是否提升。
- Precision 与 Recall 是否符合业务要求。

如果训练集效果好、验证集效果差，优先怀疑过拟合或数据划分不合理。

---

## 9. 评估模型

训练完成后，先在验证集或测试集评估：

```bash
python val.py \
  --weights runs/train/my_model/weights/best.pt \
  --data datasets/my_dataset/my_dataset.yaml \
  --img-size 640 \
  --task test \
  --device 0
```

如果没有单独测试集，将 `--task test` 改成 `--task val`。

评估后建议再做一次可视化检查：

```bash
python detect.py --weights runs/train/my_model/weights/best.pt --source datasets/my_dataset/images/test --save-txt --save-conf
```

重点查看漏检、误检和低置信度样本。这些样本通常比盲目增加训练轮数更能指导下一轮数据补充。

---

## 10. 导出 ONNX

推荐先导出固定 batch、固定输入尺寸的 ONNX，便于部署侧验证。

```bash
python export.py \
  --weights runs/train/my_model/weights/best.pt \
  --include onnx \
  --img-size 640 640 \
  --batch-size 1 \
  --simplify \
  --device 0
```

导出产物：

```text
runs/train/my_model/weights/best.onnx
```

参数说明：

| 参数 | 说明 |
|------|------|
| `--include onnx` | 导出 ONNX |
| `--img-size 640 640` | 输入高宽，与训练和部署保持一致 |
| `--batch-size 1` | 固定 batch，部署最简单 |
| `--simplify` | 简化 ONNX 图，通常建议开启 |
| `--dynamic` | 动态 batch 或动态尺寸，只有部署确实需要时再加 |
| `--half` | FP16 导出，需要目标推理后端支持 |

关于 `--opset`：

- 没有明确部署要求时，先不指定 `--opset`。
- 如果目标框架要求固定 opset，再根据 PyTorch 和部署框架版本选择。
- 较新的 PyTorch 版本可能无法稳定导出到很旧的 opset。遇到 `Resize` 算子回退错误时，优先取消旧 opset 限制，或换用支持目标 opset 的 PyTorch 版本。

---

## 11. 验证 ONNX

最小验证脚本：

```python
import cv2
import numpy as np
import onnx
import onnxruntime as ort

model_path = "runs/train/my_model/weights/best.onnx"

model = onnx.load(model_path)
onnx.checker.check_model(model)
print("ONNX structure valid")

session = ort.InferenceSession(
    model_path,
    providers=["CUDAExecutionProvider", "CPUExecutionProvider"],
)

img = cv2.imread("test.jpg")
img = cv2.resize(img, (640, 640)).astype(np.float32) / 255.0
img = img.transpose(2, 0, 1)[None]

outputs = session.run(None, {session.get_inputs()[0].name: img})
print([o.shape for o in outputs])
```

如果使用本仓库已有的 `verify_onnx.py`，需要确认脚本中的模型路径与实际导出路径一致。

验证目标：

- ONNX 能被 `onnx.load` 加载。
- `onnx.checker.check_model` 不报错。
- ONNX Runtime 能完成一次推理。
- 输出维度与部署侧预期一致。

---

## 12. 部署选择

| 部署目标 | 推荐方案 |
|----------|----------|
| 通用 Windows / Linux | ONNX Runtime |
| NVIDIA GPU 高性能推理 | TensorRT |
| Intel CPU / 边缘盒 | OpenVINO |
| Android | ONNX Runtime Mobile 或 NCNN |
| Web | ONNX Runtime Web |

常用导出命令：

```bash
# ONNX
python export.py --weights runs/train/my_model/weights/best.pt --include onnx --img-size 640 --simplify

# TensorRT
python export.py --weights runs/train/my_model/weights/best.pt --include engine --img-size 640 --half

# OpenVINO
python export.py --weights runs/train/my_model/weights/best.pt --include openvino --img-size 640
```

部署前需要固定以下内容：

- 输入尺寸和图像预处理方式。
- 输出格式和后处理方式。
- 置信度阈值和 NMS 阈值。
- CPU / GPU provider。
- 性能指标，包括平均延迟、P95 延迟和 FPS。

---

## 13. 常见问题

| 问题 | 常见原因 | 处理方式 |
|------|----------|----------|
| `Dataset not found` | yaml 中 `path` 写错 | 使用绝对路径，或从 YOLOv5 根目录运行 |
| `No labels found` | 标签路径或文件名不匹配 | 检查 `images` 与 `labels` 是否同名同结构 |
| mAP 一直为 0 | 类别顺序不一致或标签格式错误 | 检查 `class_id`、`names` 和坐标归一化 |
| `CUDA out of memory` | batch、图片尺寸或模型过大 | 降 batch、降尺寸、换小模型 |
| 训练集好但测试集差 | 过拟合或测试场景差异大 | 补充数据、增强难例、重新划分数据 |
| 小目标漏检 | 输入尺寸不足或样本少 | 提高 `img-size`，补充小目标样本 |
| ONNX 与 PyTorch 结果差异大 | 预处理、尺寸或后处理不一致 | 对齐 resize、归一化、NMS 和阈值 |
| ONNX CPU 推理太慢 | 后端选择不合适 | 尝试 OpenVINO、TensorRT 或量化 |

---

## 14. 项目检查清单

### 准备阶段

- [ ] 任务定义表完成。
- [ ] 类别名称和顺序确认。
- [ ] 部署硬件和推理框架确认。
- [ ] 数据采集计划确认。

### 数据阶段

- [ ] 图片覆盖真实场景。
- [ ] 标签格式为 YOLO 格式。
- [ ] `train`、`val`、`test` 已划分。
- [ ] `my_dataset.yaml` 路径和类别顺序正确。

### 训练阶段

- [ ] `python train.py` 能正常启动。
- [ ] loss 正常下降。
- [ ] TensorBoard 指标已记录。
- [ ] `best.pt` 已生成。

### 评估阶段

- [ ] `val.py` 已在验证集或测试集上运行。
- [ ] 已查看可视化检测结果。
- [ ] 已记录主要漏检和误检类型。

### 导出阶段

- [ ] `best.onnx` 已生成。
- [ ] ONNX 结构检查通过。
- [ ] ONNX Runtime 推理通过。
- [ ] 部署侧预处理和后处理已对齐。

---

## 15. 推荐资料

- Ultralytics YOLOv5 文档：https://docs.ultralytics.com/yolov5/
- ONNX Runtime 文档：https://onnxruntime.ai/docs/
- TensorRT 文档：https://docs.nvidia.com/deeplearning/tensorrt/
- OpenVINO 文档：https://docs.openvino.ai/
- 本仓库示例：`tutorial.ipynb`

