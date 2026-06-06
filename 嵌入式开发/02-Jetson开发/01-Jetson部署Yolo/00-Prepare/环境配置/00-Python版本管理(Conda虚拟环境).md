本文记录在当前设备上安装旧版 Miniconda，并使用 Conda 管理 Python 虚拟环境的过程。

# 一、Miniconda 安装

## 1. 当前系统情况

本机环境检查结果：

```bash
uname -s
uname -m
cat /etc/os-release
getconf GNU_LIBC_VERSION
df -h /home /tmp
```

当前设备信息：

- 系统：Ubuntu 18.04.6 LTS
- 架构：Linux aarch64 / ARM64
- 设备：Jetson-AGX
- glibc：2.27
- 可用磁盘：约 14G

由于最新版 Anaconda/Miniconda 对 Linux 的 glibc 要求更高，当前系统的 glibc 2.27 不适合安装最新版。因此选择安装旧版 Miniconda。

## 2. 下载旧版 Miniconda

当前系统是 ARM64/aarch64 架构，因此必须下载 Linux-aarch64 版本，不能下载 Linux-x86_64 版本。

推荐版本：

```text
Miniconda3-py39_23.5.2-0-Linux-aarch64.sh
```

下载命令：

```bash
cd /tmp

wget -O Miniconda3-py39_23.5.2-0-Linux-aarch64.sh https://repo.anaconda.com/miniconda/Miniconda3-py39_23.5.2-0-Linux-aarch64.sh```

## 3. 校验安装包

校验 sha256，确认安装包没有损坏：

```bash
echo "ecc06a39bdf786ebb8325a2754690a808f873154719c97d10087ef0883b69e84  Miniconda3-py39_23.5.2-0-Linux-aarch64.sh" | sha256sum -c -
```

输出下面内容，说明校验通过：

```text
Miniconda3-py39_23.5.2-0-Linux-aarch64.sh: OK
```

## 4. 安装 Miniconda

安装到当前用户目录下的 `~/miniconda3`：

```bash
bash Miniconda3-py39_23.5.2-0-Linux-aarch64.sh -b -p /home/gwb/miniconda3
```

加载 Conda：

```bash
source "/home/gwb/miniconda3/etc/profile.d/conda.sh"
```

初始化 Bash：

```bash
/home/gwb/miniconda3/bin/conda init bash
```

重新加载终端环境：

```bash
exec bash
```

## 5. 验证安装

检查 Conda 和 Python 版本：

```bash
conda --version
python --version
```

正常输出版本号，说明 Miniconda 安装成功。

# 二、Conda 常用命令

## 6. 创建 Python 虚拟环境

创建一个 Python 3.9 环境：

```bash
conda create -n py39 python=3.9
```

激活环境：

```bash
conda activate py39
```

查看当前 Python：

```bash
which python
python --version
```

退出环境：

```bash
conda deactivate
```

## 7. 常用 Conda 命令

查看所有环境：

```bash
conda env list
```

创建指定 Python 版本的环境：

```bash
conda create -n py310 python=3.10
```

删除环境：

```bash
conda remove -n py310 --all
```

在当前环境安装包：

```bash
conda install numpy pandas
```

使用 pip 安装包：

```bash
pip install 包名
```

导出环境：

```bash
conda env export > environment.yml
```

从配置文件恢复环境：

```bash
conda env create -f environment.yml
```

# 三、注意事项

当前系统是 Ubuntu 18.04，glibc 版本为 2.27。不要轻易执行以下命令：

```bash
conda update -n base conda
conda update --all
```

这些命令可能把 base 环境更新到不兼容当前系统 glibc 的版本。

建议做法：

- base 环境只用于管理 Conda，不直接做项目开发。
- 每个项目单独创建 Conda 环境。
- 安装包优先在项目环境中进行。
- Jetson 设备上的 CUDA、PyTorch、OpenCV 等依赖需要优先参考 NVIDIA/JetPack 对应版本。

# 四、卸载 Miniconda

如果需要卸载：

```bash
rm -rf "$HOME/miniconda3"
```

然后编辑 `~/.bashrc`，删除 `conda init` 添加的相关内容。

