
### 1. 第一次测试加载模型，发现缺少 Pillow

输入命令：

```bash
python3 -c "import sys, torch; sys.path.insert(0, 'yolov5-6.0'); ckpt=torch.load('yolov5s.pt', map_location='cuda:0'); model=ckpt['model'].float().fuse().eval().to('cuda:0'); print(type(model)); print(next(model.parameters()).device); print(model.names[:5] if isinstance(model.names, list) else list(model.names.items())[:5])"
```

终端返回：

```text
Traceback (most recent call last):
  File "<string>", line 1, in <module>
  File "/home/gwb/.local/lib/python3.6/site-packages/torch/serialization.py", line 712, in load
    return _load(opened_zipfile, map_location, pickle_module, **pickle_load_args)
  File "/home/gwb/.local/lib/python3.6/site-packages/torch/serialization.py", line 1046, in _load
    result = unpickler.load()
  File "/home/gwb/.local/lib/python3.6/site-packages/torch/serialization.py", line 1039, in find_class
    return super().find_class(mod_name, name)
  File "yolov5-6.0/models/yolo.py", line 20, in <module>
    from models.common import *
  File "yolov5-6.0/models/common.py", line 17, in <module>
    from PIL import Image
ModuleNotFoundError: No module named 'PIL'
```

问题分析：

- `torch.load` 加载 YOLOv5 `.pt` 权重时，需要反序列化模型类。
- 反序列化会导入 YOLOv5 源码，源码需要 `PIL.Image`。
- 当前环境缺少 Pillow。
- 因为 Python 是 3.6，不能安装最新版 Pillow，应安装兼容版本。

输入命令：

```bash
python3 -m pip install --user 'Pillow<9.0'
```

终端返回：

```text
Collecting Pillow<9.0
  Downloading Pillow-8.4.0-cp36-cp36m-manylinux_2_17_aarch64.manylinux2014_aarch64.whl (3.0 MB)
Installing collected packages: Pillow
Successfully installed Pillow-8.4.0
```

思考：

- 安装成功。
- 版本 `8.4.0` 兼容 Python 3.6 和 aarch64。

### 2. 第二次测试加载模型，发现缺少 tqdm

输入命令：

```bash
python3 -c "import sys, torch; sys.path.insert(0, 'yolov5-6.0'); ckpt=torch.load('yolov5s.pt', map_location='cuda:0'); model=ckpt['model'].float().fuse().eval().to('cuda:0'); print(type(model)); print(next(model.parameters()).device); print(model.names[:5] if isinstance(model.names, list) else list(model.names.items())[:5])"
```

终端返回：

```text
Traceback (most recent call last):
  File "<string>", line 1, in <module>
  File "/home/gwb/.local/lib/python3.6/site-packages/torch/serialization.py", line 712, in load
    return _load(opened_zipfile, map_location, pickle_module, **pickle_load_args)
  File "/home/gwb/.local/lib/python3.6/site-packages/torch/serialization.py", line 1046, in _load
    result = unpickler.load()
  File "/home/gwb/.local/lib/python3.6/site-packages/torch/serialization.py", line 1039, in find_class
    return super().find_class(mod_name, name)
  File "yolov5-6.0/models/yolo.py", line 20, in <module>
    from models.common import *
  File "yolov5-6.0/models/common.py", line 20, in <module>
    from utils.datasets import exif_transpose, letterbox
  File "yolov5-6.0/utils/datasets.py", line 27, in <module>
    from tqdm import tqdm
ModuleNotFoundError: No module named 'tqdm'
```

问题分析：

- YOLOv5 的工具模块导入 `tqdm`。
- 不能直接安装完整 `requirements.txt`，因为其中可能包含 `torch`、`torchvision` 等包，会影响当前可用 CUDA PyTorch。
- 采取最小安装策略，只补缺失的 `tqdm`。

输入命令：

```bash
python3 -m pip show tqdm
```

终端返回：

```text
WARNING: Package(s) not found: tqdm
```

输入命令：

```bash
python3 -m pip show PyYAML
```

终端返回：

```text
Name: PyYAML
Version: 3.12
Summary: YAML parser and emitter for Python
Home-page: http://pyyaml.org/wiki/PyYAML
Author: Kirill Simonov
Author-email: xi@resolvent.net
License: MIT
Location: /usr/lib/python3/dist-packages
Requires:
Required-by:
```

输入命令：

```bash
python3 -m pip show matplotlib
```

终端返回：

```text
Name: matplotlib
Version: 2.1.1
Summary: Python plotting package
Home-page: http://matplotlib.org
Author: John D. Hunter, Michael Droettboom
Author-email: matplotlib-users@python.org
License: BSD
Location: /usr/lib/python3/dist-packages
Requires:
Required-by:
```

输入命令：

```bash
python3 -m pip show pandas
```

终端返回：

```text
Name: pandas
Version: 0.22.0
Summary: Powerful data structures for data analysis, time series,and statistical
Home-page: http://pandas.pydata.org
Author:
Author-email:
License: BSD
Location: /usr/lib/python3/dist-packages
Requires:
Required-by:
```

输入命令：

```bash
python3 -m pip install --user 'tqdm<4.65'
```

终端返回：

```text
Collecting tqdm<4.65
  Downloading tqdm-4.64.1-py2.py3-none-any.whl (78 kB)
Collecting importlib-resources
  Downloading importlib_resources-5.4.0-py3-none-any.whl (28 kB)
Collecting zipp>=3.1.0
  Downloading zipp-3.6.0-py3-none-any.whl (5.3 kB)
Installing collected packages: zipp, importlib-resources, tqdm
Successfully installed importlib-resources-5.4.0 tqdm-4.64.1 zipp-3.6.0
```

思考：

- `tqdm` 安装成功。
- 使用 `<4.65` 是为了保持 Python 3.6 兼容。

### 3. 第三次测试加载模型，发现缺少 torchvision

输入命令：

```bash
python3 -c "import sys, torch; sys.path.insert(0, 'yolov5-6.0'); ckpt=torch.load('yolov5s.pt', map_location='cuda:0'); model=ckpt['model'].float().fuse().eval().to('cuda:0'); print(type(model)); print(next(model.parameters()).device); print(model.names[:5] if isinstance(model.names, list) else list(model.names.items())[:5])"
```

终端返回：

```text
Traceback (most recent call last):
  File "<string>", line 1, in <module>
  File "/home/gwb/.local/lib/python3.6/site-packages/torch/serialization.py", line 712, in load
    return _load(opened_zipfile, map_location, pickle_module, **pickle_load_args)
  File "/home/gwb/.local/lib/python3.6/site-packages/torch/serialization.py", line 1046, in _load
    result = unpickler.load()
  File "/home/gwb/.local/lib/python3.6/site-packages/torch/serialization.py", line 1039, in find_class
    return super().find_class(mod_name, name)
  File "yolov5-6.0/models/yolo.py", line 20, in <module>
    from models.common import *
  File "yolov5-6.0/models/common.py", line 20, in <module>
    from utils.datasets import exif_transpose, letterbox
  File "yolov5-6.0/utils/datasets.py", line 29, in <module>
    from utils.augmentations import Albumentations, augment_hsv, copy_paste, letterbox, mixup, random_perspective
  File "yolov5-6.0/utils/augmentations.py", line 13, in <module>
    from utils.general import colorstr, segment2box, resample_segments, check_version
  File "yolov5-6.0/utils/general.py", line 28, in <module>
    import torchvision
ModuleNotFoundError: No module named 'torchvision'
```

问题分析：

- YOLOv5 v6.0 的 `utils/general.py` 在导入时会 `import torchvision`。
- 当前 Jetson PyTorch 是 `1.11.0a0+17540c5`，普通 pip 安装的 `torchvision` 大概率不匹配，甚至可能覆盖或破坏 CUDA 版 PyTorch。
- 脚本只需要模型结构，不使用 YOLOv5 自带 `torchvision.ops.nms`。
- 采取措施：在 `Yolo/torchvision/` 下添加最小兼容层，提供 `__version__` 和 `ops.nms`，脚本自身实现纯 PyTorch NMS。

### 4. 新增脚本与兼容层

新增文件：

```text
/home/gwb/Fang/Yolo/yolo_realtime_cuda.py
/home/gwb/Fang/Yolo/torchvision/__init__.py
/home/gwb/Fang/Yolo/torchvision/ops.py
/home/gwb/Fang/Yolo/seaborn.py
/home/gwb/Fang/Yolo/YOLO实时检测使用与任务记录.md
```

动作说明：

- `yolo_realtime_cuda.py` 负责摄像头/视频/图片输入、CUDA 推理、NMS、画框、实时窗口展示。
- `torchvision/__init__.py` 与 `torchvision/ops.py` 是本地兼容层，避免安装不匹配的 `torchvision`。
- `seaborn.py` 是本地占位模块，避免可选绘图库阻塞 YOLOv5 权重加载。
- 本 Markdown 文件负责记录使用方法和完整任务过程。

### 5. 第一次运行实时脚本，发现缺少 seaborn

输入命令：

```bash
python3 yolo_realtime_cuda.py --source yolov5-6.0/data/images/bus.jpg --device cuda:0 --no-window --max-frames 1
```

终端返回：

```text
Using CUDA device: cuda:0 (Xavier)
Traceback (most recent call last):
  File "yolo_realtime_cuda.py", line 263, in <module>
    run()
  File "yolo_realtime_cuda.py", line 203, in run
    model, names, stride = load_model(args.weights, device, use_half)
  File "yolo_realtime_cuda.py", line 57, in load_model
    checkpoint = torch.load(str(weights), map_location=device)
  File "/home/gwb/.local/lib/python3.6/site-packages/torch/serialization.py", line 712, in load
    return _load(opened_zipfile, map_location, pickle_module, **pickle_load_args)
  File "/home/gwb/.local/lib/python3.6/site-packages/torch/serialization.py", line 1046, in _load
    result = unpickler.load()
  File "/home/gwb/.local/lib/python3.6/site-packages/torch/serialization.py", line 1039, in find_class
    return super().find_class(mod_name, name)
  File "/home/gwb/Fang/Yolo/yolov5-6.0/models/yolo.py", line 20, in <module>
    from models.common import *
  File "/home/gwb/Fang/Yolo/yolov5-6.0/models/common.py", line 23, in <module>
    from utils.plots import Annotator, colors
  File "/home/gwb/Fang/Yolo/yolov5-6.0/utils/plots.py", line 16, in <module>
    import seaborn as sn
ModuleNotFoundError: No module named 'seaborn'
```

问题分析：

- `seaborn` 是 YOLOv5 绘图分析模块的可选依赖。
- 实时检测脚本不调用 YOLOv5 的 `pairplot`、`histplot` 等训练分析函数。
- 安装完整 `seaborn` 会牵涉旧版 `matplotlib`、`pandas` 依赖，不适合当前 Python 3.6 环境。

采取措施：

- 查看 `yolov5-6.0/utils/plots.py` 中 `seaborn` 使用位置。

输入命令：

```bash
rg -n "seaborn|sn\\." yolov5-6.0/utils/plots.py
```

终端返回：

```text
16:import seaborn as sn
303:    # seaborn correlogram
304:    sn.pairplot(x, corner=True, diag_kind='auto', kind='hist', diag_kws=dict(bins=50), plot_kws=dict(pmax=0.9))
319:    sn.histplot(x, x='x', y='y', ax=ax[2], bins=50, pmax=0.9)
320:    sn.histplot(x, x='width', y='height', ax=ax[3], bins=50, pmax=0.9)
```

输入命令：

```bash
sed -n '1,80p' yolov5-6.0/utils/plots.py
```

终端返回：

```text
# YOLOv5 🚀 by Ultralytics, GPL-3.0 license
"""
Plotting utils
"""

import math
import os
from copy import copy
from pathlib import Path

import cv2
import matplotlib
import matplotlib.pyplot as plt
import numpy as np
import pandas as pd
import seaborn as sn
import torch
from PIL import Image, ImageDraw, ImageFont

from utils.general import user_config_dir, is_ascii, is_chinese, xywh2xyxy, xyxy2xywh
from utils.metrics import fitness

# Settings
CONFIG_DIR = user_config_dir()  # Ultralytics settings dir
RANK = int(os.getenv('RANK', -1))
matplotlib.rc('font', **{'size': 11})
matplotlib.use('Agg')  # for writing to files only
```

思考：

- `seaborn` 只在绘图函数中被调用。
- 采取最小兼容策略：新增 `seaborn.py`，只提供 `pairplot` 和 `histplot` 占位函数。
- 同时在 `yolo_realtime_cuda.py` 中设置 `RANK=1`，避免 YOLOv5 导入 `Annotator` 时尝试下载字体。
- 修正 `--max-frames` 逻辑，使用独立 `total_frames` 计数，避免 FPS 计数归零影响退出条件。

### 6. 第二次运行实时脚本，发现 PyTorch Upsample 兼容问题

输入命令：

```bash
python3 yolo_realtime_cuda.py --source yolov5-6.0/data/images/bus.jpg --device cuda:0 --no-window --max-frames 1
```

终端返回：

```text
Using CUDA device: cuda:0 (Xavier)
Loaded weights: /home/gwb/Fang/Yolo/yolov5s.pt
Model stride: 32, classes: 80
/home/gwb/Fang/Yolo/yolov5-6.0/utils/plots.py:27: UserWarning:
This call to matplotlib.use() has no effect because the backend has already
been chosen; matplotlib.use() must be called *before* pylab, matplotlib.pyplot,
or matplotlib.backends is imported for the first time.

The backend was *originally* set to 'TkAgg' by the following code:
  File "yolo_realtime_cuda.py", line 267, in <module>
    run()
  File "yolo_realtime_cuda.py", line 205, in run
    model, names, stride = load_model(args.weights, device, use_half)
  File "yolo_realtime_cuda.py", line 59, in load_model
    checkpoint = torch.load(str(weights), map_location=device)
  File "/home/gwb/.local/lib/python3.6/site-packages/torch/serialization.py", line 712, in load
    return _load(opened_zipfile, map_location, pickle_module, **pickle_load_args)
  File "/home/gwb/.local/lib/python3.6/site-packages/torch/serialization.py", line 1046, in _load
    result = unpickler.load()
  File "/home/gwb/.local/lib/python3.6/site-packages/torch/serialization.py", line 1039, in find_class
    return super().find_class(mod_name, name)
  File "/home/gwb/Fang/Yolo/yolov5-6.0/models/yolo.py", line 20, in <module>
    from models.common import *
  File "/home/gwb/Fang/Yolo/yolov5-6.0/models/common.py", line 20, in <module>
    from utils.datasets import exif_transpose, letterbox
  File "/home/gwb/Fang/Yolo/yolov5-6.0/utils/datasets.py", line 29, in <module>
    from utils.augmentations import Albumentations, augment_hsv, copy_paste, letterbox, mixup, random_perspective
  File "/home/gwb/Fang/Yolo/yolov5-6.0/utils/augmentations.py", line 13, in <module>
    from utils.general import colorstr, segment2box, resample_segments, check_version
  File "/home/gwb/Fang/Yolo/yolov5-6.0/utils/general.py", line 32, in <module>
    from utils.metrics import box_iou, fitness
  File "/home/gwb/Fang/Yolo/yolov5-6.0/utils/metrics.py", line 10, in <module>
    import matplotlib.pyplot as plt
  File "/usr/lib/python3/dist-packages/matplotlib/pyplot.py", line 72, in <module>
    from matplotlib.backends import pylab_setup
  File "/usr/lib/python3/dist-packages/matplotlib/backends/__init__.py", line 14, in <module>
    line for line in traceback.format_stack()


  matplotlib.use('Agg')  # for writing to files only
Traceback (most recent call last):
  File "yolo_realtime_cuda.py", line 267, in <module>
    run()
  File "yolo_realtime_cuda.py", line 210, in run
    model(dummy)
  File "/home/gwb/.local/lib/python3.6/site-packages/torch/nn/modules/module.py", line 1110, in _call_impl
    return forward_call(*input, **kwargs)
  File "/home/gwb/Fang/Yolo/yolov5-6.0/models/yolo.py", line 126, in forward
    return self._forward_once(x, profile, visualize)  # single-scale inference, train
  File "/home/gwb/Fang/Yolo/yolov5-6.0/models/yolo.py", line 149, in _forward_once
    x = m(x)  # run
  File "/home/gwb/.local/lib/python3.6/site-packages/torch/nn/modules/module.py", line 1110, in _call_impl
    return forward_call(*input, **kwargs)
  File "/home/gwb/.local/lib/python3.6/site-packages/torch/nn/modules/upsampling.py", line 154, in forward
    recompute_scale_factor=self.recompute_scale_factor)
  File "/home/gwb/.local/lib/python3.6/site-packages/torch/nn/modules/module.py", line 1186, in __getattr__
    type(self).__name__, name))
AttributeError: 'Upsample' object has no attribute 'recompute_scale_factor'
```

问题分析：

- 权重已经加载成功，说明前面的依赖兼容处理有效。
- 报错出现在模型 warmup 推理阶段。
- `AttributeError: 'Upsample' object has no attribute 'recompute_scale_factor'` 是 PyTorch 版本和旧模型对象反序列化之间的兼容字段问题。

采取措施：

- 在 `load_model()` 中遍历 `model.modules()`。
- 对所有 `torch.nn.Upsample` 模块，如果没有 `recompute_scale_factor` 属性，就补为 `None`。
- 这样不改变模型结构，只补齐 PyTorch forward 需要读取的字段。

### 7. 第三次运行实时脚本，发现 Detect 网格生成兼容问题

输入命令：

```bash
python3 yolo_realtime_cuda.py --source yolov5-6.0/data/images/bus.jpg --device cuda:0 --no-window --max-frames 1
```

终端返回：

```text
Using CUDA device: cuda:0 (Xavier)
Loaded weights: /home/gwb/Fang/Yolo/yolov5s.pt
Model stride: 32, classes: 80
/home/gwb/Fang/Yolo/yolov5-6.0/utils/plots.py:27: UserWarning:
This call to matplotlib.use() has no effect because the backend has already
been chosen; matplotlib.use() must be called *before* pylab, matplotlib.pyplot,
or matplotlib.backends is imported for the first time.
...
/home/gwb/.local/lib/python3.6/site-packages/torch/functional.py:568: UserWarning: torch.meshgrid: in an upcoming release, it will be required to pass the indexing argument. (Triggered internally at  /opt/package_build/pytorch/aten/src/ATen/native/TensorShape.cpp:2157.)
  return _VF.meshgrid(tensors, **kwargs)  # type: ignore[attr-defined]
Traceback (most recent call last):
  File "yolo_realtime_cuda.py", line 270, in <module>
    run()
  File "yolo_realtime_cuda.py", line 213, in run
    model(dummy)
  File "/home/gwb/.local/lib/python3.6/site-packages/torch/nn/modules/module.py", line 1110, in _call_impl
    return forward_call(*input, **kwargs)
  File "/home/gwb/Fang/Yolo/yolov5-6.0/models/yolo.py", line 126, in forward
    return self._forward_once(x, profile, visualize)  # single-scale inference, train
  File "/home/gwb/Fang/Yolo/yolov5-6.0/models/yolo.py", line 149, in _forward_once
    x = m(x)  # run
  File "/home/gwb/.local/lib/python3.6/site-packages/torch/nn/modules/module.py", line 1110, in _call_impl
    return forward_call(*input, **kwargs)
  File "/home/gwb/Fang/Yolo/yolov5-6.0/models/yolo.py", line 61, in forward
    self.grid[i], self.anchor_grid[i] = self._make_grid(nx, ny, i)
RuntimeError: The expanded size of the tensor (1) must match the existing size (80) at non-singleton dimension 3.  Target sizes: [1, 3, 1, 1, 2].  Tensor sizes: [3, 80, 80, 2]
```

问题分析：

- 报错发生在 `yolov5-6.0/models/yolo.py` 的 `Detect._make_grid()`。
- 源码中 `torch.stack((xv, yv), 2)` 的形状是 `(ny, nx, 2)`。
- 当前代码直接 `expand((1, self.na, ny, nx, 2))`，在当前 PyTorch 上不能自动变成 `(1, 1, ny, nx, 2)` 后再扩展。

采取措施：

- 不直接修改下载的 YOLOv5 源码。
- 在 `yolo_realtime_cuda.py` 中新增 `make_grid_compatible()`。
- 加载模型后找到类名为 `Detect` 的层，使用 `types.MethodType` 替换该层的 `_make_grid()`。
- 新实现先执行 `view(1, 1, ny, nx, 2)`，再 `expand(1, self.na, ny, nx, 2)`。
- 同时设置 `MPLBACKEND=Agg`，减少 YOLOv5 导入 matplotlib 时的图形后端干扰。

### 8. 检查 Detect 层实际属性，确认 anchor_grid 是旧 buffer

输入命令：

```bash
python3 -c "import sys, os, torch; sys.path.insert(0,'.'); sys.path.insert(0,'yolov5-6.0'); os.environ['RANK']='1'; import yolo_realtime_cuda as y; ckpt=torch.load('yolov5s.pt', map_location='cuda:0'); model=(ckpt.get('ema') or ckpt['model']).float().fuse().eval().to('cuda:0'); print([(m.__class__.__name__, hasattr(m,'_make_grid'), getattr(m,'nl',None)) for m in model.modules() if hasattr(m,'_make_grid')]); y.load_model('yolov5s.pt', torch.device('cuda:0'), False);"
```

终端返回：

```text
[('Detect', True, 3)]
Loaded weights: yolov5s.pt
Model stride: 32, classes: 80
/home/gwb/Fang/Yolo/yolov5-6.0/utils/plots.py:27: UserWarning:
This call to matplotlib.use() has no effect because the backend has already
been chosen; matplotlib.use() must be called *before* pylab, matplotlib.pyplot,
or matplotlib.backends is imported for the first time.
...
```

思考：

- 模型里确实有一个 `Detect` 层，`nl=3`。
- 需要继续检查 `grid` 和 `anchor_grid` 类型。

输入命令：

```bash
python3 -c "import torch, yolo_realtime_cuda as y; model, names, stride = y.load_model('yolov5s.pt', torch.device('cuda:0'), False); d=[m for m in model.modules() if hasattr(m,'_make_grid')][0]; print(type(d.grid), d.grid); print(type(d.anchor_grid), getattr(d.anchor_grid, 'shape', None), d.anchor_grid if not isinstance(d.anchor_grid, list) else 'list');"
```

终端返回摘要：

```text
Loaded weights: yolov5s.pt
Model stride: 32, classes: 80
<class 'list'> [...]
<class 'torch.Tensor'> torch.Size([3, 1, 3, 1, 1, 2]) tensor([...], device='cuda:0')
```

问题分析：

- `grid` 已是 list，但 `anchor_grid` 是 tensor。
- `Detect.forward()` 中代码是 `self.grid[i], self.anchor_grid[i] = self._make_grid(...)`。
- 如果 `anchor_grid` 是 tensor，则 `self.anchor_grid[i] = ...` 会变成对 tensor 切片赋值，导致形状不匹配。

采取措施：

- 加载模型后，对 Detect 层执行：
  - 如果 `anchor_grid` 在 `_buffers` 中，先删除该 buffer。
  - 重建 `module.grid = [torch.zeros(1, device=device)] * module.nl`。
  - 重建 `module.anchor_grid = [torch.zeros(1, device=device)] * module.nl`。
  - 再替换 `_make_grid()`。
- 这样让运行时结构和 YOLOv5 v6.0 源码期望一致。
