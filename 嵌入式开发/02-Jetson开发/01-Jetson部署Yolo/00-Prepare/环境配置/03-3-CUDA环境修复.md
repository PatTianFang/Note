# CUDA 环境修复

当前设备环境：

- Jetson/Tegra
- Ubuntu 18.04
- JetPack 4.6
- CUDA 10.2
- Python 3.6
- aarch64

## 修 CUDA 环境变量

底层 CUDA 是好的，只是 `nvcc` 不在当前 `PATH` 中。

执行：

```bash
echo 'export PATH=/usr/local/cuda/bin:$PATH' >> ~/.bashrc
echo 'export LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH' >> ~/.bashrc
source ~/.bashrc
nvcc --version
```

输出可以直接看到 CUDA 10.2 的版本信息。
