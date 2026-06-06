# PyTorch CUDA 安装

Jetson 不能直接使用普通 x86 Linux 的 PyTorch wheel。需要使用 NVIDIA 提供的 Jetson/aarch64 wheel，并匹配 JetPack 版本。

不要直接执行：

```bash
pip install torch
```

应安装 NVIDIA 提供的 Jetson/aarch64/CUDA 兼容 wheel。

## 1. 配置 CUDA 环境变量

先确保 `nvcc` 可以直接使用。

```bash
echo 'export PATH=/usr/local/cuda/bin:$PATH' >> ~/.bashrc
echo 'export LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH' >> ~/.bashrc
echo 'export OPENBLAS_CORETYPE=ARMV8' >> ~/.bashrc
source ~/.bashrc
nvcc --version
```

可以看到 CUDA 10.2 的版本信息。

## 2. 安装 PyTorch 依赖

```bash
sudo apt update
sudo apt install -y libopenblas-base libopenmpi-dev libomp-dev python3-dev build-essential wget
python3 -m pip install --user --upgrade 'pip<22' setuptools wheel
python3 -m pip install --user aiohttp 'numpy==1.19.4' 'scipy==1.5.3'
```

说明：当前系统 Python 是 3.6，较新的 pip 已经不再支持 Python 3.6，因此这里限制为 `pip<22`。

## 3. 下载 Jetson PyTorch wheel

当前可用的 NVIDIA Jetson PyTorch wheel：

```text
torch-1.11.0a0+17540c5+nv22.01-cp36-cp36m-linux_aarch64.whl
```

该 wheel 的含义：

- `torch-1.11.0a0`：PyTorch 版本。
- `nv22.01`：NVIDIA 发布版本。
- `cp36-cp36m`：匹配 Python 3.6。
- `linux_aarch64`：匹配 Jetson ARM64。
- `jp/v461`：NVIDIA JetPack 4.6.1 下载目录。

设置下载变量：

```bash
export TORCH_WHL=torch-1.11.0a0+17540c5+nv22.01-cp36-cp36m-linux_aarch64.whl
export TORCH_URL=https://developer.download.nvidia.com/compute/redist/jp/v461/pytorch/${TORCH_WHL}
```

下载：

```bash
wget -O ${TORCH_WHL} ${TORCH_URL}
```

如果下载慢或失败，可以使用中国区地址：

```bash
export TORCH_URL=https://developer.download.nvidia.cn/compute/redist/jp/v461/pytorch/${TORCH_WHL}
wget -O ${TORCH_WHL} ${TORCH_URL}
```

## 4. 安装 PyTorch

如果之前安装过不兼容的 PyTorch，先卸载：

```bash
python3 -m pip uninstall -y torch torchvision torchaudio
```

安装 wheel：

```bash
python3 -m pip install --user --no-cache-dir ${TORCH_WHL}
```

## 5. 验证 CUDA 是否可用

```bash
OPENBLAS_CORETYPE=ARMV8 python3 - <<'PY'
import torch
print("torch:", torch.__version__)
print("cuda version:", torch.version.cuda)
print("cuda available:", torch.cuda.is_available())
if torch.cuda.is_available():
    print("device:", torch.cuda.get_device_name(0))
PY
```
结果：

```text
torch: 1.11.0a0+17540c5
cuda version: 10.2
cuda available: True
device: Xavier
```

如果没有设置 `OPENBLAS_CORETYPE=ARMV8`，可能出现：

```text
非法指令 (核心已转储)
```

如果 `torch.cuda.is_available()` 是 `False`，或仍然出现非法指令，先检查：

```bash
echo $OPENBLAS_CORETYPE
nvcc --version
python3 -m pip show torch
python3 -m pip show numpy
ldconfig -p | grep libcudart
```

## 6. 版本注意事项

当前设备检测到的是 JetPack 4.6，但 NVIDIA 可下载的 PyTorch wheel 位于 `v461` 目录，也就是 JetPack 4.6.1。

JetPack 4.6 和 4.6.1 同属 L4T R32.x / CUDA 10.2 系列，通常可以尝试使用该 wheel。但最稳妥的方式是升级到 JetPack 4.6.1 后再安装。

如果安装失败，不建议继续混装其他 PyTorch wheel，应先确认：

- 当前 JetPack/L4T 版本。
- Python 是否为 3.6。
- wheel 是否为 `cp36-cp36m-linux_aarch64`。
- CUDA 是否为 10.2。
- 是否已经正确配置 CUDA 环境变量。




## 参考文档

- NVIDIA PyTorch for Jetson: `https://docs.nvidia.com/deeplearning/frameworks/install-pytorch-jetson-platform/index.html`
- NVIDIA PyTorch for Jetson release notes: `https://docs.nvidia.com/deeplearning/frameworks/install-pytorch-jetson-platform-release-notes/pytorch-jetson-rel.html`
- PyTorch torchvision version compatibility: `https://github.com/pytorch/vision#installation`
