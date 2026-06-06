# YOLO 实时检测任务记录

记录文件：`/home/gwb/Fang/RECORD2.md`

脚本文件：`/home/gwb/Fang/yolo_realtime.py`

桌面说明：`/home/gwb/Desktop/yolo_realtime_usage.md`

## 结论

已完成 Python 版 YOLO 实时检测脚本。

当前环境中的系统 OpenCV 没有 `cv2.dnn`，不能直接用 OpenCV DNN 跑 YOLO。因此采取的方案是：

1. 使用已有本地 OpenCV 3.2.0 做摄像头读取、窗口显示和画框。
2. 编译 CPU 版 Darknet，生成 `libdarknet.so`。
3. Python 通过 `ctypes` 调用 Darknet 的 YOLOv3-tiny 推理接口。
4. 使用 YOLOv3-tiny + COCO 80 类模型做实时检测。

验证结果：

- 摄像头单帧 YOLO 脚本测试成功：`./yolo_realtime.py --test-only --save-frame yolo_test.jpg`
- 静态图片 YOLO 检测成功：`dog.jpg` 检出 6 个目标，并保存 `yolo_dog_result.jpg`
- 实时窗口启动成功：`timeout 10s ./yolo_realtime.py --skip-frames 10 --thresh 0.25`
- `timeout` 返回码 `124` 是到时自动结束，不是脚本错误。

## 文件清单

- `/home/gwb/Fang/yolo_realtime.py`：Python YOLO 实时检测脚本。
- `/home/gwb/Fang/models/yolov3-tiny/yolov3-tiny.cfg`：YOLOv3-tiny 配置。
- `/home/gwb/Fang/models/yolov3-tiny/yolov3-tiny.weights`：YOLOv3-tiny 权重。
- `/home/gwb/Fang/models/yolov3-tiny/coco.names`：COCO 类别名。
- `/home/gwb/Fang/third_party/darknet-master/libdarknet.so`：Darknet 动态库。
- `/home/gwb/Desktop/yolo_realtime_usage.md`：桌面使用说明。
- `/home/gwb/Fang/RECORD2.md`：本记录。

## 详细过程

### 1. 检查当前目录和已有摄像头环境

目的：确认在原项目目录继续工作，并验证前一个摄像头脚本仍可用。

#### 命令 1

```bash
pwd
```

返回码：0

输出：

```text
/home/gwb/Fang
```

思考：当前目录是 `/home/gwb/Fang`，新脚本和记录文件放在这里。

#### 命令 2

```bash
ls -la
```

返回码：0

输出：

```text
总用量 72
drwxrwxr-x  5 gwb gwb  4096 6月   1 11:15 .
drwxr-xr-x 26 gwb gwb  4096 6月   1 10:51 ..
-rwxrwxr-x  1 gwb gwb  4401 6月   1 11:03 camera_viewer.py
drwxrwxr-x  2 gwb gwb  4096 6月   1 10:58 local_debs
drwxr-xr-x  4 gwb gwb  4096 9月  20  2018 opencv_local
-rw-rw-r--  1 gwb gwb 44137 6月   1 11:15 RECORD.md
drwxrwxr-x  2 gwb gwb  4096 6月   1 10:16 ToDo
```

思考：已有 `camera_viewer.py` 和 `opencv_local`，可复用 OpenCV 的摄像头和显示能力。

#### 命令 3

```bash
./camera_viewer.py --test-only --save-frame opencv_test_for_yolo.jpg
```

返回码：0

输出：

```text
OpenCV 3.2.0 opened camera 0 and read frame 640x480.
```

思考：摄像头和本地 OpenCV 仍然可以正常读取一帧。

#### 命令 4

```bash
./camera_viewer.py --test-only
```

返回码：1

输出：

```text
VIDEOIO ERROR: V4L2: Pixel format of incoming image is unsupported by OpenCV
Unable to stop the stream: Device or resource busy
VIDEOIO ERROR: V4L2: Pixel format of incoming image is unsupported by OpenCV
Unable to stop the stream: Device or resource busy
Traceback (most recent call last):
  File "./camera_viewer.py", line 131, in <module>
    main()
  File "./camera_viewer.py", line 94, in main
    cap = open_camera(device, args.width, args.height)
  File "./camera_viewer.py", line 61, in open_camera
    raise RuntimeError("Cannot open camera device: {}".format(device))
RuntimeError: Cannot open camera device: 0
```

问题：该命令与上一条摄像头命令并行执行，导致 `/dev/video0` 被占用。

思考：摄像头通常只能被一个进程独占打开。后续摄像头相关验证必须串行执行，避免设备忙。

### 2. 检查 Python、摄像头串行读取、OpenCV DNN 能力和下载工具

#### 命令 5

```bash
python3 --version
```

返回码：0

输出：

```text
Python 3.6.9
```

#### 命令 6

```bash
./camera_viewer.py --test-only
```

返回码：0

输出：

```text
OpenCV 3.2.0 opened camera 0 and read frame 640x480.
```

思考：串行执行后摄像头读取正常，前面的失败确认为资源占用，不是摄像头故障。

#### 命令 7

```bash
PYTHONPATH=/home/gwb/Fang/opencv_local/usr/lib/python3/dist-packages LD_LIBRARY_PATH=/home/gwb/Fang/opencv_local/usr/lib/aarch64-linux-gnu:/home/gwb/Fang/opencv_local/usr/lib python3 -c "import cv2; print(cv2.__version__); print(hasattr(cv2, 'dnn')); print(hasattr(cv2.dnn, 'readNetFromDarknet') if hasattr(cv2, 'dnn') else 'no dnn')"
```

返回码：0

输出：

```text
3.2.0
False
no dnn
```

问题：当前 OpenCV 没有 `cv2.dnn` 模块。

思考：不能直接使用 `cv2.dnn.readNetFromDarknet()` 实现 YOLO。需要换方案。

#### 命令 8

```bash
command -v wget
```

返回码：0

输出：

```text
/usr/bin/wget
```

思考：可以使用 `wget` 下载源码和模型文件。

### 3. 检查是否已有其它推理框架

目的：判断是否可直接用 PyTorch、ONNX Runtime 或系统 Darknet。

#### 命令 9

```bash
python3 -c "import torch; print(torch.__version__)"
```

返回码：1

输出：

```text
Traceback (most recent call last):
  File "<string>", line 1, in <module>
ModuleNotFoundError: No module named 'torch'
```

#### 命令 10

```bash
python3 -c "import onnxruntime; print(onnxruntime.__version__)"
```

返回码：1

输出：

```text
Traceback (most recent call last):
  File "<string>", line 1, in <module>
ModuleNotFoundError: No module named 'onnxruntime'
```

#### 命令 11

```bash
command -v darknet
```

返回码：1

输出：

```text

```

#### 命令 12

```bash
python3 -c "import cv2; print('system cv2', cv2.__version__)"
```

返回码：1

输出：

```text
Traceback (most recent call last):
  File "<string>", line 1, in <module>
ModuleNotFoundError: No module named 'cv2'
```

思考：没有 PyTorch、ONNX Runtime、系统 Darknet，也没有系统级 OpenCV。可行方案是本地编译 Darknet CPU 动态库，再由 Python 调用。

### 4. 检查构建工具

#### 命令 13

```bash
command -v git
```

返回码：0

输出：

```text
/usr/bin/git
```

#### 命令 14

```bash
command -v make
```

返回码：0

输出：

```text
/usr/bin/make
```

#### 命令 15

```bash
command -v gcc
```

返回码：0

输出：

```text
/usr/bin/gcc
```

#### 命令 16

```bash
gcc --version
```

返回码：0

输出：

```text
gcc (Ubuntu/Linaro 7.5.0-3ubuntu1~18.04) 7.5.0
Copyright (C) 2017 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
```

思考：本机有编译 Darknet 所需的基本工具。

### 5. 尝试从 GitHub 获取 Darknet 源码

#### 命令 17

```bash
mkdir -p third_party models/yolov3-tiny
```

返回码：0

输出：

```text

```

#### 命令 18

```bash
git clone https://github.com/pjreddie/darknet.git third_party/darknet
```

返回码：124

输出：

```text
command timed out after 120022 milliseconds
正克隆到 'third_party/darknet'...
```

问题：GitHub 克隆 120 秒超时。

思考：网络到 GitHub 当前很慢或不可用，需要检查是否留下半成品目录，并寻找镜像下载。

#### 命令 19

```bash
ls -la third_party
```

返回码：0

输出：

```text
总用量 12
drwxrwxr-x 3 gwb gwb 4096 6月   1 11:18 .
drwxrwxr-x 7 gwb gwb 4096 6月   1 11:17 ..
drwxrwxr-x 3 gwb gwb 4096 6月   1 11:18 darknet
```

#### 命令 20

```bash
ls -la third_party/darknet
```

返回码：0

输出：

```text
总用量 12
drwxrwxr-x 3 gwb gwb 4096 6月   1 11:18 .
drwxrwxr-x 3 gwb gwb 4096 6月   1 11:18 ..
drwxrwxr-x 7 gwb gwb 4096 6月   1 11:20 .git
```

#### 命令 21

```bash
git -C third_party/darknet status --short
```

返回码：0

输出：

```text

```

#### 命令 22

```bash
git -C third_party/darknet rev-parse --is-inside-work-tree
```

返回码：0

输出：

```text
true
```

#### 命令 23

```bash
git -C third_party/darknet rev-parse HEAD
```

返回码：128

输出：

```text
HEAD
fatal: 有歧义的参数 'HEAD'：未知的版本或路径不存在于工作区中。
使用 '--' 来分隔版本和路径，例如：
'git <命令> [<版本>...] -- [<文件>...]'
```

问题：目录是 Git 工作区，但没有有效 `HEAD`，说明克隆未完成。

#### 命令 24

```bash
git -C third_party/darknet branch --show-current
```

返回码：129

输出：

```text
error: unknown option `show-current'
用法：git branch [<选项>] [-r | -a] [--merged | --no-merged]
   或：git branch [<选项>] [-l] [-f] <分支名> [<起始点>]
   或：git branch [<选项>] [-r] (-d | -D) <分支名>...
   或：git branch [<选项>] (-m | -M) [<旧分支>] <新分支>
   或：git branch [<选项>] (-c | -C) [<老分支>] <新分支>
   或：git branch [<选项>] [-r | -a] [--points-at]
   或：git branch [<选项>] [-r | -a] [--format]

通用选项
    -v, --verbose         显示哈希值和主题，若参数出现两次则显示上游分支
    -q, --quiet           不显示信息
    -t, --track           设置跟踪模式（参见 git-pull(1)）
    -u, --set-upstream-to <上游>
                          改变上游信息
    --unset-upstream      取消上游信息的设置
    --color[=<何时>]      使用彩色输出
    -r, --remotes         作用于远程跟踪分支
    --contains <提交>     只打印包含该提交的分支
    --no-contains <提交>  只打印不包含该提交的分支
    --abbrev[=<n>]        用 <n> 位数字显示 SHA-1 哈希值

具体的 git-branch 动作：
    -a, --all             列出远程跟踪及本地分支
    -d, --delete          删除完全合并的分支
    -D                    删除分支（即使没有合并）
    -m, --move            移动/重命名一个分支，以及它的引用日志
    -M                    移动/重命名一个分支，即使目标已存在
    -c, --copy            拷贝一个分支和它的引用日志
    -C                    拷贝一个分支，即使目标已存在
    --list                列出分支名
    -l, --create-reflog   创建分支的引用日志
    --edit-description    标记分支的描述
    -f, --force           强制创建、移动/重命名、删除
    --merged <提交>       只打印已经合并的分支
    --no-merged <提交>    只打印尚未合并的分支
    --column[=<风格>]     以列的方式显示分支
    --sort <key>          排序的字段名
    --points-at <对象>    只打印指向该对象的分支
    -i, --ignore-case     排序和过滤属于大小写不敏感
    --format <格式>       输出格式
```

说明：此错误是因为系统 Git 版本不支持 `--show-current`，不是主要问题。主要问题仍是 GitHub 克隆未完成。

### 6. 尝试 GitHub tar 包，仍然超时

#### 命令 25

```bash
wget -O third_party/darknet-master.tar.gz https://github.com/pjreddie/darknet/archive/refs/heads/master.tar.gz
```

返回码：124

输出：

```text
command timed out after 120048 milliseconds
--2026-06-01 11:20:41--  https://github.com/pjreddie/darknet/archive/refs/heads/master.tar.gz
正在解析主机 github.com (github.com)... 20.205.243.166
正在连接 github.com (github.com)|20.205.243.166|:443... 已连接。
已发出 HTTP 请求，正在等待回应... 没有接收到数据。
重试中。

--2026-06-01 11:22:13--  (尝试次数： 2)  https://github.com/pjreddie/darknet/archive/refs/heads/master.tar.gz
正在连接 github.com (github.com)|20.205.243.166|:443...
```

问题：GitHub tar 包也超时。

思考：继续依赖 GitHub 会阻塞任务，改为查找可用镜像。随后检索到 SourceForge 上有 Darknet mirror。

### 7. 检查 apt 是否有可用 Darknet/YOLO 包

#### 命令 26

```bash
apt-cache search darknet
```

返回码：0

输出：

```text

```

#### 命令 27

```bash
apt-cache search yolo
```

返回码：0

输出：

```text

```

#### 命令 28

```bash
apt-cache search opencv | grep -E 'dnn|opencv' | head -n 50
```

返回码：0

输出：

```text
cl-opencv-apps - opencv_apps Robot OS package - LISP bindings
gstreamer1.0-opencv - GStreamer OpenCV plugins
libgstreamer-opencv1.0-0 - GStreamer OpenCV libraries
libopencv-apps-dev - Opencv_apps Robot OS package - development files
libopencv-apps0d - opencv_apps Robot OS package - runtime files
libopencv-calib3d-dev - development files for libopencv-calibd3.2
libopencv-calib3d3.2 - computer vision Camera Calibration library
libopencv-contrib-dev - development files for libopencv-contrib3.2
libopencv-core-dev - development files for libopencv-core3.2
libopencv-core3.2 - computer vision core library
libopencv-features2d-dev - development files for libopencv-features2d3.2
libopencv-features2d3.2 - computer vision Feature Detection and Descriptor Extraction library
libopencv-flann-dev - development files for libopencv-flann3.2
libopencv-flann3.2 - computer vision Clustering and Search in Multi-Dimensional spaces library
libopencv-highgui-dev - development files for libopencv-highgui3.2
libopencv-highgui3.2 - computer vision High-level GUI and Media I/O library
libopencv-imgcodecs-dev - development files for libopencv-imgcodecs3.2
libopencv-imgcodecs3.2 - computer vision Image Codecs library
libopencv-imgproc-dev - development files for libopencv-imgproc3.2
libopencv-imgproc3.2 - computer vision Image Processing library
libopencv-ml-dev - development files for libopencv-ml3.2
libopencv-ml3.2 - computer vision Machine Learning library
libopencv-objdetect-dev - development files for libopencv-objdetect3.2
libopencv-photo-dev - development files for libopencv-photo3.2
libopencv-photo3.2 - computer vision computational photography library
libopencv-shape-dev - development files for libopencv-shape3.2
libopencv-shape3.2 - computer vision shape descriptors and matchers library
libopencv-stitching-dev - development files for libopencv-stitching3.2
libopencv-stitching3.2 - computer vision image stitching library
libopencv-superres-dev - development files for libopencv-superres3.2
libopencv-superres3.2 - computer vision Super Resolution library
libopencv-ts-dev - development files for TS library of OpenCV (Open Computer Vision)
libopencv-video-dev - development files for libopencv-video3.2
libopencv-video3.2 - computer vision Video analysis library
libopencv-videoio-dev - development files for libopencv-videoio3.2
libopencv-videoio3.2 - computer vision Video I/O library
libopencv-videostab-dev - development files for libopencv-videostab3.2
libopencv-videostab3.2 - computer vision video stabilization library
libopencv-viz-dev - development files for libopencv-viz3.2
libopencv-viz3.2 - computer vision 3D data visualization library
libopencv3.2-java - Java bindings for the computer vision library
libopencv3.2-jni - Java jni library
opencv-data - development data for opencv
opencv-doc - OpenCV documentation and examples
python-opencv - Python bindings for the computer vision library
python-opencv-apps - opencv_apps Robot OS package - Python bindings
python3-opencv - Python 3 bindings for the computer vision library
ros-opencv-apps - opencv_apps Robot OS package - apps
libopencv-contrib3.2 - computer vision contrlib library
```

思考：apt 没有 Darknet/YOLO 包，OpenCV 源也仍然是 3.2 系列，无法解决 `cv2.dnn` 缺失。继续走 SourceForge Darknet 源码方案。

### 8. 从 SourceForge 下载 Darknet 源码并解压

#### 命令 29

```bash
wget -O third_party/darknet-source.tar.gz 'https://sourceforge.net/projects/darknet.mirror/files/darknet%20source%20code.tar.gz/download'
```

返回码：0

输出摘录：

```text
--2026-06-01 11:23:27--  https://sourceforge.net/projects/darknet.mirror/files/darknet%20source%20code.tar.gz/download
正在解析主机 sourceforge.net (sourceforge.net)... 2606:4700::6812:c95, 2606:4700::6812:d95, 104.18.12.149, ...
正在连接 sourceforge.net (sourceforge.net)|2606:4700::6812:c95|:443... 已连接。
已发出 HTTP 请求，正在等待回应... 302 Found
位置：https://downloads.sourceforge.net/project/darknet.mirror/darknet%20source%20code.tar.gz?... [跟随至新的 URL]
...
长度： 3341948 (3.2M) [application/x-gzip]
正在保存至: “third_party/darknet-source.tar.gz”
...
2026-06-01 11:24:06 (91.9 KB/s) - 已保存 “third_party/darknet-source.tar.gz” [3341948/3341948])
```

说明：终端实际输出包含完整下载进度条，内容很长；关键反馈是下载成功并保存 3.2M 文件。

#### 命令 30

```bash
tar -tzf third_party/darknet-source.tar.gz | head -n 20
```

返回码：0

输出：

```text
darknet-master/
darknet-master/.gitignore
darknet-master/LICENSE
darknet-master/LICENSE.fuck
darknet-master/LICENSE.gen
darknet-master/LICENSE.gpl
darknet-master/LICENSE.meta
darknet-master/LICENSE.mit
darknet-master/LICENSE.v1
darknet-master/Makefile
darknet-master/README.md
darknet-master/cfg/
darknet-master/cfg/alexnet.cfg
darknet-master/cfg/cifar.cfg
darknet-master/cfg/cifar.test.cfg
darknet-master/cfg/coco.data
darknet-master/cfg/combine9k.data
darknet-master/cfg/darknet.cfg
darknet-master/cfg/darknet19.cfg
darknet-master/cfg/darknet19_448.cfg
```

思考：源码包结构正常，包含 Makefile、cfg、data 等文件。

#### 命令 31

```bash
tar -xzf third_party/darknet-source.tar.gz -C third_party
```

返回码：0

输出：

```text

```

### 9. 检查 Darknet Makefile

#### 命令 32

```bash
ls -la third_party/darknet-master | head -n 30
```

返回码：0

输出：

```text
总用量 116
drwxrwxr-x 9 gwb gwb  4096 7月  18  2022 .
drwxrwxr-x 4 gwb gwb  4096 6月   1 11:24 ..
drwxrwxr-x 2 gwb gwb  4096 7月  18  2022 cfg
drwxrwxr-x 3 gwb gwb  4096 7月  18  2022 data
drwxrwxr-x 2 gwb gwb  4096 7月  18  2022 examples
-rw-rw-r-- 1 gwb gwb   200 7月  18  2022 .gitignore
drwxrwxr-x 2 gwb gwb  4096 7月  18  2022 include
-rw-rw-r-- 1 gwb gwb   515 7月  18  2022 LICENSE
-rw-rw-r-- 1 gwb gwb   474 7月  18  2022 LICENSE.fuck
-rw-rw-r-- 1 gwb gwb  6652 7月  18  2022 LICENSE.gen
-rw-rw-r-- 1 gwb gwb 35141 7月  18  2022 LICENSE.gpl
-rw-rw-r-- 1 gwb gwb   360 7月  18  2022 LICENSE.meta
-rw-rw-r-- 1 gwb gwb  1071 7月  18  2022 LICENSE.mit
-rw-rw-r-- 1 gwb gwb   461 7月  18  2022 LICENSE.v1
-rw-rw-r-- 1 gwb gwb  3040 7月  18  2022 Makefile
drwxrwxr-x 2 gwb gwb  4096 7月  18  2022 python
-rw-rw-r-- 1 gwb gwb  5072 7月  18  2022 README.md
drwxrwxr-x 2 gwb gwb  4096 7月  18  2022 scripts
drwxrwxr-x 2 gwb gwb  4096 7月  18  2022 src
```

#### 命令 33

```bash
grep -n "LIBSO" third_party/darknet-master/Makefile
```

返回码：1

输出：

```text

```

思考：这个 Makefile 没有 `LIBSO` 开关，但后续检查发现默认目标会构建 `libdarknet.so`。

#### 命令 34

```bash
grep -n "^GPU\|^CUDNN\|^OPENCV\|^OPENMP" third_party/darknet-master/Makefile
```

返回码：0

输出：

```text
1:GPU=0
2:CUDNN=0
3:OPENCV=0
4:OPENMP=0
```

思考：GPU、CUDNN、Darknet 内部 OpenCV 都关闭，适合当前 CPU 环境。Python 仍会用已有 OpenCV 做显示。

#### 命令 35

```bash
sed -n '1,180p' third_party/darknet-master/Makefile
```

返回码：0

输出摘录：

```text
GPU=0
CUDNN=0
OPENCV=0
OPENMP=0
DEBUG=0
...
SLIB=libdarknet.so
ALIB=libdarknet.a
EXEC=darknet
...
all: obj backup results $(SLIB) $(ALIB) $(EXEC)
...
$(SLIB): $(OBJS)
	$(CC) $(CFLAGS) -shared $^ -o $@ $(LDFLAGS)
```

思考：`make` 默认会生成 `libdarknet.so`、`libdarknet.a` 和 `darknet` 可执行文件。

### 10. 编译 Darknet

#### 命令 36

```bash
make -j2
```

工作目录：`/home/gwb/Fang/third_party/darknet-master`

返回码：0

输出摘录：

```text
mkdir -p obj
mkdir -p backup
mkdir -p results
gcc -Iinclude/ -Isrc/ -Wall -Wno-unused-result -Wno-unknown-pragmas -Wfatal-errors -fPIC -Ofast -c ./src/gemm.c -o obj/gemm.o
gcc -Iinclude/ -Isrc/ -Wall -Wno-unused-result -Wno-unknown-pragmas -Wfatal-errors -fPIC -Ofast -c ./src/utils.c -o obj/utils.o
...
gcc -Wall -Wno-unused-result -Wno-unknown-pragmas -Wfatal-errors -fPIC -Ofast -shared obj/gemm.o ... obj/image_opencv.o -o libdarknet.so -lm -pthread 
ar rcs libdarknet.a obj/gemm.o ... obj/image_opencv.o
gcc -Iinclude/ -Isrc/ -Wall -Wno-unused-result -Wno-unknown-pragmas -Wfatal-errors -fPIC -Ofast obj/captcha.o ... obj/darknet.o libdarknet.a -o darknet -lm -pthread  libdarknet.a
```

说明：编译输出包含大量 `.c` 到 `.o` 的 gcc 编译行，关键反馈是返回码 0，并生成 `libdarknet.so` 与 `darknet`。

### 11. 准备 YOLOv3-tiny 模型文件

#### 命令 37

```bash
cp third_party/darknet-master/cfg/yolov3-tiny.cfg models/yolov3-tiny/yolov3-tiny.cfg
```

返回码：0

输出：

```text

```

#### 命令 38

```bash
cp third_party/darknet-master/data/coco.names models/yolov3-tiny/coco.names
```

返回码：0

输出：

```text

```

#### 命令 39

```bash
ls -l third_party/darknet-master/libdarknet.so third_party/darknet-master/darknet
```

返回码：0

输出：

```text
-rwxrwxr-x 1 gwb gwb 577880 6月   1 11:25 third_party/darknet-master/darknet
-rwxrwxr-x 1 gwb gwb 505072 6月   1 11:25 third_party/darknet-master/libdarknet.so
```

#### 命令 40

```bash
wget -O models/yolov3-tiny/yolov3-tiny.weights https://pjreddie.com/media/files/yolov3-tiny.weights
```

返回码：0

输出摘录：

```text
--2026-06-01 11:25:46--  https://pjreddie.com/media/files/yolov3-tiny.weights
正在解析主机 pjreddie.com (pjreddie.com)... 2606:4700:3037::6815:589c, 2606:4700:3030::ac43:b9c7, 172.67.185.199, ...
正在连接 pjreddie.com (pjreddie.com)|2606:4700:3037::6815:589c|:443... 已连接。
已发出 HTTP 请求，正在等待回应... 301 Moved Permanently
位置：https://data.pjreddie.com/files/yolov3-tiny.weights [跟随至新的 URL]
--2026-06-01 11:25:47--  https://data.pjreddie.com/files/yolov3-tiny.weights
正在解析主机 data.pjreddie.com (data.pjreddie.com)... 2606:4700:3030::ac43:b9c7, 2606:4700:3037::6815:589c, 172.67.185.199, ...
正在连接 data.pjreddie.com (data.pjreddie.com)|2606:4700:3030::ac43:b9c7|:443... 已连接。
已发出 HTTP 请求，正在等待回应... 200 OK
长度： 35434956 (34M) [application/octet-stream]
正在保存至: “models/yolov3-tiny/yolov3-tiny.weights”
...
2026-06-01 11:26:17 (1.19 MB/s) - 已保存 “models/yolov3-tiny/yolov3-tiny.weights” [35434956/35434956])
```

说明：终端实际输出包含 708 行下载进度，执行界面中间部分被截断显示；关键反馈是权重文件 34M 成功保存。

#### 命令 41

```bash
ls -lh models/yolov3-tiny
```

返回码：0

输出：

```text
总用量 34M
-rw-rw-r-- 1 gwb gwb  625 6月   1 11:25 coco.names
-rw-rw-r-- 1 gwb gwb 1.9K 6月   1 11:25 yolov3-tiny.cfg
-rw-rw-r-- 1 gwb gwb  34M 4月  22  2025 yolov3-tiny.weights
```

#### 命令 42

```bash
wc -l models/yolov3-tiny/coco.names
```

返回码：0

输出：

```text
80 models/yolov3-tiny/coco.names
```

思考：COCO 类别文件有 80 类，符合 YOLOv3-tiny COCO 模型。

#### 命令 43

```bash
nm -D third_party/darknet-master/libdarknet.so | grep -E 'load_network|network_width|network_predict_image|get_network_boxes|do_nms_obj|free_detections'
```

返回码：0

输出：

```text
000000000005a4d8 T do_nms_obj
000000000004f388 T free_detections
000000000004f308 T get_network_boxes
000000000004df30 T load_network
000000000004f3f0 T network_predict_image
000000000004f4a0 T network_width
```

#### 命令 44

```bash
nm -D third_party/darknet-master/libdarknet.so | grep -E 'network_height|make_image|free_image'
```

返回码：0

输出：

```text
000000000003db50 T free_image
00000000000385e0 T make_image
000000000004f4a8 T network_height
```

思考：动态库导出了 Python ctypes 调用所需的核心函数。

### 12. 编写 Python YOLO 实时检测脚本

操作：新增 `/home/gwb/Fang/yolo_realtime.py`。

设计原因：

- 当前 OpenCV 没有 DNN 模块，所以不走 `cv2.dnn`。
- Python 使用 `ctypes.CDLL` 加载 `libdarknet.so`。
- 摄像头读取和窗口显示仍用 OpenCV。
- 检测框绘制也用 OpenCV。
- 为 CPU 推理增加 `--skip-frames`，避免每帧推理导致窗口过慢。
- 增加 `--test-only` 用于非交互测试。
- 增加 `--image` 用于静态图验证 Python 推理链路。

#### 命令 45

```bash
chmod +x yolo_realtime.py
```

返回码：0

输出：

```text

```

### 13. 摄像头单帧 YOLO 测试

#### 命令 46

```bash
./yolo_realtime.py --test-only --save-frame yolo_test.jpg
```

返回码：0

输出：

```text
Loaded YOLOv3-tiny 416x416 with 80 classes.
Processed one frame from camera 0; detections=0.
layer     filters    size              input                output
    0 conv     16  3 x 3 / 1   416 x 416 x   3   ->   416 x 416 x  16  0.150 BFLOPs
    1 max          2 x 2 / 2   416 x 416 x  16   ->   208 x 208 x  16
...
   23 yolo
Loading weights from /home/gwb/Fang/models/yolov3-tiny/yolov3-tiny.weights...Done!
```

说明：Darknet 的 C 层会打印网络结构，部分输出在 Python 提示之后刷新。`detections=0` 表示当前摄像头画面没有超过阈值的 COCO 目标，不代表失败。

### 14. 用 Darknet 自带示例图验证模型有效性

#### 命令 47

```bash
./darknet detect cfg/yolov3-tiny.cfg ../../models/yolov3-tiny/yolov3-tiny.weights data/dog.jpg -dont_show
```

工作目录：`/home/gwb/Fang/third_party/darknet-master`

返回码：0

输出：

```text
data/dog.jpg: Predicted in 1.390313 seconds.
dog: 57%
car: 52%
truck: 56%
car: 62%
bicycle: 59%
layer     filters    size              input                output
    0 conv     16  3 x 3 / 1   416 x 416 x   3   ->   416 x 416 x  16  0.150 BFLOPs
    1 max          2 x 2 / 2   416 x 416 x  16   ->   208 x 208 x  16
...
   23 yolo
Loading weights from ../../models/yolov3-tiny/yolov3-tiny.weights...Done!
```

思考：Darknet 命令行可以识别 dog、car、truck、bicycle，证明模型权重有效。

### 15. 增加静态图测试参数并验证 Python 推理链路

操作：修改 `yolo_realtime.py`，新增 `--image` 参数，用于静态图片推理和保存检测结果。

#### 命令 48

```bash
./yolo_realtime.py --image third_party/darknet-master/data/dog.jpg --test-only --save-frame yolo_dog_result.jpg
```

返回码：0

输出：

```text
Loaded YOLOv3-tiny 416x416 with 80 classes.
Processed image third_party/darknet-master/data/dog.jpg; detections=6.
layer     filters    size              input                output
    0 conv     16  3 x 3 / 1   416 x 416 x   3   ->   416 x 416 x  16  0.150 BFLOPs
    1 max          2 x 2 / 2   416 x 416 x  16   ->   208 x 208 x  16
...
   23 yolo
Loading weights from /home/gwb/Fang/models/yolov3-tiny/yolov3-tiny.weights...Done!
```

思考：Python 调用 `libdarknet.so` 的链路有效，静态图检出 6 个目标。

#### 命令 49

```bash
file yolo_dog_result.jpg
```

返回码：0

输出：

```text
yolo_dog_result.jpg: JPEG image data, JFIF standard 1.01, aspect ratio, density 1x1, segment length 16, baseline, precision 8, 768x576, frames 3
```

#### 命令 50

```bash
file yolo_test.jpg
```

返回码：0

输出：

```text
yolo_test.jpg: JPEG image data, JFIF standard 1.01, aspect ratio, density 1x1, segment length 16, baseline, precision 8, 640x480, frames 3
```

#### 命令 51

```bash
ls -lh yolo_dog_result.jpg yolo_test.jpg
```

返回码：0

输出：

```text
-rw-rw-r-- 1 gwb gwb 179K 6月   1 11:29 yolo_dog_result.jpg
-rw-rw-r-- 1 gwb gwb  71K 6月   1 11:28 yolo_test.jpg
```

反馈：人工查看 `yolo_dog_result.jpg`，确认图像中有检测框和标签。

### 16. 验证实时窗口启动

#### 命令 52

```bash
timeout 10s ./yolo_realtime.py --skip-frames 10 --thresh 0.25
```

返回码：124

输出：

```text
Loaded YOLOv3-tiny 416x416 with 80 classes.
Showing YOLO detections. Press q or Esc in the window to exit.
layer     filters    size              input                output
    0 conv     16  3 x 3 / 1   416 x 416 x   3   ->   416 x 416 x  16  0.150 BFLOPs
    1 max          2 x 2 / 2   416 x 416 x  16   ->   208 x 208 x  16
...
   23 yolo
Loading weights from /home/gwb/Fang/models/yolov3-tiny/yolov3-tiny.weights...Done!
```

说明：返回码 124 是 `timeout 10s` 到时终止。脚本已经打印实时窗口提示，说明进入了窗口显示循环。

### 17. 创建桌面使用说明

操作：新增 `/home/gwb/Desktop/yolo_realtime_usage.md`。

内容包括：

- 实时运行命令。
- 推荐 CPU 参数。
- 摄像头单帧测试命令。
- 静态图片测试命令。
- 当前文件说明。
- 当前验证结论。

### 18. 创建本记录文件

操作：新增 `/home/gwb/Fang/RECORD2.md`。

## 使用方法

实时检测：

```bash
cd /home/gwb/Fang
./yolo_realtime.py
```

CPU 推荐参数：

```bash
cd /home/gwb/Fang
./yolo_realtime.py --skip-frames 10 --thresh 0.25
```

摄像头单帧测试：

```bash
cd /home/gwb/Fang
./yolo_realtime.py --test-only --save-frame yolo_test.jpg
```

静态图测试：

```bash
cd /home/gwb/Fang
./yolo_realtime.py --image third_party/darknet-master/data/dog.jpg --test-only --save-frame yolo_dog_result.jpg
```

退出实时窗口：在窗口里按 `q` 或 `Esc`。
