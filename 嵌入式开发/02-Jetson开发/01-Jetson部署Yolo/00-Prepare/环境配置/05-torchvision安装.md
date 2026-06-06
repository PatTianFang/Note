

## torchvision 安装

当前 PyTorch 已安装成功，但 `torchvision` 缺失：

```bash
python3 - <<'PY'
try:
    import torchvision
    print("torchvision:", torchvision.__version__)
except Exception as e:
    print("torchvision_import_error:", repr(e))
PY
```

出现：

```text
torchvision_import_error: ModuleNotFoundError("No module named 'torchvision'",)
```

### 当前环境

检查当前 Python、PyTorch、CUDA、L4T 和架构：

```bash
python3 - <<'PY'
import sys
import torch
print("python:", sys.version)
print("torch:", torch.__version__)
print("cuda:", torch.version.cuda)
print("cuda_available:", torch.cuda.is_available())
PY

cat /etc/nv_tegra_release
python3 -m pip --version
uname -m
```

输出：

```text
python: 3.6.9
torch: 1.11.0a0+17540c5
cuda: 10.2
cuda_available: True
L4T R32.6.1
pip 21.3.1
aarch64
```

### 不能直接安装普通 pip 版本

Jetson 上的 PyTorch 是 NVIDIA 针对 JetPack、CUDA、Python 和 aarch64 架构构建的版本，不能随意执行：

```bash
python3 -m pip install torchvision
```

- 普通 PyPI 版本可能不匹配 NVIDIA 版 `torch`。
- pip 可能会尝试替换已经安装好的 NVIDIA PyTorch。
- `torchvision` 的 C++/CUDA 扩展需要和当前 `torch` ABI 匹配。

安装时必须使用 `--no-deps`，避免 pip 自动改动 `torch`。

### 尝试 torchvision 0.12.0

`torch 1.11` 通常对应 `torchvision 0.12`，因此先尝试构建 `v0.12.0`：

```bash
env BUILD_VERSION=0.12.0 FORCE_CUDA=1 python3 -m pip install --user --no-deps --no-cache-dir https://github.com/pytorch/vision/archive/refs/tags/v0.12.0.tar.gz
```

第一次失败原因：

```text
ModuleNotFoundError: No module named 'torch'
```

原因是 pip 默认使用 PEP 517 隔离构建环境，在隔离环境中看不到当前已经安装的 NVIDIA `torch`。

因此改用 `--no-build-isolation`：

```bash
env BUILD_VERSION=0.12.0 FORCE_CUDA=1 python3 -m pip install --user --no-deps --no-cache-dir --no-build-isolation https://github.com/pytorch/vision/archive/refs/tags/v0.12.0.tar.gz
```

第二次失败原因：

```text
ERROR: Package 'torchvision' requires a different Python: 3.6.9 not in '>=3.7'
```

结论：当前系统是 Python 3.6.9，不能安装 `torchvision 0.12.0`。如果要严格使用 `torch 1.11 + torchvision 0.12`，需要升级到 Python 3.7 或更新的 JetPack/Python 环境。但在当前 JetPack 4.x / Python 3.6 环境下，不适合这样做。

### 最终安装 torchvision 0.11.3

选择仍可在 Python 3.6 环境下构建的 `torchvision 0.11.3`：

```bash
env BUILD_VERSION=0.11.3 FORCE_CUDA=1 python3 -m pip install --user --no-deps --no-cache-dir --no-build-isolation https://github.com/pytorch/vision/archive/refs/tags/v0.11.3.tar.gz
```

实测构建成功：

```text
Successfully built torchvision
Successfully installed torchvision-0.11.3
```

生成的 wheel：

```text
torchvision-0.11.3-cp36-cp36m-linux_aarch64.whl
```

说明：

- `cp36-cp36m`：匹配 Python 3.6。
- `linux_aarch64`：匹配 Jetson ARM64。
- `--no-deps`：避免 pip 替换 NVIDIA 版 PyTorch。
- `--no-build-isolation`：让构建过程能使用当前环境中的 `torch`。
- `FORCE_CUDA=1`：强制构建 CUDA 扩展。

### 验证 torchvision

验证导入、版本和 CUDA：

```bash
python3 - <<'PY'
import torch
import torchvision
print("torch:", torch.__version__)
print("torchvision:", torchvision.__version__)
print("cuda available:", torch.cuda.is_available())
PY
```

实测结果：

```text
torch: 1.11.0a0+17540c5
torchvision: 0.11.3
cuda available: True
```

验证 `torchvision.ops.nms` 的 CUDA 扩展：

```bash
python3 - <<'PY'
import torch
from torchvision.ops import nms
boxes = torch.tensor([[0., 0., 10., 10.], [1., 1., 11., 11.]], device="cuda")
scores = torch.tensor([0.9, 0.8], device="cuda")
print(nms(boxes, scores, 0.5).cpu().tolist())
PY
```

实测结果：

```text
[0]
```

结论：`torchvision` 已安装成功，且核心 C++/CUDA ops 可用。

### torchvision.io.image 警告

导入时仍可能出现警告：

```text
UserWarning: Failed to load image Python extension:
```

检查发现系统中只有 `zlib1g-dev`，缺少 `libjpeg-dev` 和 `libpng-dev`：

```bash
dpkg -l libjpeg-dev zlib1g-dev libpng-dev
```

原因：构建 `torchvision` 时缺少 JPEG/PNG 开发库，因此可选的 `torchvision.io.image` 图片 C 扩展没有生成。该警告不影响 `torchvision` 主体、models、datasets、transforms 和 `torchvision.ops` 的基本使用；如果使用 PIL 读取图片，通常也可以继续工作。

如需修复该警告，需要先在有 sudo 的终端中安装系统依赖：

```bash
sudo apt-get update
sudo apt-get install -y libjpeg-dev libpng-dev
```

然后重新编译安装：

```bash
python3 -m pip uninstall -y torchvision
env BUILD_VERSION=0.11.3 FORCE_CUDA=1 python3 -m pip install --user --no-deps --no-cache-dir --no-build-isolation https://github.com/pytorch/vision/archive/refs/tags/v0.11.3.tar.gz
```

因此本次只完成了 `torchvision 0.11.3` 主体安装和 CUDA ops 验证，图片 C 扩展警告留待安装系统依赖后重新编译处理。

## 参考文档

- NVIDIA PyTorch for Jetson: `https://docs.nvidia.com/deeplearning/frameworks/install-pytorch-jetson-platform/index.html`
- NVIDIA PyTorch for Jetson release notes: `https://docs.nvidia.com/deeplearning/frameworks/install-pytorch-jetson-platform-release-notes/pytorch-jetson-rel.html`
- PyTorch torchvision version compatibility: `https://github.com/pytorch/vision#installation`
