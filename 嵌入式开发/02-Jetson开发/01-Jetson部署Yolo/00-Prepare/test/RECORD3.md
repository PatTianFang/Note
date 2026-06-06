# Jetson CUDA 开发环境检查与 CUDA YOLO 脚本记录

记录文件：`/home/gwb/Fang/RECORD3.md`

CUDA YOLO 脚本：`/home/gwb/Fang/yolo_cuda_realtime.py`

桌面说明：`/home/gwb/Desktop/yolo_cuda_usage.md`

## 最终结论

1. 这台设备当前系统信息不符合“Jetson AGX Orin”的典型环境。
   - 当前 L4T：`R32.6.1`
   - 当前内核：`4.9.253-tegra`
   - 当前 Ubuntu：`18.04.6`
   - 设备树：`Jetson-AGX`
   - nvpmodel：`t194`
   - AGX Orin 通常应是 t234 + L4T R35/R36 + Ubuntu 20.04/22.04。
   - 因此当前系统更像 Jetson AGX Xavier / JetPack 4.6 环境。

2. 系统原本没有完整 CUDA 开发套件。
   - 没有 `/usr/local/cuda`
   - 没有 `nvcc`
   - 没有 `libcudart`
   - 没有 cuDNN
   - 没有 TensorRT
   - 没有 PyCUDA / CuPy / Numba / PyTorch
   - 只有 Jetson 驱动侧 `libcuda.so`

3. 标准系统安装不可用。
   - 执行 `sudo apt-get install -y nvidia-jetpack` 失败。
   - 原因是当前系统 `sudo` 受 `nosuid` 限制，不能获得 root 权限。

4. 已采取用户目录安装方案。
   - 下载并解包最小 CUDA 10.2 工具链到：

```bash
/home/gwb/Fang/cuda_local/usr/local/cuda-10.2
```

   - 包含 `nvcc`、cudart、cublas、curand 和对应头文件。
   - 已用 `nvcc` 编译并运行 CUDA kernel smoke test。
   - 输出：`CUDA result: 11.0 22.0 33.0 44.0`

5. 已编译 GPU 版 Darknet。
   - GPU 版动态库：

```bash
/home/gwb/Fang/third_party/darknet-gpu/libdarknet.so
```

   - Python CUDA YOLO 脚本：

```bash
/home/gwb/Fang/yolo_cuda_realtime.py
```

6. 已验证 CUDA YOLO 脚本。
   - 静态图检测成功，检出 6 个目标。
   - 摄像头单帧处理成功。
   - 实时窗口启动成功。

## 使用方法

实时 CUDA YOLO 检测：

```bash
cd /home/gwb/Fang
./yolo_cuda_realtime.py --skip-frames 10 --thresh 0.25
```

摄像头单帧测试：

```bash
cd /home/gwb/Fang
./yolo_cuda_realtime.py --test-only --save-frame yolo_cuda_test.jpg
```

静态图测试：

```bash
cd /home/gwb/Fang
./yolo_cuda_realtime.py --image third_party/darknet-gpu/data/dog.jpg --test-only --save-frame yolo_cuda_dog_result.jpg
```

CUDA smoke test：

```bash
cd /home/gwb/Fang
LD_LIBRARY_PATH=/home/gwb/Fang/cuda_local/usr/local/cuda-10.2/lib64:/usr/lib/aarch64-linux-gnu/tegra ./cuda_smoke_test
```

## 详细过程记录

### 1. 检查系统、L4T、CUDA 可执行文件

目的：确认这台设备的 Jetson/L4T 版本、系统版本、CUDA 是否已安装。

#### 命令 1

```bash
uname -a
```

返回码：0

输出：

```text
Linux gwb 4.9.253-tegra #6 SMP PREEMPT Sun Feb 20 17:12:35 CST 2022 aarch64 aarch64 aarch64 GNU/Linux
```

思考：内核是 tegra 内核，说明是 Jetson 系统；但内核 4.9 属于较老 JetPack 4.x 系列，不像 AGX Orin 常见环境。

#### 命令 2

```bash
cat /etc/os-release
```

返回码：0

输出：

```text
NAME="Ubuntu"
VERSION="18.04.6 LTS (Bionic Beaver)"
ID=ubuntu
ID_LIKE=debian
PRETTY_NAME="Ubuntu 18.04.6 LTS"
VERSION_ID="18.04"
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
VERSION_CODENAME=bionic
UBUNTU_CODENAME=bionic
```

#### 命令 3

```bash
cat /etc/nv_tegra_release
```

返回码：0

输出：

```text
# R32 (release), REVISION: 6.1, GCID: 27863751, BOARD: t186ref, EABI: aarch64, DATE: Mon Jul 26 19:36:31 UTC 2021
```

思考：L4T R32.6.1 对应 JetPack 4.6 系列；AGX Orin 通常不应是 R32。

#### 命令 4

```bash
ls -l /usr/local
```

返回码：0

输出：

```text
总用量 32
drwxr-xr-x 2 root root 4096 6月   1 08:20 bin
drwxr-xr-x 2 root root 4096 4月  27  2018 etc
drwxr-xr-x 2 root root 4096 4月  27  2018 games
drwxr-xr-x 2 root root 4096 4月  27  2018 include
drwxr-xr-x 4 root root 4096 6月   4  2018 lib
lrwxrwxrwx 1 root root    9 4月  27  2018 man -> share/man
drwxr-xr-x 2 root root 4096 4月  27  2018 sbin
drwxr-xr-x 7 root root 4096 5月  21  2018 share
drwxr-xr-x 2 root root 4096 4月  27  2018 src
```

思考：没有 `/usr/local/cuda`，CUDA Toolkit 未按标准方式安装。

#### 命令 5

```bash
command -v nvcc
```

返回码：1

输出：

```text

```

思考：系统 PATH 中没有 `nvcc`。

#### 命令 6

```bash
command -v nvidia-smi
```

返回码：1

输出：

```text

```

说明：Jetson 上通常没有桌面显卡环境那种 `nvidia-smi`，不能仅凭该项判断 GPU 不可用。

### 2. 检查设备树、Jetson 包和 CUDA 驱动库

#### 命令 7

```bash
tr -d '\0' < /proc/device-tree/model
```

返回码：0

输出：

```text
Jetson-AGX
```

思考：设备树只显示 `Jetson-AGX`，未显示 Orin。

#### 命令 8

```bash
ls -l /etc/nv*
```

返回码：0

输出：

```text
-rw-r--r-- 1 root root  208 7月  22  2021 /etc/nv_boot_control.conf
-rw-r--r-- 1 root root 1804 7月  27  2021 /etc/nv-oem-config.conf
-rw-r--r-- 1 root root 2916 7月  27  2021 /etc/nvphsd_common.conf
-rw-r--r-- 1 root root 1767 7月  27  2021 /etc/nvphsd.conf.t186
-rw-r--r-- 1 root root 1767 7月  27  2021 /etc/nvphsd.conf.t194
lrwxrwxrwx 1 root root   32 7月  22  2021 /etc/nvpmodel.conf -> /etc/nvpmodel/nvpmodel_t194.conf
-rw-r--r-- 1 root root  114 7月  27  2021 /etc/nv_tegra_release

/etc/nv:
总用量 0

/etc/nvidia-container-runtime:
总用量 4
drwxr-xr-x 2 root root 4096 6月   1 08:22 host-files-for-container.d

/etc/nvpmodel:
总用量 112
-rw-r--r-- 1 root root  5436 7月  27  2021 nvpmodel_t186.conf
-rw-r--r-- 1 root root  4579 7月  27  2021 nvpmodel_t186_p3636.conf
-rw-r--r-- 1 root root  4577 7月  27  2021 nvpmodel_t186_storm_ucm1.conf
-rw-r--r-- 1 root root  4575 7月  27  2021 nvpmodel_t186_storm_ucm2.conf
-rw-r--r-- 1 root root  7012 7月  27  2021 nvpmodel_t194_8gb.conf
-rw-r--r-- 1 root root  8431 7月  27  2021 nvpmodel_t194_agxi.conf
-rw-r--r-- 1 root root  9678 7月  22  2021 nvpmodel_t194.conf
-rw-r--r-- 1 root root  4347 7月  27  2021 nvpmodel_t194_e3900_dGPU.conf
-rw-r--r-- 1 root root  9677 7月  27  2021 nvpmodel_t194_e3900_iGPU.conf
-rw-r--r-- 1 root root 12895 7月  27  2021 nvpmodel_t194_p3668.conf
-rw-r--r-- 1 root root 12246 7月  27  2021 nvpmodel_t194_p3668_emul.conf
```

思考：nvpmodel 指向 `t194`，这进一步说明当前系统是 Xavier 级配置，不是 Orin 的 t234。

#### 命令 9

```bash
dpkg -l | grep -E 'nvidia-jetpack|cuda|cudnn|tensorrt|nvinfer|vpi|visionworks'
```

返回码：0

输出：

```text
ii  nvidia-l4t-cuda                               32.6.1-20210916210945                      arm64        NVIDIA CUDA Package
```

思考：只安装了 `nvidia-l4t-cuda`，它提供驱动侧 CUDA 库，不等于完整 CUDA Toolkit。

#### 命令 10

```bash
find /usr -maxdepth 3 -name nvcc -type f 2>/dev/null | sort
```

返回码：0

输出：

```text

```

#### 命令 11

```bash
find /usr -maxdepth 4 -name 'libcudart.so*' 2>/dev/null | sort
```

返回码：0

输出：

```text

```

思考：系统没有 `nvcc`，也没有 CUDA Runtime `libcudart`。

### 3. 检查 nvidia-l4t-cuda 内容和 GPU 状态工具

#### 命令 12

```bash
dpkg -L nvidia-l4t-cuda
```

返回码：0

输出：

```text
/.
/usr
/usr/share
/usr/share/doc
/usr/share/doc/nvidia-l4t-cuda
/usr/share/doc/nvidia-l4t-cuda/copyright
/usr/share/doc/nvidia-l4t-cuda/changelog.Debian.gz
/usr/lib
/usr/lib/aarch64-linux-gnu
/usr/lib/aarch64-linux-gnu/tegra
/usr/lib/aarch64-linux-gnu/tegra/libcuda.so.1.1
/usr/lib/aarch64-linux-gnu/libcuda.so
/usr/lib/aarch64-linux-gnu/tegra/libcuda.so
```

思考：该包只提供 `libcuda.so`，即驱动 API，不提供开发工具链。

#### 命令 13

```bash
ls -ld /usr/local/cuda*
```

返回码：2

输出：

```text
ls: 无法访问'/usr/local/cuda*': 没有那个文件或目录
```

#### 命令 14

```bash
ls -l /usr/lib/aarch64-linux-gnu/tegra | head -n 50
```

返回码：0

输出摘录：

```text
总用量 141288
-rw-r--r-- 1 root root       33 7月  22  2021 ld.so.conf
lrwxrwxrwx 1 root root       14 9月  17  2021 libcuda.so -> libcuda.so.1.1
lrwxrwxrwx 1 root root       14 7月  22  2021 libcuda.so.1 -> libcuda.so.1.1
-rw-r--r-- 1 root root 15870624 9月  17  2021 libcuda.so.1.1
...
```

#### 命令 15

```bash
command -v tegrastats
```

返回码：0

输出：

```text
/usr/bin/tegrastats
```

#### 命令 16

```bash
command -v nvpmodel
```

返回码：0

输出：

```text
/usr/sbin/nvpmodel
```

#### 命令 17

```bash
command -v jetson_release
```

返回码：1

输出：

```text

```

### 4. 检查 GPU 运行状态、ldconfig 和 apt 源

#### 命令 18

```bash
timeout 3s tegrastats
```

返回码：124

输出：

```text
RAM 3149/31920MB (lfb 5889x4MB) SWAP 0/15960MB (cached 0MB) CPU [57%@2188,56%@2188,57%@2188,63%@2188,off,off,off,off] EMC_FREQ 0% GR3D_FREQ 39% AO@35C GPU@35C Tdiode@37.75C PMIC@50C AUX@34.5C CPU@37.5C thermal@35.55C Tboard@35C
RAM 3153/31920MB (lfb 5888x4MB) SWAP 0/15960MB (cached 0MB) CPU [49%@2188,71%@2188,48%@2188,50%@2188,off,off,off,off] EMC_FREQ 0% GR3D_FREQ 42% AO@34.5C GPU@35C Tdiode@37.75C PMIC@50C AUX@34.5C CPU@37.5C thermal@35.55C Tboard@35C
```

说明：返回 124 是 `timeout` 到时终止。`GR3D_FREQ` 有值，说明 GPU 图形/计算域存在活动。

#### 命令 19

```bash
nvpmodel -q
```

返回码：0

输出：

```text
NV Fan Mode:quiet
NV Power Mode: MODE_15W_DESKTOP
7
NVPM ERROR: Error opening /sys/kernel/nvpmodel_emc_cap/emc_iso_cap: 13
NVPM ERROR: failed to read PARAM EMC: ARG MAX_FREQ: PATH /sys/kernel/nvpmodel_emc_cap/emc_iso_cap
NVPM ERROR: Error opening /sys/kernel/nvpmodel_emc_cap/nafll_dla: 13
NVPM ERROR: failed to read PARAM DLA_CORE: ARG MAX_FREQ: PATH /sys/kernel/nvpmodel_emc_cap/nafll_dla
NVPM ERROR: Error opening /sys/kernel/nvpmodel_emc_cap/nafll_dla_falcon: 13
NVPM ERROR: failed to read PARAM DLA_FALCON: ARG MAX_FREQ: PATH /sys/kernel/nvpmodel_emc_cap/nafll_dla_falcon
NVPM ERROR: Error opening /sys/kernel/nvpmodel_emc_cap/nafll_pva_vps: 13
NVPM ERROR: failed to read PARAM PVA_VPS: ARG MAX_FREQ: PATH /sys/kernel/nvpmodel_emc_cap/nafll_pva_vps
NVPM ERROR: Error opening /sys/kernel/nvpmodel_emc_cap/nafll_pva_core: 13
NVPM ERROR: failed to read PARAM PVA_CORE: ARG MAX_FREQ: PATH /sys/kernel/nvpmodel_emc_cap/nafll_pva_core
NVPM ERROR: Error opening /sys/kernel/nvpmodel_emc_cap/nafll_cvnas: 13
NVPM ERROR: failed to read PARAM CVNAS: ARG MAX_FREQ: PATH /sys/kernel/nvpmodel_emc_cap/nafll_cvnas
NVPM ERROR: Error opening /sys/bus/platform/devices/pwm-fan/fan_profile: 13
NVPM ERROR: Fan failed to set mode:quiet
```

思考：能查询到电源模式，但部分 sysfs 权限不足。

#### 命令 20

```bash
ldconfig -p | grep -E 'libcuda|libcudart|libcudnn|libnvinfer'
```

返回码：0

输出：

```text
	libcuda.so.1 (libc6,AArch64) => /usr/lib/aarch64-linux-gnu/tegra/libcuda.so.1
	libcuda.so (libc6,AArch64) => /usr/lib/aarch64-linux-gnu/libcuda.so
	libcuda.so (libc6,AArch64) => /usr/lib/aarch64-linux-gnu/tegra/libcuda.so
```

思考：只有 CUDA Driver API，没有 cudart、cuDNN、TensorRT。

#### 命令 21

```bash
apt-cache policy nvidia-jetpack cuda-toolkit-10-2 cuda-cudart-10-2 libcudnn8 libnvinfer8
```

返回码：0

输出：

```text
nvidia-jetpack:
  已安装：(无)
  候选： 4.6-b199
  版本列表：
     4.6-b199 500
        500 https://repo.download.nvidia.com/jetson/t194 r32.6/main arm64 Packages
     4.6-b197 500
        500 https://repo.download.nvidia.com/jetson/t194 r32.6/main arm64 Packages
cuda-toolkit-10-2:
  已安装：(无)
  候选： 10.2.460-1
  版本列表：
     10.2.460-1 500
        500 https://repo.download.nvidia.com/jetson/common r32.6/main arm64 Packages
cuda-cudart-10-2:
  已安装：(无)
  候选： 10.2.300-1
  版本列表：
     10.2.300-1 500
        500 https://repo.download.nvidia.com/jetson/common r32.6/main arm64 Packages
libcudnn8:
  已安装：(无)
  候选： 8.2.1.32-1+cuda10.2
  版本列表：
     8.2.1.32-1+cuda10.2 500
        500 https://repo.download.nvidia.com/jetson/common r32.6/main arm64 Packages
libnvinfer8:
  已安装：(无)
  候选： 8.0.1-1+cuda10.2
  版本列表：
     8.0.1-1+cuda10.2 500
        500 https://repo.download.nvidia.com/jetson/common r32.6/main arm64 Packages
```

思考：apt 源中有 JetPack 4.6 / CUDA 10.2 包，但系统尚未安装。

### 5. 尝试标准安装 JetPack，失败

#### 命令 22

```bash
sudo apt-get install -y nvidia-jetpack
```

返回码：1

输出：

```text
sudo: 有效用户 ID 不是 0，/usr/bin/sudo 位于一个设置了“nosuid”选项的文件系统或没有 root 权限的 NFS 文件系统中吗？
```

问题：不能使用 root 权限安装系统包。

思考：这与前面任务中安装 OpenCV 时遇到的限制一致。为继续完成任务，改为下载 `.deb` 并解包到用户目录。

### 6. 分析 CUDA Toolkit 最小依赖

#### 命令 23

```bash
apt-cache depends cuda-toolkit-10-2
```

返回码：0

输出：

```text
cuda-toolkit-10-2
  依赖: cuda-compiler-10-2
  依赖: cuda-libraries-10-2
  依赖: cuda-libraries-dev-10-2
  依赖: cuda-tools-10-2
  依赖: cuda-documentation-10-2
  依赖: cuda-nvml-dev-10-2
  依赖: cuda-samples-10-2
```

#### 命令 24

```bash
apt-cache policy cuda-nvcc-10-2 cuda-cudart-dev-10-2 cuda-cublas-dev-10-2 cuda-curand-dev-10-2 cuda-cudart-10-2 cuda-cublas-10-2 cuda-curand-10-2
```

返回码：0

输出：

```text
cuda-nvcc-10-2:
  已安装：(无)
  候选： 10.2.300-1
  版本列表：
     10.2.300-1 500
        500 https://repo.download.nvidia.com/jetson/common r32.6/main arm64 Packages
cuda-cudart-dev-10-2:
  已安装：(无)
  候选： 10.2.300-1
  版本列表：
     10.2.300-1 500
        500 https://repo.download.nvidia.com/jetson/common r32.6/main arm64 Packages
cuda-curand-dev-10-2:
  已安装：(无)
  候选： (无)
  版本列表：
cuda-cudart-10-2:
  已安装：(无)
  候选： 10.2.300-1
  版本列表：
     10.2.300-1 500
        500 https://repo.download.nvidia.com/jetson/common r32.6/main arm64 Packages
cuda-curand-10-2:
  已安装：(无)
  候选： (无)
  版本列表：
```

说明：部分包名不是 `cuda-curand-*`，实际是 `libcurand-*`。

#### 命令 25

```bash
apt-cache depends cuda-nvcc-10-2 cuda-cudart-dev-10-2 cuda-cublas-dev-10-2 cuda-curand-dev-10-2
```

返回码：0

输出：

```text
cuda-nvcc-10-2
  依赖: cuda-cudart-dev-10-2
  依赖: build-essential
  破坏: cuda-cudart-10-2
  破坏: <cuda-misc-headers-10-2>
  替换: cuda-cudart-10-2
    cuda-nvcc-10-2
  替换: <cuda-misc-headers-10-2>
cuda-cudart-dev-10-2
  依赖: cuda-cudart-10-2
  依赖: cuda-driver-dev-10-2
  破坏: <cuda-misc-headers-10-2>
  替换: <cuda-misc-headers-10-2>
<cuda-curand-dev-10-2>
```

#### 命令 26

```bash
apt-cache search cublas | head -n 50
```

返回码：0

输出：

```text
python3-gimmik - generator of matrix multiplication kernels (Python 3)
libcublas-dev - CUBLAS native dev links, headers
libcublas10 - CUBLAS native runtime libraries
```

#### 命令 27

```bash
apt-cache search curand | head -n 50
```

返回码：0

输出：

```text
libcurand-10-2 - CURAND native runtime libraries
libcurand-dev-10-2 - CURAND native dev links, headers
```

#### 命令 28

```bash
apt-cache depends cuda-libraries-dev-10-2
```

返回码：0

输出：

```text
cuda-libraries-dev-10-2
  依赖: cuda-cudart-dev-10-2
  依赖: cuda-driver-dev-10-2
  依赖: cuda-nvrtc-dev-10-2
  依赖: libcublas-dev
  依赖: libcufft-dev-10-2
  依赖: libcurand-dev-10-2
  依赖: libcusolver-dev-10-2
  依赖: libcusparse-dev-10-2
  依赖: libnpp-dev-10-2
  依赖: cuda-nvgraph-dev-10-2
```

#### 命令 29

```bash
apt-cache depends cuda-libraries-10-2
```

返回码：0

输出：

```text
cuda-libraries-10-2
  依赖: cuda-cudart-10-2
  依赖: cuda-nvrtc-10-2
  依赖: libcublas10
  依赖: libcufft-10-2
  依赖: libcurand-10-2
  依赖: libcusolver-10-2
  依赖: libcusparse-10-2
  依赖: libnpp-10-2
  依赖: cuda-nvgraph-10-2
```

思考：Darknet GPU 主要需要 `nvcc`、cudart、cublas、curand，不需要完整 CUDA Toolkit 全量包。

#### 命令 30

```bash
apt-cache show cuda-nvcc-10-2 cuda-cudart-dev-10-2 cuda-cudart-10-2 cuda-driver-dev-10-2 libcublas-dev libcublas10 libcurand-dev-10-2 libcurand-10-2 | grep -E '^(Package|Version|Installed-Size|Size):'
```

返回码：0

输出：

```text
Package: cuda-nvcc-10-2
Version: 10.2.300-1
Installed-Size: 54006
Size: 15301944
Package: cuda-cudart-dev-10-2
Version: 10.2.300-1
Installed-Size: 15632
Size: 1543164
Package: cuda-cudart-10-2
Version: 10.2.300-1
Installed-Size: 558
Size: 125174
Package: cuda-driver-dev-10-2
Version: 10.2.300-1
Installed-Size: 122
Size: 23200
Package: libcublas-dev
Version: 10.2.3.300-1
Installed-Size: 130285
Size: 46059272
Package: libcublas10
Version: 10.2.3.300-1
Installed-Size: 112577
Size: 45952946
Package: libcurand-dev-10-2
Version: 10.1.2.300-1
Installed-Size: 62406
Size: 39095976
Package: libcurand-10-2
Version: 10.1.2.300-1
Installed-Size: 61304
Size: 39015260
```

#### 命令 31

```bash
apt-cache policy cuda-driver-dev-10-2 libcublas-dev libcublas10 libcurand-dev-10-2 libcurand-10-2
```

返回码：0

输出：

```text
cuda-driver-dev-10-2:
  已安装：(无)
  候选： 10.2.300-1
  版本列表：
     10.2.300-1 500
        500 https://repo.download.nvidia.com/jetson/common r32.6/main arm64 Packages
libcublas-dev:
  已安装：(无)
  候选： 10.2.3.300-1
  版本列表：
     10.2.3.300-1 500
        500 https://repo.download.nvidia.com/jetson/common r32.6/main arm64 Packages
libcublas10:
  已安装：(无)
  候选： 10.2.3.300-1
  版本列表：
     10.2.3.300-1 500
        500 https://repo.download.nvidia.com/jetson/common r32.6/main arm64 Packages
libcurand-dev-10-2:
  已安装：(无)
  候选： 10.1.2.300-1
  版本列表：
     10.1.2.300-1 500
        500 https://repo.download.nvidia.com/jetson/common r32.6/main arm64 Packages
libcurand-10-2:
  已安装：(无)
  候选： 10.1.2.300-1
  版本列表：
     10.1.2.300-1 500
        500 https://repo.download.nvidia.com/jetson/common r32.6/main arm64 Packages
```

### 7. 下载并解包最小 CUDA 10.2 工具链

#### 命令 32

```bash
mkdir -p cuda_debs cuda_local
```

返回码：0

输出：

```text

```

#### 命令 33

```bash
apt-get download cuda-nvcc-10-2 cuda-cudart-dev-10-2 cuda-cudart-10-2 cuda-driver-dev-10-2 libcublas-dev libcublas10 libcurand-dev-10-2 libcurand-10-2
```

工作目录：`/home/gwb/Fang/cuda_debs`

返回码：0

输出：

```text
获取:1 https://repo.download.nvidia.cn/jetson/common r32.6/main arm64 cuda-cudart-10-2 arm64 10.2.300-1 [125 kB]
获取:2 https://repo.download.nvidia.cn/jetson/common r32.6/main arm64 cuda-cudart-dev-10-2 arm64 10.2.300-1 [1,543 kB]
获取:3 https://repo.download.nvidia.cn/jetson/common r32.6/main arm64 cuda-driver-dev-10-2 arm64 10.2.300-1 [23.2 kB]
获取:4 https://repo.download.nvidia.cn/jetson/common r32.6/main arm64 cuda-nvcc-10-2 arm64 10.2.300-1 [15.3 MB]
获取:5 https://repo.download.nvidia.cn/jetson/common r32.6/main arm64 libcublas-dev arm64 10.2.3.300-1 [46.1 MB]
获取:6 https://repo.download.nvidia.cn/jetson/common r32.6/main arm64 libcublas10 arm64 10.2.3.300-1 [46.0 MB]
获取:7 https://repo.download.nvidia.cn/jetson/common r32.6/main arm64 libcurand-10-2 arm64 10.1.2.300-1 [39.0 MB]
获取:8 https://repo.download.nvidia.cn/jetson/common r32.6/main arm64 libcurand-dev-10-2 arm64 10.1.2.300-1 [39.1 MB]
已下载 187 MB，耗时 1分 23秒 (2,257 kB/s)
```

#### 命令 34

```bash
for f in *.deb; do dpkg-deb -x "$f" ../cuda_local; done
```

工作目录：`/home/gwb/Fang/cuda_debs`

返回码：0

输出：

```text

```

#### 命令 35

```bash
find cuda_local -maxdepth 4 -name nvcc -type f -o -name 'libcudart.so*' -o -name 'libcublas.so*' -o -name 'libcurand.so*' | sort
```

返回码：0

输出：

```text

```

说明：该 `find` 命令因为表达式优先级和 `maxdepth` 设置不合适没有列出结果，后续用更明确路径重新查。

#### 命令 36

```bash
cuda_local/usr/local/cuda-10.2/bin/nvcc --version
```

返回码：0

输出：

```text
nvcc: NVIDIA (R) Cuda compiler driver
Copyright (c) 2005-2021 NVIDIA Corporation
Built on Sun_Feb_28_22:34:44_PST_2021
Cuda compilation tools, release 10.2, V10.2.300
Build cuda_10.2_r440.TC440_70.29663091_0
```

#### 命令 37

```bash
ls -l cuda_local/usr/local/cuda-10.2
```

返回码：0

输出：

```text
总用量 16
drwxr-xr-x 3 gwb gwb 4096 3月   2  2021 bin
lrwxrwxrwx 1 gwb gwb   29 3月   2  2021 include -> targets/aarch64-linux/include
lrwxrwxrwx 1 gwb gwb   25 3月   2  2021 lib64 -> targets/aarch64-linux/lib
drwxr-xr-x 7 gwb gwb 4096 3月   2  2021 nvvm
drwxr-xr-x 7 gwb gwb 4096 3月   2  2021 nvvmx
drwxr-xr-x 3 gwb gwb 4096 3月   2  2021 targets
```

#### 命令 38

```bash
find cuda_local/usr/local/cuda-10.2 -name 'libcudart.so*' -o -name 'libcublas.so*' -o -name 'libcurand.so*' | sort
```

返回码：0

输出：

```text
cuda_local/usr/local/cuda-10.2/targets/aarch64-linux/lib/libcublas.so
cuda_local/usr/local/cuda-10.2/targets/aarch64-linux/lib/libcublas.so.10
cuda_local/usr/local/cuda-10.2/targets/aarch64-linux/lib/libcublas.so.10.2.3.300
cuda_local/usr/local/cuda-10.2/targets/aarch64-linux/lib/libcudart.so
cuda_local/usr/local/cuda-10.2/targets/aarch64-linux/lib/libcudart.so.10.2
cuda_local/usr/local/cuda-10.2/targets/aarch64-linux/lib/libcudart.so.10.2.300
cuda_local/usr/local/cuda-10.2/targets/aarch64-linux/lib/libcurand.so
cuda_local/usr/local/cuda-10.2/targets/aarch64-linux/lib/libcurand.so.10
cuda_local/usr/local/cuda-10.2/targets/aarch64-linux/lib/libcurand.so.10.1.2.300
cuda_local/usr/local/cuda-10.2/targets/aarch64-linux/lib/stubs/libcublas.so
cuda_local/usr/local/cuda-10.2/targets/aarch64-linux/lib/stubs/libcurand.so
```

#### 命令 39

```bash
find cuda_local/usr/local/cuda-10.2 -name cuda_runtime.h -o -name cublas_v2.h -o -name curand.h | sort
```

返回码：0

输出：

```text
cuda_local/usr/local/cuda-10.2/targets/aarch64-linux/include/cublas_v2.h
cuda_local/usr/local/cuda-10.2/targets/aarch64-linux/include/cuda_runtime.h
cuda_local/usr/local/cuda-10.2/targets/aarch64-linux/include/curand.h
```

#### 命令 40

```bash
ldd cuda_local/usr/local/cuda-10.2/bin/nvcc | head -n 30
```

返回码：0

输出：

```text
	linux-vdso.so.1 (0x0000007fab895000)
	libgtk3-nocsd.so.0 => /usr/lib/aarch64-linux-gnu/libgtk3-nocsd.so.0 (0x0000007fab831000)
	libpthread.so.0 => /lib/aarch64-linux-gnu/libpthread.so.0 (0x0000007fab805000)
	libc.so.6 => /lib/aarch64-linux-gnu/libc.so.6 (0x0000007fab6ac000)
	libdl.so.2 => /lib/aarch64-linux-gnu/libdl.so.2 (0x0000007fab697000)
	/lib/ld-linux-aarch64.so.1 (0x0000007fab869000)
```

### 8. 编译 GPU 版 Darknet

思考：当前系统为 t194 / CUDA 10.2，采用 `sm_72`。如果真是 Orin，CUDA 10.2 不支持 Orin 的 `sm_87`，这也说明当前系统环境不匹配 AGX Orin。

#### 命令 41

```bash
mkdir -p third_party/darknet-gpu && tar -xzf third_party/darknet-source.tar.gz --strip-components=1 -C third_party/darknet-gpu
```

返回码：0

输出：

```text

```

#### 命令 42

```bash
make -j2 GPU=1 CUDNN=0 OPENCV=0 NVCC=/home/gwb/Fang/cuda_local/usr/local/cuda-10.2/bin/nvcc ARCH='-gencode arch=compute_72,code=sm_72' COMMON='-Iinclude/ -Isrc/ -DGPU -I/home/gwb/Fang/cuda_local/usr/local/cuda-10.2/include' CFLAGS='-Wall -Wno-unused-result -Wno-unknown-pragmas -Wfatal-errors -fPIC -Ofast -DGPU' LDFLAGS='-lm -pthread -L/home/gwb/Fang/cuda_local/usr/local/cuda-10.2/lib64 -L/usr/lib/aarch64-linux-gnu/tegra -lcuda -lcudart -lcublas -lcurand'
```

工作目录：`/home/gwb/Fang/third_party/darknet-gpu`

返回码：2

输出摘录：

```text
mkdir -p obj
mkdir -p backup
mkdir -p results
gcc -Iinclude/ -Isrc/ -DGPU -I/home/gwb/Fang/cuda_local/usr/local/cuda-10.2/include ... -c ./src/gemm.c -o obj/gemm.o
...
/home/gwb/Fang/cuda_local/usr/local/cuda-10.2/bin/nvcc -gencode arch=compute_72,code=sm_72 ... -c ./src/convolutional_kernels.cu -o obj/convolutional_kernels.o
...
gcc ... -shared ... -o libdarknet.so -lm -pthread -L/home/gwb/Fang/cuda_local/usr/local/cuda-10.2/lib64 -L/usr/lib/aarch64-linux-gnu/tegra -lcuda -lcudart -lcublas -lcurand
...
/usr/bin/ld: warning: libcublasLt.so.10, needed by /home/gwb/Fang/cuda_local/usr/local/cuda-10.2/lib64/libcublas.so, not found (try using -rpath or -rpath-link)
/home/gwb/Fang/cuda_local/usr/local/cuda-10.2/lib64/libcublas.so：对‘cublasLtShutdownCtx@libcublasLt.so.10’未定义的引用
...
collect2: error: ld returned 1 exit status
make: *** [darknet] Error 1
```

问题：`libdarknet.so` 已生成，但最终链接 `darknet` 可执行文件时缺少 `libcublasLt.so.10`。

原因分析：CUDA 10.2 的 `libcublas.so` 依赖 `libcublasLt.so.10`，链接命令只写了 `-lcublas`，没有显式加 `-lcublasLt`。

#### 命令 43

```bash
apt-cache search cublaslt
```

返回码：0

输出：

```text

```

#### 命令 44

```bash
find cuda_local/usr/local/cuda-10.2 -name 'libcublasLt.so*' | sort
```

返回码：0

输出：

```text
cuda_local/usr/local/cuda-10.2/targets/aarch64-linux/lib/libcublasLt.so
cuda_local/usr/local/cuda-10.2/targets/aarch64-linux/lib/libcublasLt.so.10
cuda_local/usr/local/cuda-10.2/targets/aarch64-linux/lib/libcublasLt.so.10.2.3.300
cuda_local/usr/local/cuda-10.2/targets/aarch64-linux/lib/stubs/libcublasLt.so
```

#### 命令 45

```bash
ls -l third_party/darknet-gpu/libdarknet.so third_party/darknet-gpu/darknet
```

返回码：2

输出：

```text
-rwxrwxr-x 1 gwb gwb 998800 6月   1 11:43 third_party/darknet-gpu/libdarknet.so
ls: 无法访问'third_party/darknet-gpu/darknet': 没有那个文件或目录
```

思考：动态库已生成，二进制未生成；只需要补 `-lcublasLt` 重新链接。

#### 命令 46

```bash
make -j2 GPU=1 CUDNN=0 OPENCV=0 NVCC=/home/gwb/Fang/cuda_local/usr/local/cuda-10.2/bin/nvcc ARCH='-gencode arch=compute_72,code=sm_72' COMMON='-Iinclude/ -Isrc/ -DGPU -I/home/gwb/Fang/cuda_local/usr/local/cuda-10.2/include' CFLAGS='-Wall -Wno-unused-result -Wno-unknown-pragmas -Wfatal-errors -fPIC -Ofast -DGPU' LDFLAGS='-lm -pthread -L/home/gwb/Fang/cuda_local/usr/local/cuda-10.2/lib64 -L/usr/lib/aarch64-linux-gnu/tegra -lcuda -lcudart -lcublas -lcublasLt -lcurand'
```

工作目录：`/home/gwb/Fang/third_party/darknet-gpu`

返回码：0

输出：

```text
gcc -Iinclude/ -Isrc/ -DGPU -I/home/gwb/Fang/cuda_local/usr/local/cuda-10.2/include -Wall -Wno-unused-result -Wno-unknown-pragmas -Wfatal-errors -fPIC -Ofast -DGPU obj/captcha.o obj/lsd.o obj/super.o obj/art.o obj/tag.o obj/cifar.o obj/go.o obj/rnn.o obj/segmenter.o obj/regressor.o obj/classifier.o obj/coco.o obj/yolo.o obj/detector.o obj/nightmare.o obj/instance-segmenter.o obj/darknet.o libdarknet.a -o darknet -lm -pthread -L/home/gwb/Fang/cuda_local/usr/local/cuda-10.2/lib64 -L/usr/lib/aarch64-linux-gnu/tegra -lcuda -lcudart -lcublas -lcublasLt -lcurand libdarknet.a
```

### 9. 验证 GPU 版 Darknet 动态库依赖

#### 命令 47

```bash
ldd third_party/darknet-gpu/libdarknet.so | grep -E 'cuda|cublas|curand|not found'
```

返回码：0

输出：

```text
	libcudart.so.10.2 => not found
	libcublas.so.10 => not found
	libcurand.so.10 => not found
```

思考：不设置 `LD_LIBRARY_PATH` 时，动态库找不到用户目录 CUDA 运行库。

#### 命令 48

```bash
ldd third_party/darknet-gpu/darknet | grep -E 'cuda|cublas|curand|not found'
```

返回码：0

输出：

```text
	libcudart.so.10.2 => not found
	libcublas.so.10 => not found
	libcurand.so.10 => not found
```

#### 命令 49

```bash
ls -lh third_party/darknet-gpu/libdarknet.so third_party/darknet-gpu/darknet
```

返回码：0

输出：

```text
-rwxrwxr-x 1 gwb gwb 1.1M 6月   1 11:44 third_party/darknet-gpu/darknet
-rwxrwxr-x 1 gwb gwb 976K 6月   1 11:43 third_party/darknet-gpu/libdarknet.so
```

#### 命令 50

```bash
LD_LIBRARY_PATH=/home/gwb/Fang/cuda_local/usr/local/cuda-10.2/lib64:/usr/lib/aarch64-linux-gnu/tegra ldd third_party/darknet-gpu/libdarknet.so | grep -E 'cuda|cublas|curand|not found'
```

返回码：0

输出：

```text
	libcudart.so.10.2 => /home/gwb/Fang/cuda_local/usr/local/cuda-10.2/lib64/libcudart.so.10.2 (0x0000007fa6745000)
	libcublas.so.10 => /home/gwb/Fang/cuda_local/usr/local/cuda-10.2/lib64/libcublas.so.10 (0x0000007fa19dd000)
	libcurand.so.10 => /home/gwb/Fang/cuda_local/usr/local/cuda-10.2/lib64/libcurand.so.10 (0x0000007f9d8ac000)
	libcublasLt.so.10 => /home/gwb/Fang/cuda_local/usr/local/cuda-10.2/lib64/libcublasLt.so.10 (0x0000007f9b455000)
```

#### 命令 51

```bash
LD_LIBRARY_PATH=/home/gwb/Fang/cuda_local/usr/local/cuda-10.2/lib64:/usr/lib/aarch64-linux-gnu/tegra ldd third_party/darknet-gpu/darknet | grep -E 'cuda|cublas|curand|not found'
```

返回码：0

输出：

```text
	libcudart.so.10.2 => /home/gwb/Fang/cuda_local/usr/local/cuda-10.2/lib64/libcudart.so.10.2 (0x0000007f8f877000)
	libcublas.so.10 => /home/gwb/Fang/cuda_local/usr/local/cuda-10.2/lib64/libcublas.so.10 (0x0000007f8ab0f000)
	libcurand.so.10 => /home/gwb/Fang/cuda_local/usr/local/cuda-10.2/lib64/libcurand.so.10 (0x0000007f869de000)
	libcublasLt.so.10 => /home/gwb/Fang/cuda_local/usr/local/cuda-10.2/lib64/libcublasLt.so.10 (0x0000007f84587000)
```

### 10. 验证 GPU 版 Darknet 推理

#### 命令 52

```bash
LD_LIBRARY_PATH=/home/gwb/Fang/cuda_local/usr/local/cuda-10.2/lib64:/usr/lib/aarch64-linux-gnu/tegra ./darknet detect cfg/yolov3-tiny.cfg ../../models/yolov3-tiny/yolov3-tiny.weights data/dog.jpg -dont_show
```

工作目录：`/home/gwb/Fang/third_party/darknet-gpu`

返回码：0

输出：

```text
data/dog.jpg: Predicted in 1.667395 seconds.
dog: 57%
car: 52%
truck: 56%
car: 62%
bicycle: 59%
layer     filters    size              input                output
    0 conv     16  3 x 3 / 1   416 x 416 x   3   ->   416 x 416 x  16  0.150 BFLOPs
...
   23 yolo
Loading weights from ../../models/yolov3-tiny/yolov3-tiny.weights...Done!
```

思考：检测结果与 CPU 版一致，GPU 版 Darknet 可以运行。

### 11. 编写 Python CUDA YOLO 脚本

操作：新增 `/home/gwb/Fang/yolo_cuda_realtime.py`。

设计：

- 自动设置本地 CUDA 运行库路径：

```text
/home/gwb/Fang/cuda_local/usr/local/cuda-10.2/lib64
/usr/lib/aarch64-linux-gnu/tegra
```

- 默认加载 GPU 版 Darknet：

```text
/home/gwb/Fang/third_party/darknet-gpu/libdarknet.so
```

- 复用已有 `yolo_realtime.py` 的摄像头读取、窗口显示和 ctypes YOLO 推理逻辑。

#### 命令 53

```bash
chmod +x yolo_cuda_realtime.py
```

返回码：0

输出：

```text

```

#### 命令 54

```bash
./yolo_cuda_realtime.py --image third_party/darknet-gpu/data/dog.jpg --test-only --save-frame yolo_cuda_dog_result.jpg
```

返回码：0

输出：

```text
Loaded YOLOv3-tiny 416x416 with 80 classes.
Processed image third_party/darknet-gpu/data/dog.jpg; detections=6.
layer     filters    size              input                output
    0 conv     16  3 x 3 / 1   416 x 416 x   3   ->   416 x 416 x  16  0.150 BFLOPs
...
   23 yolo
Loading weights from /home/gwb/Fang/models/yolov3-tiny/yolov3-tiny.weights...Done!
```

#### 命令 55

```bash
file yolo_cuda_dog_result.jpg
```

返回码：0

输出：

```text
yolo_cuda_dog_result.jpg: JPEG image data, JFIF standard 1.01, aspect ratio, density 1x1, segment length 16, baseline, precision 8, 768x576, frames 3
```

#### 命令 56

```bash
ldd third_party/darknet-gpu/libdarknet.so | grep -E 'cuda|cublas|curand|not found'
```

返回码：0

输出：

```text
	libcudart.so.10.2 => not found
	libcublas.so.10 => not found
	libcurand.so.10 => not found
```

#### 命令 57

```bash
LD_LIBRARY_PATH=/home/gwb/Fang/cuda_local/usr/local/cuda-10.2/lib64:/usr/lib/aarch64-linux-gnu/tegra ldd third_party/darknet-gpu/libdarknet.so | grep -E 'cuda|cublas|curand|not found'
```

返回码：0

输出：

```text
	libcudart.so.10.2 => /home/gwb/Fang/cuda_local/usr/local/cuda-10.2/lib64/libcudart.so.10.2 (0x0000007f929d9000)
	libcublas.so.10 => /home/gwb/Fang/cuda_local/usr/local/cuda-10.2/lib64/libcublas.so.10 (0x0000007f8dc71000)
	libcurand.so.10 => /home/gwb/Fang/cuda_local/usr/local/cuda-10.2/lib64/libcurand.so.10 (0x0000007f89b40000)
	libcublasLt.so.10 => /home/gwb/Fang/cuda_local/usr/local/cuda-10.2/lib64/libcublasLt.so.10 (0x0000007f876e9000)
```

#### 命令 58

```bash
./yolo_cuda_realtime.py --test-only --save-frame yolo_cuda_test.jpg
```

返回码：0

输出：

```text
Loaded YOLOv3-tiny 416x416 with 80 classes.
Processed one frame from camera 0; detections=0.
layer     filters    size              input                output
    0 conv     16  3 x 3 / 1   416 x 416 x   3   ->   416 x 416 x  16  0.150 BFLOPs
...
   23 yolo
Loading weights from /home/gwb/Fang/models/yolov3-tiny/yolov3-tiny.weights...Done!
```

说明：摄像头画面中没有超过阈值的 COCO 目标，所以 `detections=0` 正常。

#### 命令 59

```bash
timeout 10s ./yolo_cuda_realtime.py --skip-frames 10 --thresh 0.25
```

返回码：124

输出：

```text
Loaded YOLOv3-tiny 416x416 with 80 classes.
Showing YOLO detections. Press q or Esc in the window to exit.
layer     filters    size              input                output
    0 conv     16  3 x 3 / 1   416 x 416 x   3   ->   416 x 416 x  16  0.150 BFLOPs
...
   23 yolo
Loading weights from /home/gwb/Fang/models/yolov3-tiny/yolov3-tiny.weights...Done!
```

说明：返回 124 是 `timeout` 到时终止。脚本已经进入实时窗口显示循环。

### 12. 检查 Python CUDA 常见包

#### 命令 60

```bash
python3 -c "import pycuda.driver as cuda; print('pycuda ok')"
```

返回码：1

输出：

```text
Traceback (most recent call last):
  File "<string>", line 1, in <module>
ModuleNotFoundError: No module named 'pycuda'
```

#### 命令 61

```bash
python3 -c "import cupy; print(cupy.__version__)"
```

返回码：1

输出：

```text
Traceback (most recent call last):
  File "<string>", line 1, in <module>
ModuleNotFoundError: No module named 'cupy'
```

#### 命令 62

```bash
python3 -c "import numba; print(numba.__version__)"
```

返回码：1

输出：

```text
Traceback (most recent call last):
  File "<string>", line 1, in <module>
ModuleNotFoundError: No module named 'numba'
```

#### 命令 63

```bash
file yolo_cuda_test.jpg
```

返回码：0

输出：

```text
yolo_cuda_test.jpg: JPEG image data, JFIF standard 1.01, aspect ratio, density 1x1, segment length 16, baseline, precision 8, 640x480, frames 3
```

思考：常见 Python CUDA 包未安装；当前可用的 Python CUDA 开发路径是通过 Python `ctypes` 调用 CUDA/GPU 动态库。

### 13. CUDA kernel smoke test

操作：新增 `/home/gwb/Fang/cuda_smoke_test.cu`。

目的：验证 `nvcc` 编译和 CUDA runtime 运行链路。

#### 命令 64

```bash
cuda_local/usr/local/cuda-10.2/bin/nvcc -arch=sm_72 cuda_smoke_test.cu -o cuda_smoke_test -L/home/gwb/Fang/cuda_local/usr/local/cuda-10.2/lib64 -L/usr/lib/aarch64-linux-gnu/tegra -lcudart -lcuda
```

返回码：0

输出：

```text

```

#### 命令 65

```bash
LD_LIBRARY_PATH=/home/gwb/Fang/cuda_local/usr/local/cuda-10.2/lib64:/usr/lib/aarch64-linux-gnu/tegra ./cuda_smoke_test
```

返回码：0

输出：

```text
CUDA result: 11.0 22.0 33.0 44.0
```

结论：本地 CUDA Toolkit 可以编译 CUDA 程序，GPU runtime 可以执行 CUDA kernel。

### 14. 桌面说明和本记录

操作：新增 `/home/gwb/Desktop/yolo_cuda_usage.md`。

操作：新增 `/home/gwb/Fang/RECORD3.md`。

## 后续建议

如果这台机器确实应当是 AGX Orin，建议重新核对硬件和系统镜像：

- AGX Orin 应使用 JetPack 5/6，对应 L4T R35/R36。
- Orin GPU 架构是 Ampere，常用 CUDA 架构为 `sm_87`。
- 当前系统是 R32.6.1 / CUDA 10.2 / t194 配置，不是 Orin 的正常软件栈。

在当前受限环境下，我已经用用户目录方案补齐了最小 CUDA 10.2 开发能力，并完成 CUDA/GPU YOLO 检测脚本。
