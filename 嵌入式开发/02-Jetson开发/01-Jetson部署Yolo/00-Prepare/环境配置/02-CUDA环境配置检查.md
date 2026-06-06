# CUDA 环境配置检查

## 系统与硬件

- 系统：Ubuntu 18.04.6 LTS
- 内核：`4.9.253-tegra`
- 架构：`aarch64`
- L4T：`R32.6.1`
- JetPack：`4.6-b199`
- GPU：Xavier
- CUDA Compute Capability：`7.2`
- CUDA 可见设备数：`1`
- 全局内存：约 `31920 MB`
- CUDA 核心数：`512`

说明：这是 Jetson/Tegra 设备，不是普通 x86 NVIDIA 桌面机。因此 `nvidia-smi` 不存在是常见情况，不能仅凭 `nvidia-smi` 判断 CUDA 不可用。

## CUDA Toolkit

直接运行 `nvcc`：

```bash
nvcc --version
```

结果：失败，原因是 `nvcc` 没有加入当前 `PATH`。

使用完整路径运行：

```bash
/usr/local/cuda/bin/nvcc --version
```

结果：成功。

检测到的 CUDA 版本：

```text
Cuda compilation tools, release 10.2, V10.2.300
```

已安装的 CUDA 相关组件包括：

- `cuda-toolkit-10-2`
- `cuda-compiler-10-2`
- `cuda-command-line-tools-10-2`
- `cuda-libraries-10-2`
- `cuda-libraries-dev-10-2`
- `cuda-cudart-10-2`
- `cuda-cudart-dev-10-2`
- `cuda-nvcc-10-2`
- `cuda-samples-10-2`

## CUDA Runtime 验证

已将 CUDA samples 中的 `deviceQuery` 复制到临时目录编译运行。

结果：

```text
Detected 1 CUDA Capable device(s)
Device 0: "Xavier"
CUDA Driver Version / Runtime Version 10.2 / 10.2
CUDA Capability Major/Minor version number: 7.2
Result = PASS
```

结论：底层 CUDA runtime 正常，设备可以被 CUDA 程序识别并使用。

## cuDNN

已安装：

- `libcudnn8`
- `libcudnn8-dev`
- `libcudnn8-samples`

版本：

```text
8.2.1.32-1+cuda10.2
```

结论：cuDNN runtime、开发头文件和示例均已安装。

## TensorRT

已安装：

- `tensorrt`
- `libnvinfer8`
- `libnvinfer-dev`
- `libnvinfer-plugin8`
- `libnvinfer-plugin-dev`
- `libnvonnxparsers8`
- `libnvonnxparsers-dev`
- `python3-libnvinfer`
- `python3-libnvinfer-dev`

版本：

```text
8.0.1.6-1+cuda10.2
```

Python 检查结果：

```text
tensorrt: OK version=8.0.1.6
```

结论：TensorRT C++/Python 开发环境可用。

## OpenCV

已安装：

- `libopencv`
- `libopencv-dev`
- `libopencv-python`
- `nvidia-opencv`

Python 检查结果：

```text
cv2: OK version=4.1.1
cv2 CUDA devices=0
```

结论：OpenCV Python 可用，但当前 Python OpenCV 看起来没有可用的 CUDA 支持。若项目需要 `cv2.cuda`，可能需要确认 OpenCV 是否以 CUDA 支持重新编译。

## Python 环境

Python 版本：

```text
Python 3.6.9
```

Python 路径：

```text
/usr/bin/python3
```

pip 版本：

```text
pip 20.3.4 from /home/gwb/.local/lib/python3.6/site-packages/pip
```

Python 开发头文件：

```text
/usr/include/python3.6m
```

已安装：

- `python3-dev`
- `python3.6-dev`
- `libpython3-dev`
- `libpython3.6-dev`

说明：Python 3.6.9 是 JetPack 4.6 常见环境，但版本较旧。安装 PyTorch、TensorFlow、CuPy 等库时，需要选择兼容 JetPack 4.6、CUDA 10.2、aarch64、Python 3.6 的版本，不能直接使用普通 x86 Linux 的安装包。

## Python CUDA 相关库检查

检查结果：

```text
numpy: OK version=1.13.3
torch: FAIL ModuleNotFoundError: No module named 'torch'
tensorflow: FAIL ModuleNotFoundError: No module named 'tensorflow'
cupy: FAIL ModuleNotFoundError: No module named 'cupy'
cv2: OK version=4.1.1
tensorrt: OK version=8.0.1.6
pycuda: FAIL ModuleNotFoundError: No module named 'pycuda'
```

结论：

- TensorRT Python 可用。
- PyTorch 未安装。
- TensorFlow 未安装。
- CuPy 未安装。
- PyCUDA 未安装。
- OpenCV Python 可用，但未检测到 CUDA device。

## C/C++ 开发工具

已确认可用：

```text
gcc 7.5.0
g++ 7.5.0
GNU Make 4.1
CMake 3.10.2
```

结论：C/C++ CUDA 开发基础工具可用。

## 当前主要问题

1. `nvcc` 没有加入当前 `PATH`。
2. 当前 `LD_LIBRARY_PATH` 为空，不过 CUDA 库已被 `ldconfig` 识别，底层运行没有问题。
3. Python 侧缺少常用 CUDA/深度学习库。
4. OpenCV Python 未检测到 CUDA device。
5. Python 版本较旧，安装第三方库时需要特别注意 JetPack 4.6 兼容性。

## 建议配置

建议把 CUDA 加入 shell 环境变量。

临时生效：

```bash
export PATH=/usr/local/cuda/bin:$PATH
export LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH
```

长期生效可以加入 `~/.bashrc`：

```bash
echo 'export PATH=/usr/local/cuda/bin:$PATH' >> ~/.bashrc
echo 'export LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH' >> ~/.bashrc
source ~/.bashrc
```

添加后验证：

```bash
nvcc --version
```

预期可以直接看到 CUDA 10.2 的版本信息。
