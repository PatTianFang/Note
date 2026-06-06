# 05 模型训练、数据集与 YOLOv5 导出

这一篇讲模型从哪里来。Jetson 上的 C++ 程序最终需要 `.engine`，但 `.engine` 的源头通常是训练得到的 `best.pt`。

## 先分清训练和部署

训练阶段：

```text
大量图片 + 标注
  -> Python
  -> PyTorch
  -> YOLOv5
  -> 得到 best.pt
```

部署阶段：

```text
best.pt
  -> 导出 best.onnx
  -> Jetson 上转换 best.engine
  -> C++ 加载 best.engine 推理
```

训练可以在电脑、服务器、云平台上做。部署才放到 Jetson 上。

## 数据集是什么

假设你要识别安全帽，那么数据集就是很多图片，每张图片里标出：

1. 安全帽的位置。
2. 没戴安全帽的人。
3. 其他你关心的类别。

YOLOv5 常用标注格式是：

```text
class_id center_x center_y width height
```

这些数值是归一化比例，不是像素。例如：

```text
0 0.5123 0.4412 0.1200 0.3000
```

含义是：

1. 类别编号是 0。
2. 框中心点在图片宽度的 51.23% 处。
3. 框中心点在图片高度的 44.12% 处。
4. 框宽度占图片宽度的 12%。
5. 框高度占图片高度的 30%。

## 数据集目录结构

建议结构：

```text
my_dataset/
  images/
    train/
      0001.jpg
      0002.jpg
    val/
      1001.jpg
      1002.jpg
  labels/
    train/
      0001.txt
      0002.txt
    val/
      1001.txt
      1002.txt
  data.yaml
```

图片和标签要一一对应：

```text
images/train/0001.jpg
labels/train/0001.txt
```

## `data.yaml` 示例

比如你识别 3 类：

```yaml
path: /home/user/datasets/my_dataset
train: images/train
val: images/val

names:
  0: helmet
  1: no_helmet
  2: person
```

类别顺序非常重要。训练、导出、C++ 显示标签时必须一致。

## 数据集质量比模型大小更重要

新手常犯的错误是急着换大模型，却忽视数据。实际项目里，以下问题更常见：

1. 图片太少。
2. 场景单一。
3. 标注不准确。
4. 训练集和实际摄像头画面差异很大。
5. 白天、夜晚、逆光、遮挡样本不够。
6. 只有正样本，缺少容易误检的负样本。

建议第一版数据：

| 项目 | 建议 |
| --- | --- |
| 每类图片数量 | 至少几百张，越多越好 |
| 验证集比例 | 约 10% 到 20% |
| 图片来源 | 尽量来自真实摄像头 |
| 标注标准 | 框要贴近目标，不要忽大忽小 |
| 负样本 | 要有，减少误检 |

## 在电脑或云端训练 YOLOv5

克隆 YOLOv5：

```bash
git clone https://github.com/ultralytics/yolov5.git
cd yolov5
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

训练 YOLOv5s：

```bash
python train.py \
  --img 640 \
  --batch 16 \
  --epochs 100 \
  --data /home/user/datasets/my_dataset/data.yaml \
  --weights yolov5s.pt \
  --name my_yolov5s
```

训练完成后，权重通常在：

```text
runs/train/my_yolov5s/weights/best.pt
```

## 为什么不建议在 Jetson 上训练

Jetson AGX Xavier 的优势是边缘推理，不是训练。新手在 Jetson 上训练容易遇到：

1. Python 版本冲突。
2. PyTorch 安装麻烦。
3. 显存不够。
4. 训练慢。
5. 系统依赖被改乱。

Ubuntu 18.04 默认 Python 偏旧，而当前许多深度学习项目的依赖版本已经更新。为了少踩坑，建议训练和导出在电脑或云端完成。

## 导出 ONNX

训练完成后，在 YOLOv5 目录中执行：

```bash
python3 export.py \
  --weights runs/train/my_yolov5s/weights/best.pt \
  --img 640 640 \
  --batch 1 \
  --include onnx \
  --opset 12 \
  --simplify
```

说明：

| 参数 | 含义 |
| --- | --- |
| `--weights` | 要导出的 PyTorch 权重 |
| `--img 640 640` | 输入尺寸固定为 640x640 |
| `--batch 1` | 部署时一帧一帧推理 |
| `--include onnx` | 导出 ONNX |
| `--opset 12` | 对 JetPack 4.x 的 TensorRT 更稳妥 |
| `--simplify` | 简化 ONNX 图，有时更容易被 TensorRT 解析 |

如果 `--simplify` 报错，可以先去掉它，导出普通 ONNX。

## 检查 ONNX

导出后应看到：

```text
best.onnx
```

建议在电脑上安装 Netron 查看模型：

```bash
pip install netron
netron best.onnx
```

你要确认：

1. 输入名称，常见为 `images`。
2. 输入形状，通常是 `1x3x640x640`。
3. 输出形状，常见类似 `1x25200x(5+类别数)`。

例如 3 个类别时，输出最后一维通常是：

```text
5 + 3 = 8
```

其中 5 表示：

```text
x, y, w, h, objectness
```

后面是每个类别的置信度。

## 是否导出带 NMS 的模型

新手第一版建议不要导出带 NMS 的模型。也就是让 ONNX 输出候选框，然后在 C++ 里做：

1. 置信度过滤。
2. 坐标转换。
3. NMS 去重。

这样更容易理解，也方便调试框偏移、阈值、类别名称等问题。

如果导出时启用了 NMS，输出格式会变化，C++ 后处理也要跟着变。等第一版跑通后再考虑。

## 把 ONNX 拷到 Jetson

```bash
scp runs/train/my_yolov5s/weights/best.onnx nvidia@<jetson_ip>:/home/nvidia/models/
scp /home/user/datasets/my_dataset/labels.txt nvidia@<jetson_ip>:/home/nvidia/models/
```

`labels.txt` 一行一个类别名：

```text
helmet
no_helmet
person
```

顺序必须和 `data.yaml` 里的 `names` 一致。

## 本篇验收标准

完成本篇后，你应该有：

1. `best.pt`。
2. `best.onnx`。
3. `labels.txt`。
4. 清楚知道模型输入尺寸。
5. 清楚知道类别数量。
6. 清楚知道 ONNX 输出形状。

如果不知道输出形状，先不要写 C++ 后处理。

## 官方资料

- YOLOv5 自定义数据训练：<https://docs.ultralytics.com/yolov5/tutorials/train_custom_data/>
- YOLOv5 模型导出：<https://docs.ultralytics.com/yolov5/tutorials/model_export/>
- ONNX 官方介绍：<https://onnx.ai/onnx/intro/concepts.html>

