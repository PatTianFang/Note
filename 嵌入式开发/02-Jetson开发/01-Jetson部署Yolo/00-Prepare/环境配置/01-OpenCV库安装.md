# OpenCV 库安装记录

## 1. 使用 Python 3 安装 OpenCV

由于系统默认 `python` 指向 Python 2.7，使用 `python3` 和 `python3 -m pip` 来指定使用Python 3 环境。

Pip原本的软件源下载比较慢，采用清华 PyPI 镜像源安装：

```bash
python3 -m pip install --user -i https://pypi.tuna.tsinghua.edu.cn/simple opencv-python
```

参数说明：

- `python3 -m pip`：使用 Python 3 调用 pip。
- `--user`：安装到当前用户目录，不需要管理员权限。
- `-i https://pypi.tuna.tsinghua.edu.cn/simple`：使用清华镜像源，提高下载速度。
- `opencv-python`：Python 版本的 OpenCV 包，安装后可通过 `import cv2` 使用。

## 2. pip config 不可用的情况

尝试配置默认镜像源：

```bash
python3 -m pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
```

报错：

```text
ERROR: unknown command "config"
```

说明当前 `pip` 版本较旧，不支持 `pip config` 命令。

这种情况下可以直接在安装命令中使用 `-i` 指定镜像源，不影响安装：

```bash
python3 -m pip install --user -i https://pypi.tuna.tsinghua.edu.cn/simple opencv-python
```

## 3. 下载包 hash 不匹配或一直下载源码包

再次使用清华镜像安装时，可能会遇到类似错误：

```text
THESE PACKAGES DO NOT MATCH THE HASHES FROM THE REQUIREMENTS FILE.
Expected sha256 ...
Got        ...
```

这表示下载到的安装包校验值和索引中声明的校验值不一致。可能原因包括：

- 镜像源文件损坏或尚未同步完整。
- 下载中断或本地缓存损坏。
- 网络代理或 CDN 返回了异常内容。

遇到这种情况不要强行忽略校验错误，建议换源或禁用缓存重新下载。

旧版本 `pip` 可能不支持清理缓存命令：

```bash
python3 -m pip cache purge
```

如果报错：

```text
ERROR: unknown command "cache" - maybe you meant "check"
```

说明当前 `pip` 版本较旧，可以先不用清理缓存，改用 `--no-cache-dir`：

```bash
python3 -m pip install --user --no-cache-dir opencv-python
```

如果下载的是 `.tar.gz` 文件，例如：

```text
Downloading ... opencv-python-4.12.0.88.tar.gz
```

说明当前环境没有直接使用预编译 wheel 包，而是在下载源码包。源码包体积大，下载慢，后续还可能需要本地编译 OpenCV，容易失败。

此时建议先停止当前下载：

```text
Ctrl+C
```

然后检查 Python、pip 和系统架构：

```bash
python3 --version
python3 -m pip --version
uname -m
```

再尝试保守升级旧版 `pip` 和基础构建工具：

```bash
python3 -m pip install --user --upgrade "pip<21" wheel setuptools
```

升级后，强制只安装预编译二进制包，避免下载源码包：

```bash
python3 -m pip install --user --only-binary=:all: opencv-python
```

如果提示找不到匹配版本，通常说明当前 Python 版本太旧，无法安装最新版 OpenCV。可以按 Python 版本选择较旧的 OpenCV：

Python 3.5：

```bash
python3 -m pip install --user --only-binary=:all: "opencv-python==4.2.0.32"
```

Python 3.6：

```bash
python3 -m pip install --user --only-binary=:all: "opencv-python==4.6.0.66"
```

如果只在命令行或服务器环境使用 OpenCV，不需要 `cv2.imshow()` 等图形窗口功能，也可以安装 headless 版本：

```bash
python3 -m pip install --user --only-binary=:all: opencv-python-headless
```

## 4. OpenCV 安装成功但 numpy 版本冲突

使用 `--only-binary=:all:` 后，OpenCV 可以成功安装到当前用户目录：

```text
Successfully installed opencv-python-4.6.0.66
```

但安装过程中可能仍然提示 numpy 版本冲突：

```text
opencv-python 4.6.0.66 requires numpy>=1.19.3;
but you have numpy 1.13.3 which is incompatible.
```

这说明系统自带的 numpy 版本太旧，例如 `/usr/lib/python3/dist-packages` 中的 `numpy 1.13.3`。虽然 OpenCV 已经安装成功，但后续导入 `cv2` 时可能因为 numpy 版本过低而出错。

当前环境是 Python 3.6、Linux、`aarch64` 架构时，可以安装兼容的 numpy 1.19.5：

```bash
python3 -m pip install --user --force-reinstall --no-cache-dir --only-binary=:all: "numpy==1.19.5"
```

参数说明：

- `--user`：安装到当前用户目录，避免修改系统 Python 包。
- `--force-reinstall`：即使系统里已经有旧版 numpy，也重新安装指定版本。
- `--no-cache-dir`：避免继续使用可能损坏或过期的缓存。
- `--only-binary=:all:`：只使用预编译 wheel 包，不下载源码包。
- `"numpy==1.19.5"`：Python 3.6 可用的较新 numpy 版本。

安装完成后验证 numpy 和 OpenCV：

```bash
python3 -c "import numpy; print(numpy.__version__); import cv2; print(cv2.__version__)"
```

能看到输出：

```text
1.19.5
4.6.0
```


## 5. 导入 numpy 或 cv2 时出现非法指令

安装 `numpy 1.19.5` 和 `opencv-python 4.6.0.66` 后，执行验证命令时出现：

```bash
python3 -c "import numpy; print(numpy.__version__); import cv2; print(cv2.__version__)"
```

报错：

```text
非法指令 (核心已转储)
```

先单独测试 numpy：

```bash
python3 -c "import numpy; print(numpy.__version__)"
```

单独导入 numpy 也报：

```text
非法指令 (核心已转储)
```

说明问题不在 OpenCV，而是在 numpy 使用的底层 BLAS 库。当前环境中，numpy wheel 使用 OpenBLAS，OpenBLAS 在运行时自动识别 ARM CPU 指令集时选择了当前设备不支持的指令，导致程序直接崩溃。

临时指定 OpenBLAS 使用通用 ARMv8 内核：

```bash
OPENBLAS_CORETYPE=ARMV8 python3 -c "import numpy; print(numpy.__version__)"
```

正常输出：

```text
1.19.5
```

再验证 OpenCV：

```bash
OPENBLAS_CORETYPE=ARMV8 python3 -c "import numpy; print(numpy.__version__); import cv2; print(cv2.__version__)"
```

输出：

```text
1.19.5
4.6.0
```

说明解决方法有效，是因为对CPU指令集检测存在问题，识别错误。把环境变量永久写入当前用户的 shell 配置：

```bash
echo 'export OPENBLAS_CORETYPE=ARMV8' >> ~/.bashrc
source ~/.bashrc
```

之后再次验证时就不需要在命令前手动添加 `OPENBLAS_CORETYPE=ARMV8`：

```bash
python3 -c "import numpy; print(numpy.__version__); import cv2; print(cv2.__version__)"
```

## 6. 验证安装

安装完成后运行：

```bash
python3 -c "import cv2; print(cv2.__version__)"
```

能输出 OpenCV 版本号，说明安装成功。

以后运行需要 OpenCV 的 Python 程序时，建议使用：

```bash
python3 your_script.py
```

不要直接使用 `python your_script.py`，因为当前系统里的 `python` 指向 Python 2.7。
