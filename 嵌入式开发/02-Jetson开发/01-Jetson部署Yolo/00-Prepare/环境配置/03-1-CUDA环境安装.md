## Python CUDA 开发

如果要用 Python 做深度学习或 GPU 计算，建议根据实际需求安装对应库：

- 深度学习训练/推理：安装 JetPack 4.6 兼容的 PyTorch 或 TensorFlow。
- TensorRT 推理：安装 Python TensorRT。
- NumPy 风格 GPU 计算：安装 JetPack/CUDA 10.2/aarch64 兼容的 CuPy。
- 自定义 CUDA kernel：安装 PyCUDA，或直接使用 C++/CUDA 扩展。

注意：不要直接使用通用的 `pip install torch` 或 x86 wheel。Jetson 需要使用 NVIDIA 或社区提供的 aarch64 专用 wheel，且版本必须匹配 JetPack 4.6 和 CUDA 10.2。

## 工具安装方式

本节命令按当前设备环境编写：Jetson/Tegra、Ubuntu 18.04、JetPack 4.6、L4T R32.6.1、CUDA 10.2、Python 3.6、aarch64。

执行安装前建议先确认版本：

```bash
cat /etc/nv_tegra_release
dpkg -l | grep nvidia-jetpack
python3 --version
/usr/local/cuda/bin/nvcc --version
```

### JetPack 整套组件

Jetson 上推荐优先通过 JetPack 安装 CUDA、cuDNN、TensorRT、OpenCV、VPI、VisionWorks、容器运行时等组件。

安装整套 JetPack：

```bash
sudo apt update
sudo apt install nvidia-jetpack
```

如果磁盘空间紧张，也可以只安装 `nvidia-jetpack` 依赖项：

```bash
sudo apt update
apt depends nvidia-jetpack | awk '{print $2}' | xargs -I {} sudo apt install -y {}
```

验证：

```bash
dpkg -l | grep nvidia-jetpack
```

参考：

- NVIDIA JetPack 4.6 安装文档：`https://docs.nvidia.com/jetson/jetpack/4.6/install-jetpack/index.html`
- NVIDIA JetPack 4.6 发布页：`https://developer.nvidia.com/embedded/jetpack-sdk-46`

JetPack 是 NVIDIA 为 Jetson 嵌入式平台提供的完整 AI 软件栈，含以下核心组件：

| 组件 | 简介 | 典型用途 |
|------|------|----------|
| **CUDA (Compute Unified Device Architecture)** | NVIDIA 提供的通用并行计算平台和编程模型，扩展 C/C++ 支持 GPU 加速。CUDA 是所有 NVIDIA GPU 计算的基础。 | 编写自定义 GPU kernel、科学计算、深度学习框架底层加速 |
| **cuDNN (CUDA Deep Neural Network Library)** | GPU 加速的深度学习原语库，提供卷积、池化、归一化、激活等高性能实现。 | 深度学习训练/推理加速（PyTorch、TensorFlow 底层调用） |
| **TensorRT** | 高性能深度学习推理引擎，支持算子融合、动态张量、FP16/INT8 量化等优化。 | 生产环境实时推理，低延迟高吞吐 |
| **OpenCV (Open Computer Vision)** | 主流计算机视觉库，含图像/视频处理、特征检测、目标跟踪等功能。JetPack 版本已启用 CUDA 加速。 | 视觉预处理、相机流、目标检测前后处理 |
| **VPI (Vision Programming Interface)** | NVIDIA 提供的多后端视觉处理 API，支持 GPU、PVP (Programmable Vision Processing)、CPU 后端。 | 立体匹配、背景移除、畸变校正等嵌入式视觉任务 |
| **VisionWorks** | 基于 OpenVX 的图像和视觉处理框架，提供可扩展的 SFM (Structure from Motion) 等高级功能。 | 立体视觉、SLAM、3D 重建 |
| **容器运行时 (NVIDIA Container Runtime)** | 使 Docker 容器能够访问宿主 GPU，通过 `nvidia-container-toolkit` 实现 `--gpus` 支持。 | 容器化部署 AI 应用，环境隔离与可移植性 |

> **为什么通过 JetPack 统一安装？** 这些组件之间存在严格的版本兼容性要求（如 cuDNN 需与 CUDA 版本对应，TensorRT 需与 cuDNN/CUDA 匹配）。手动逐个安装容易出现版本冲突，JetPack 可确保整体版本一致。



### 安装后的总体验证

完成安装后建议重新运行以下检查：

```bash
nvcc --version
python3 - <<'PY'
mods = ['numpy', 'torch', 'tensorflow', 'cupy', 'cv2', 'tensorrt', 'pycuda']
for name in mods:
    try:
        mod = __import__(name)
        print(name, 'OK', getattr(mod, '__version__', 'unknown'))
    except Exception as exc:
        print(name, 'FAIL', type(exc).__name__, exc)
PY
```

再运行 CUDA samples：

```bash
cp -r /usr/local/cuda/samples /tmp/cuda-samples-check
make -C /tmp/cuda-samples-check/1_Utilities/deviceQuery
/tmp/cuda-samples-check/1_Utilities/deviceQuery/deviceQuery
```

预期结果包含：

```text
Detected 1 CUDA Capable device(s)
Result = PASS
```
