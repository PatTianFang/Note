# 13 YOLOv5 预处理、后处理、坐标还原详解

这一篇专门讲 YOLOv5 部署里最容易错的部分：图片怎么喂给模型，模型输出怎么变成原图上的框。

## 为什么这部分重要

很多时候 TensorRT engine 没问题，C++ 也能运行，但结果不对：

1. 没有框。
2. 框整体偏上或偏下。
3. 框比例不对。
4. 类别乱。
5. 同一个物体很多框。

这些大多来自预处理和后处理。

## OpenCV 图片格式

OpenCV 读取图片后：

```text
shape = H x W x 3
channel = BGR
type = uint8
range = 0 到 255
memory layout = HWC
```

YOLOv5 常见输入：

```text
shape = 1 x 3 x 640 x 640
channel = RGB
type = float32
range = 0.0 到 1.0
memory layout = NCHW
```

所以必须转换。

## Letterbox 的目的

假设原图是：

```text
1280 x 720
```

模型输入是：

```text
640 x 640
```

如果直接 resize：

```text
1280 x 720 -> 640 x 640
```

画面会被拉伸。更好的做法是保持比例：

```text
1280 x 720 -> 640 x 360
上下补边到 640 x 640
```

这就是 letterbox。

## Letterbox 计算

设：

```text
原图宽 = original_w
原图高 = original_h
模型宽 = input_w
模型高 = input_h
```

缩放比例：

```text
scale = min(input_w / original_w, input_h / original_h)
```

缩放后的宽高：

```text
resized_w = round(original_w * scale)
resized_h = round(original_h * scale)
```

补边：

```text
pad_x = (input_w - resized_w) / 2
pad_y = (input_h - resized_h) / 2
```

例子：

```text
original_w = 1280
original_h = 720
input_w = 640
input_h = 640

scale = min(640/1280, 640/720) = min(0.5, 0.8889) = 0.5
resized_w = 640
resized_h = 360
pad_x = 0
pad_y = 70
```

注意：这里上下各补 70，不是 140。总共补 140。

## BGR 转 RGB

OpenCV：

```cpp
cv::cvtColor(letterbox_image, rgb_image, cv::COLOR_BGR2RGB);
```

如果忘记这一步，识别效果通常会明显变差。

## 归一化

把 `uint8` 转成 `float` 并除以 255：

```text
0 -> 0.0
255 -> 1.0
128 -> 0.5019
```

C++ 中常见做法：

```cpp
rgb_image.convertTo(float_image, CV_32FC3, 1.0 / 255.0);
```

## HWC 转 NCHW

OpenCV Mat 内存是：

```text
像素1: R,G,B
像素2: R,G,B
像素3: R,G,B
```

模型要的是：

```text
所有 R
所有 G
所有 B
```

伪代码：

```cpp
std::vector<float> input(3 * input_h * input_w);

for (int y = 0; y < input_h; ++y) {
    for (int x = 0; x < input_w; ++x) {
        cv::Vec3f pixel = float_image.at<cv::Vec3f>(y, x);
        input[0 * input_h * input_w + y * input_w + x] = pixel[0];
        input[1 * input_h * input_w + y * input_w + x] = pixel[1];
        input[2 * input_h * input_w + y * input_w + x] = pixel[2];
    }
}
```

这里 `pixel[0]` 是 R，因为已经做过 BGR 转 RGB。

## YOLOv5 输出解释

假设输出 shape 是：

```text
1 x 25200 x 8
```

类别数是 3，那么每个候选框 8 个数：

```text
cx, cy, w, h, objectness, class0, class1, class2
```

计算：

```text
best_class = 最大 class 分数的编号
class_score = 最大 class 分数
confidence = objectness * class_score
```

如果：

```text
objectness = 0.80
class_score = 0.90
```

最终：

```text
confidence = 0.72
```

如果阈值是 0.25，这个框保留。

## 从中心点格式转角点格式

YOLOv5 常见框格式是中心点：

```text
cx, cy, w, h
```

画框通常需要：

```text
x1, y1, x2, y2
```

转换：

```text
x1 = cx - w / 2
y1 = cy - h / 2
x2 = cx + w / 2
y2 = cy + h / 2
```

这些坐标还在模型输入图上，通常是 640x640 坐标。

## 坐标还原到原图

先减补边，再除缩放比例：

```text
orig_x1 = (x1 - pad_x) / scale
orig_y1 = (y1 - pad_y) / scale
orig_x2 = (x2 - pad_x) / scale
orig_y2 = (y2 - pad_y) / scale
```

然后限制范围：

```text
orig_x1 = max(0, min(orig_x1, original_w - 1))
orig_y1 = max(0, min(orig_y1, original_h - 1))
orig_x2 = max(0, min(orig_x2, original_w - 1))
orig_y2 = max(0, min(orig_y2, original_h - 1))
```

如果不 clamp，画框可能越界。

## NMS 处理

候选框过滤后，仍可能有多个重叠框。使用 NMS：

```cpp
cv::dnn::NMSBoxes(
    boxes,
    scores,
    conf_threshold,
    nms_threshold,
    indices
);
```

建议按类别分别做 NMS。否则不同类别但位置重叠的目标可能互相删除。

第一版如果类别很少，也可以先全局 NMS，跑通后再优化。

## 调试方法

### 方法 1：打印前 5 个高分框

打印：

```text
class_id
objectness
class_score
confidence
model_box
original_box
```

如果 model_box 正常、original_box 偏，问题在坐标还原。

如果 confidence 全很低，问题可能在预处理或模型。

### 方法 2：用同一张图片对比 Python 结果

用 Python YOLOv5 检测同一张图，再用 C++ 检测同一张图。比较：

1. 框数量。
2. 置信度。
3. 类别。
4. 框位置。

允许有小差异，但不能完全不同。

### 方法 3：先降低阈值

如果没有框：

```text
conf_threshold = 0.10
```

如果低阈值有框，高阈值没框，说明模型信心不足或阈值过高。

如果低阈值也完全没框，优先检查预处理和输出解析。

## 最常见错误清单

1. 忘记 BGR 转 RGB。
2. 忘记除以 255。
3. HWC/NCHW 写错。
4. `pad_x`、`pad_y` 算错。
5. 原图宽高顺序写反。
6. 类别数量写错。
7. 输出 shape 写死但和实际模型不一致。
8. 没有做 NMS。
9. `labels.txt` 顺序和训练不一致。

## 本篇验收标准

你应该能做到：

1. 手算一张 1280x720 图片到 640x640 的 letterbox 参数。
2. 解释 BGR、RGB、HWC、NCHW 的区别。
3. 把 `cx, cy, w, h` 转成 `x1, y1, x2, y2`。
4. 把 640 坐标还原成原图坐标。
5. 知道没有检测框时先检查哪些地方。

