# 摄像头检查与 OpenCV 预览脚本任务记录

记录位置：`/home/gwb/Fang/RECORD.md`

脚本位置：`/home/gwb/Fang/camera_viewer.py`

桌面使用说明：`/home/gwb/Desktop/camera_viewer_usage.md`

测试截图：

- `camera_test.jpg`：使用 `ffmpeg` 直接从 V4L2 设备抓取。
- `opencv_test.jpg`：使用本次编写的 OpenCV 脚本抓取。

## 结论

摄像头可以被正常驱动和读取。

依据：

1. 系统存在摄像头设备节点 `/dev/video0`。
2. 当前用户 `gwb` 属于 `video` 组，具备访问视频设备的权限。
3. `ffmpeg` 可以枚举 `/dev/video0` 的格式并抓取一帧 640x480 图像。
4. 系统原本没有 `cv2` 模块；因 root 安装受限，改为下载并解包 Ubuntu 软件源中的 OpenCV 相关 `.deb` 到本地目录 `opencv_local`。
5. 本地 OpenCV 3.2.0 可以被 Python 导入。
6. `camera_viewer.py --test-only` 可以用 OpenCV 打开摄像头并读取 640x480 图像。
7. `camera_viewer.py` 可以启动实时预览窗口；测试时使用 `timeout 5s` 自动结束，返回码 124 是预期的超时退出。

## 任务过程记录


### 2. 检查摄像头设备、用户权限、Python 和 OpenCV

目的：确认系统是否识别摄像头，当前用户是否有权限访问，Python 和 OpenCV 是否可用。

#### 命令 5

```bash
ls -l /dev/video*
```

返回码：0

输出：

```text
crw-rw----+ 1 root video 81, 0 6月   1 10:25 /dev/video0
```

思考：系统存在 `/dev/video0`，说明内核已识别到视频采集设备。


#### 命令 7

```bash
python3 --version
```

返回码：0

输出：

```text
Python 3.6.9
```

思考：系统 Python 是 3.6.9，后续选择 OpenCV 版本时必须兼容 Python 3.6。

#### 命令 8

```bash
python3 -c "import cv2; print(cv2.__version__)"
```

返回码：1

输出：

```text
Traceback (most recent call last):
  File "<string>", line 1, in <module>
ModuleNotFoundError: No module named 'cv2'
```

问题：Python 中没有 `cv2` 模块。

思考：摄像头设备存在，但无法直接编写并运行 OpenCV 脚本；需要补齐 OpenCV。

#### 命令 9

```bash
command -v v4l2-ctl
```

返回码：1

输出：

```text

```

问题：系统未安装 `v4l2-ctl`。

思考：不能用 `v4l2-ctl` 进一步枚举设备；改用已安装的 `ffmpeg` 检测 V4L2。



### 4. 尝试系统安装 OpenCV 并处理权限问题

目的：优先尝试最标准的系统安装方式。

#### 命令 14

```bash
sudo apt-get install -y python3-opencv
```

返回码：1

输出：

```text
sudo: 有效用户 ID 不是 0，/usr/bin/sudo 位于一个设置了“nosuid”选项的文件系统或没有 root 权限的 NFS 文件系统中吗？
```

问题：`sudo` 无法获得 root 权限，不能直接安装系统包。

思考：这不是 OpenCV 包本身的问题，而是当前运行环境对 `sudo` 的限制。为避免卡住任务，改为免 root 的方案：下载 `.deb` 包并解包到用户目录，通过环境变量加载。

### 5. 检查依赖、缓存和可用视频工具

目的：确认 OpenCV 依赖关系，同时找可替代工具验证摄像头驱动链路。

#### 命令 15

```bash
apt-cache depends python3-opencv
```

返回码：0

输出：

```text
python3-opencv
  依赖: libopencv-calib3d3.2
  依赖: libopencv-features2d3.2
  依赖: libopencv-flann3.2
  依赖: libopencv-highgui3.2
  依赖: libopencv-imgcodecs3.2
  依赖: libopencv-imgproc3.2
  依赖: libopencv-ml3.2
  依赖: libopencv-objdetect3.2
  依赖: libopencv-photo3.2
  依赖: libopencv-shape3.2
  依赖: libopencv-stitching3.2
  依赖: libopencv-superres3.2
  依赖: libopencv-video3.2
  依赖: libopencv-videoio3.2
  依赖: libopencv-videostab3.2
  依赖: libopencv-viz3.2
  依赖: python3
  依赖: python3
  依赖: python3-numpy
  依赖: <python3-numpy-abi9>
    python3-numpy
  依赖: libc6
  依赖: libgcc1
    libgcc-s1
  依赖: libopencv-contrib3.2
  依赖: libopencv-core3.2
  依赖: libstdc++6
```

思考：`python3-opencv` 的 Python 绑定还依赖一批 OpenCV 共享库，后续本地解包时需要一并处理。

#### 命令 16

```bash
ls -la /var/cache/apt/archives | head -n 50
```

返回码：0

输出：

```text
总用量 284836
drwxr-xr-x 3 root root     36864 6月   1 10:27 .
drwxr-xr-x 3 root root      4096 6月   1 10:50 ..
-rw-r--r-- 1 root root   7156504 4月  16  2021 cpp-10_10.3.0-1ubuntu1~18.04~1_arm64.deb
-rw-r--r-- 1 root root     89244 3月  27  2018 dh-python_3.20180325ubuntu2_all.deb
-rw-r--r-- 1 root root      5544 2月  26  2018 fcitx_1%3a4.2.9.6-1_all.deb
-rw-r--r-- 1 root root     39916 2月  26  2018 fcitx-bin_1%3a4.2.9.6-1_arm64.deb
-rw-r--r-- 1 root root     15082 7月   6  2014 fcitx-chewing_0.2.2-1_arm64.deb
-rw-r--r-- 1 root root      9444 2月  26  2018 fcitx-config-common_0.4.10-1_all.deb
-rw-r--r-- 1 root root     30932 2月  26  2018 fcitx-config-gtk_0.4.10-1_arm64.deb
-rw-r--r-- 1 root root     82060 2月  26  2018 fcitx-data_1%3a4.2.9.6-1_all.deb
-rw-r--r-- 1 root root      5492 2月  26  2018 fcitx-frontend-all_1%3a4.2.9.6-1_all.deb
-rw-r--r-- 1 root root     13480 2月  26  2018 fcitx-frontend-gtk2_1%3a4.2.9.6-1_arm64.deb
-rw-r--r-- 1 root root     14284 2月  26  2018 fcitx-frontend-gtk3_1%3a4.2.9.6-1_arm64.deb
-rw-r--r-- 1 root root     53164 2月  26  2018 fcitx-frontend-qt4_1%3a4.2.9.6-1_arm64.deb
-rw-r--r-- 1 root root     32084 4月  16  2018 fcitx-frontend-qt5_1.1.1-1build3_arm64.deb
-rw-r--r-- 1 root root     19428 3月  10  2018 fcitx-module-cloudpinyin_0.3.6-1_arm64.deb
-rw-r--r-- 1 root root     27360 2月  26  2018 fcitx-module-dbus_1%3a4.2.9.6-1_arm64.deb
-rw-r--r-- 1 root root     16252 2月  26  2018 fcitx-module-kimpanel_1%3a4.2.9.6-1_arm64.deb
-rw-r--r-- 1 root root     15336 2月  26  2018 fcitx-module-lua_1%3a4.2.9.6-1_arm64.deb
-rw-r--r-- 1 root root   1438676 2月  26  2018 fcitx-modules_1%3a4.2.9.6-1_arm64.deb
-rw-r--r-- 1 root root     75596 2月  26  2018 fcitx-module-x11_1%3a4.2.9.6-1_arm64.deb
-rw-r--r-- 1 root root   2053936 2月  26  2018 fcitx-pinyin_1%3a4.2.9.6-1_arm64.deb
-rw-r--r-- 1 root root     48816 11月  2  2017 fcitx-sunpinyin_0.4.2-1_arm64.deb
-rw-r--r-- 1 root root     42240 2月  26  2018 fcitx-table_1%3a4.2.9.6-1_arm64.deb
-rw-r--r-- 1 root root     26928 2月  26  2018 fcitx-table-cangjie_1%3a4.2.9.6-1_all.deb
-rw-r--r-- 1 root root    497228 2月  26  2018 fcitx-table-wubi_1%3a4.2.9.6-1_all.deb
-rw-r--r-- 1 root root     89164 2月  26  2018 fcitx-ui-classic_1%3a4.2.9.6-1_arm64.deb
-rw-r--r-- 1 root root    147120 10月  2  2017 fcitx-ui-qimpanel_2.1.3-1_arm64.deb
-rw-r--r-- 1 root root   8287866 11月 24  2013 fonts-arphic-ukai_0.2.20080216.2-4ubuntu2_all.deb
-rw-r--r-- 1 root root   7813672 4月  17  2018 fonts-arphic-uming_0.2.20080216.2-7ubuntu3_all.deb
-rw-r--r-- 1 root root 131011984 7月   1  2019 fonts-noto-cjk-extra_1%3a20190409+repack1-0ubuntu0.18.04_all.deb
-rw-r--r-- 1 root root   8116968 4月  16  2021 g++-10_10.3.0-1ubuntu1~18.04~1_arm64.deb
-rw-r--r-- 1 root root  14654876 4月  16  2021 gcc-10_10.3.0-1ubuntu1~18.04~1_arm64.deb
-rw-r--r-- 1 root root     20176 4月  16  2021 gcc-10-base_10.3.0-1ubuntu1~18.04~1_arm64.deb
-rw-r--r-- 1 root root     19228 7月  13  2023 gcc-11-base_11.4.0-2ubuntu1~18.04_arm64.deb
-rw-r--r-- 1 root root     19704 3月  20 05:47 gcc-16-base_16-20260315-1ubuntu1~18~ppa3_arm64.deb
-rw-r--r-- 1 root root     99852 7月  17  2018 gnome-getting-started-docs-zh-hk_3.28.2-0ubuntu0.1_all.deb
-rw-r--r-- 1 root root     99800 7月  17  2018 gnome-getting-started-docs-zh-tw_3.28.2-0ubuntu0.1_all.deb
-rw-r--r-- 1 root root    146176 7月  17  2018 gnome-user-docs-zh-hans_3.28.2+git20180715-0ubuntu0.1_all.deb
-rw-r--r-- 1 root root    685412 5月  31  2019 ibus-libpinyin_1.11.0-1ubuntu0.18.04.1_arm64.deb
-rw-r--r-- 1 root root    334718 12月 11  2016 ibus-table-cangjie3_1.8.2-2_all.deb
-rw-r--r-- 1 root root    623872 12月 11  2016 ibus-table-cangjie5_1.8.2-2_all.deb
-rw-r--r-- 1 root root     69938 12月 11  2016 ibus-table-quick-classic_1.8.2-2_all.deb
-rw-r--r-- 1 root root   3004190 12月 11  2016 ibus-table-wubi_1.8.2-2_all.deb
-rw-r--r-- 1 root root    126532 12月 10  2017 kde-config-fcitx_0.5.5-1_arm64.deb
-rw-r--r-- 1 root root      1956 7月  17  2018 language-pack-gnome-zh-hans_1%3a18.04+20180712_all.deb
-rw-r--r-- 1 root root   1481496 7月  17  2018 language-pack-gnome-zh-hans-base_1%3a18.04+20180712_all.deb
-rw-r--r-- 1 root root      1960 7月  17  2018 language-pack-gnome-zh-hant_1%3a18.04+20180712_all.deb
```

思考：缓存里没有明显的 OpenCV 包，需要从 apt 源下载。

#### 命令 17

```bash
command -v ffmpeg
```

返回码：0

输出：

```text
/usr/bin/ffmpeg
```

思考：可以用 `ffmpeg` 直接测试 V4L2 摄像头。

#### 命令 18

```bash
command -v gst-launch-1.0
```

返回码：0

输出：

```text
/usr/bin/gst-launch-1.0
```

思考：系统也有 GStreamer 工具，但 `ffmpeg` 已足够完成基础摄像头验证。

### 6. 用 ffmpeg 检查摄像头基础驱动链路

目的：在 OpenCV 还不可用时，先确认摄像头设备能被系统驱动读取。

#### 命令 19

```bash
find /var/cache/apt/archives -maxdepth 1 -type f -name '*opencv*' -printf '%f\n' | sort
```

返回码：0

输出：

```text

```

思考：apt 缓存中没有 OpenCV 包。

#### 命令 20

```bash
find /var/cache/apt/archives -maxdepth 1 -type f -name '*numpy*' -printf '%f\n' | sort
```

返回码：0

输出：

```text

```

思考：apt 缓存中没有 numpy 包；不过系统 Python 后续检查发现 numpy 已安装。

#### 命令 21

```bash
python3 -c "import numpy; print(numpy.__version__)"
```

返回码：0

输出：

```text
1.13.3
```

思考：OpenCV 依赖的 numpy 已存在，不需要单独处理。

#### 命令 22

```bash
ffmpeg -hide_banner -f v4l2 -list_formats all -i /dev/video0
```

返回码：1

输出：

```text
[video4linux2,v4l2 @ 0x55a8868af0] Raw       :     yuyv422 :           YUYV 4:2:2 : 640x480 352x288 320x240 176x144 160x120
/dev/video0: Immediate exit requested
```

说明：该命令用于列出格式，`ffmpeg` 在列出格式后退出，返回码 1 不代表摄像头故障。

思考：设备支持 YUYV 格式和多个分辨率，包含 640x480，说明 V4L2 查询正常。

#### 命令 23

```bash
ffmpeg -hide_banner -f v4l2 -video_size 640x480 -i /dev/video0 -frames:v 1 camera_test.jpg -y
```

返回码：0

输出：

```text
Input #0, video4linux2,v4l2, from '/dev/video0':
  Duration: N/A, start: 1777.332949, bitrate: 147456 kb/s
    Stream #0:0: Video: rawvideo (YUY2 / 0x32595559), yuyv422, 640x480, 147456 kb/s, 30 fps, 30 tbr, 1000k tbn, 1000k tbc
Stream mapping:
  Stream #0:0 -> #0:0 (rawvideo (native) -> mjpeg (native))
Press [q] to stop, [?] for help
[swscaler @ 0x559ea2be90] deprecated pixel format used, make sure you did set range correctly
Output #0, image2, to 'camera_test.jpg':
  Metadata:
    encoder         : Lavf57.83.100
    Stream #0:0: Video: mjpeg, yuvj422p(pc), 640x480, q=2-31, 200 kb/s, 30 fps, 30 tbn, 30 tbc
    Metadata:
      encoder         : Lavc57.107.100 mjpeg
    Side data:
      cpb: bitrate max/min/avg: 0/0/200000 buffer size: 0 vbv_delay: -1
frame=    1 fps=0.0 q=4.6 Lsize=N/A time=00:00:00.03 bitrate=N/A speed=0.791x    
video:19kB audio:0kB subtitle:0kB other streams:0kB global headers:0kB muxing overhead: unknown
```

思考：成功抓取一帧，证明摄像头可以被系统驱动读取。输出文件存在，尺寸为 640x480。随后通过图像查看工具人工查看，确认画面有效，不是空白图。

### 7. 本地解包 OpenCV

目的：在不能使用 `sudo apt-get install` 的情况下，让当前用户可以运行 OpenCV 脚本。

#### 命令 26

```bash
mkdir -p local_debs opencv_local
```

返回码：0

输出：

```text

```

思考：创建两个目录：`local_debs` 存放下载的 `.deb`，`opencv_local` 存放解包后的运行文件。

#### 命令 27

```bash
apt-get download python3-opencv
```

工作目录：`/home/gwb/Fang/local_debs`

返回码：0

输出：

```text
获取:1 http://ports.ubuntu.com/ubuntu-ports bionic-updates/universe arm64 python3-opencv arm64 3.2.0+dfsg-4ubuntu0.1 [474 kB]
已下载 474 kB，耗时 2秒 (229 kB/s)
```

#### 命令 28

```bash
dpkg-deb -x python3-opencv_3.2.0+dfsg-4ubuntu0.1_arm64.deb ../opencv_local
```

工作目录：`/home/gwb/Fang/local_debs`

返回码：0

输出：

```text

```

#### 命令 29

```bash
PYTHONPATH=/home/gwb/Fang/opencv_local/usr/lib/python3/dist-packages python3 -c "import cv2; print(cv2.__version__)"
```

返回码：1

输出：

```text
Traceback (most recent call last):
  File "<string>", line 1, in <module>
ImportError: libopencv_hdf.so.3.2: cannot open shared object file: No such file or directory
```

问题：只解包 `python3-opencv` 后，Python 绑定能被找到，但底层 OpenCV 共享库缺失。

思考：需要继续下载并解包 `python3-opencv` 的 OpenCV 动态库依赖。

#### 命令 30

```bash
apt-get download libopencv-calib3d3.2 libopencv-features2d3.2 libopencv-flann3.2 libopencv-highgui3.2 libopencv-imgcodecs3.2 libopencv-imgproc3.2 libopencv-ml3.2 libopencv-objdetect3.2 libopencv-photo3.2 libopencv-shape3.2 libopencv-stitching3.2 libopencv-superres3.2 libopencv-video3.2 libopencv-videoio3.2 libopencv-videostab3.2 libopencv-viz3.2 libopencv-contrib3.2 libopencv-core3.2
```

工作目录：`/home/gwb/Fang/local_debs`

返回码：0

输出：

```text
获取:1 http://ports.ubuntu.com/ubuntu-ports bionic-updates/universe arm64 libopencv-calib3d3.2 arm64 3.2.0+dfsg-4ubuntu0.1 [374 kB]
获取:2 http://ports.ubuntu.com/ubuntu-ports bionic-updates/universe arm64 libopencv-contrib3.2 arm64 3.2.0+dfsg-4ubuntu0.1 [1,293 kB]
获取:3 http://ports.ubuntu.com/ubuntu-ports bionic-updates/universe arm64 libopencv-core3.2 arm64 3.2.0+dfsg-4ubuntu0.1 [631 kB]
获取:4 http://ports.ubuntu.com/ubuntu-ports bionic-updates/universe arm64 libopencv-features2d3.2 arm64 3.2.0+dfsg-4ubuntu0.1 [188 kB]
获取:5 http://ports.ubuntu.com/ubuntu-ports bionic-updates/universe arm64 libopencv-flann3.2 arm64 3.2.0+dfsg-4ubuntu0.1 [81.8 kB]
获取:6 http://ports.ubuntu.com/ubuntu-ports bionic-updates/universe arm64 libopencv-highgui3.2 arm64 3.2.0+dfsg-4ubuntu0.1 [17.7 kB]
获取:7 http://ports.ubuntu.com/ubuntu-ports bionic-updates/universe arm64 libopencv-imgcodecs3.2 arm64 3.2.0+dfsg-4ubuntu0.1 [74.9 kB]
获取:8 http://ports.ubuntu.com/ubuntu-ports bionic-updates/universe arm64 libopencv-imgproc3.2 arm64 3.2.0+dfsg-4ubuntu0.1 [713 kB]
获取:9 http://ports.ubuntu.com/ubuntu-ports bionic-updates/universe arm64 libopencv-ml3.2 arm64 3.2.0+dfsg-4ubuntu0.1 [196 kB]
获取:10 http://ports.ubuntu.com/ubuntu-ports bionic-updates/universe arm64 libopencv-objdetect3.2 arm64 3.2.0+dfsg-4ubuntu0.1 [126 kB]
获取:11 http://ports.ubuntu.com/ubuntu-ports bionic-updates/universe arm64 libopencv-photo3.2 arm64 3.2.0+dfsg-4ubuntu0.1 [166 kB]
获取:12 http://ports.ubuntu.com/ubuntu-ports bionic-updates/universe arm64 libopencv-shape3.2 arm64 3.2.0+dfsg-4ubuntu0.1 [50.1 kB]
获取:13 http://ports.ubuntu.com/ubuntu-ports bionic-updates/universe arm64 libopencv-stitching3.2 arm64 3.2.0+dfsg-4ubuntu0.1 [148 kB]
获取:14 http://ports.ubuntu.com/ubuntu-ports bionic-updates/universe arm64 libopencv-superres3.2 arm64 3.2.0+dfsg-4ubuntu0.1 [37.9 kB]
获取:15 http://ports.ubuntu.com/ubuntu-ports bionic-updates/universe arm64 libopencv-video3.2 arm64 3.2.0+dfsg-4ubuntu0.1 [105 kB]
获取:16 http://ports.ubuntu.com/ubuntu-ports bionic-updates/universe arm64 libopencv-videoio3.2 arm64 3.2.0+dfsg-4ubuntu0.1 [72.8 kB]
获取:17 http://ports.ubuntu.com/ubuntu-ports bionic-updates/universe arm64 libopencv-videostab3.2 arm64 3.2.0+dfsg-4ubuntu0.1 [86.6 kB]
获取:18 http://ports.ubuntu.com/ubuntu-ports bionic-updates/universe arm64 libopencv-viz3.2 arm64 3.2.0+dfsg-4ubuntu0.1 [100 kB]
已下载 4,463 kB，耗时 3秒 (1,526 kB/s)
```

#### 命令 31

```bash
for f in *.deb; do dpkg-deb -x "$f" ../opencv_local; done
```

工作目录：`/home/gwb/Fang/local_debs`

返回码：0

输出：

```text

```

#### 命令 32

```bash
PYTHONPATH=/home/gwb/Fang/opencv_local/usr/lib/python3/dist-packages LD_LIBRARY_PATH=/home/gwb/Fang/opencv_local/usr/lib/aarch64-linux-gnu python3 -c "import cv2; print(cv2.__version__)"
```

返回码：1

输出：

```text
Traceback (most recent call last):
  File "<string>", line 1, in <module>
ImportError: libhdf5_serial.so.100: cannot open shared object file: No such file or directory
```

问题：OpenCV 共享库的二级依赖缺失。

思考：继续用 `ldd` 查缺失库，并根据 apt 依赖补齐。

#### 命令 33

```bash
PYTHONPATH=/home/gwb/Fang/opencv_local/usr/lib/python3/dist-packages LD_LIBRARY_PATH=/home/gwb/Fang/opencv_local/usr/lib/aarch64-linux-gnu ldd /home/gwb/Fang/opencv_local/usr/lib/python3/dist-packages/cv2.cpython-36m-aarch64-linux-gnu.so | grep 'not found'
```

返回码：0

输出：

```text
	libhdf5_serial.so.100 => not found
	libtesseract.so.4 => not found
	libtbb.so.2 => not found
	libtbb.so.2 => not found
	libgdcmMSFF.so.2.8 => not found
	libgdal.so.20 => not found
	libgdcmDSED.so.2.8 => not found
	libtbb.so.2 => not found
	libtbb.so.2 => not found
```

思考：缺失库来自 HDF5、Tesseract、TBB、GDCM、GDAL 等包。

#### 命令 34

```bash
apt-cache depends libopencv-contrib3.2 libopencv-imgcodecs3.2 libopencv-core3.2
```

返回码：0

输出：

```text
libopencv-contrib3.2
  依赖: libopencv-calib3d3.2
  依赖: libopencv-core3.2
  依赖: libopencv-features2d3.2
  依赖: libopencv-flann3.2
  依赖: libopencv-highgui3.2
  依赖: libopencv-imgcodecs3.2
  依赖: libopencv-imgproc3.2
  依赖: libopencv-ml3.2
  依赖: libopencv-objdetect3.2
  依赖: libopencv-photo3.2
  依赖: libopencv-shape3.2
  依赖: libopencv-stitching3.2
  依赖: libopencv-superres3.2
  依赖: libopencv-video3.2
  依赖: libopencv-videoio3.2
  依赖: libopencv-videostab3.2
  依赖: libopencv-viz3.2
  依赖: libc6
  依赖: libgcc1
    libgcc-s1
  依赖: libhdf5-100
  依赖: libstdc++6
  依赖: libtbb2
  依赖: libtesseract4
libopencv-imgcodecs3.2
  依赖: libopencv-imgproc3.2
  依赖: <gdal-abi-2-2-3>
    libgdal20
  依赖: libc6
  依赖: libgcc1
    libgcc-s1
  依赖: libgdal20
  依赖: libgdcm2.8
  依赖: libilmbase12
  依赖: libjpeg8
  依赖: libopencv-core3.2
  依赖: libopenexr22
  依赖: libpng16-16
  依赖: libstdc++6
  依赖: libtiff5
  依赖: libwebp6
libopencv-core3.2
  依赖: libc6
  依赖: libgcc1
    libgcc-s1
  依赖: libstdc++6
  依赖: libtbb2
  依赖: zlib1g
```

思考：根据依赖关系下载第一批二级依赖。

#### 命令 35

```bash
apt-get download libhdf5-100 libtesseract4 libtbb2 libgdal20 libgdcm2.8
```

工作目录：`/home/gwb/Fang/local_debs`

返回码：0

输出：

```text
获取:1 http://ports.ubuntu.com/ubuntu-ports bionic/universe arm64 libgdal20 arm64 2.2.3+dfsg-2 [4,668 kB]
获取:2 http://ports.ubuntu.com/ubuntu-ports bionic/universe arm64 libgdcm2.8 arm64 2.8.4-1build2 [1,669 kB]
获取:3 http://ports.ubuntu.com/ubuntu-ports bionic/universe arm64 libhdf5-100 arm64 1.10.0-patch1+docs-4 [1,010 kB]
获取:4 http://ports.ubuntu.com/ubuntu-ports bionic/universe arm64 libtbb2 arm64 2017~U7-8 [87.8 kB]
获取:5 http://ports.ubuntu.com/ubuntu-ports bionic/universe arm64 libtesseract4 arm64 4.00~git2288-10f4998a-2 [1,078 kB]
已下载 8,512 kB，耗时 4秒 (1,914 kB/s)
```

#### 命令 36

```bash
for f in *.deb; do dpkg-deb -x "$f" ../opencv_local; done
```

工作目录：`/home/gwb/Fang/local_debs`

返回码：0

输出：

```text

```

#### 命令 37

```bash
PYTHONPATH=/home/gwb/Fang/opencv_local/usr/lib/python3/dist-packages LD_LIBRARY_PATH=/home/gwb/Fang/opencv_local/usr/lib/aarch64-linux-gnu python3 -c "import cv2; print(cv2.__version__)"
```

返回码：1

输出：

```text
Traceback (most recent call last):
  File "<string>", line 1, in <module>
ImportError: libgdal.so.20: cannot open shared object file: No such file or directory
```

问题：`libgdal.so.20` 已解包，但不在当前 `LD_LIBRARY_PATH` 的目录中。

#### 命令 38

```bash
find /home/gwb/Fang/opencv_local -name 'libgdal.so*' -o -name 'libhdf5_serial.so*' -o -name 'libtesseract.so*' -o -name 'libtbb.so*' -o -name 'libgdcmMSFF.so*' | sort
```

返回码：0

输出：

```text
/home/gwb/Fang/opencv_local/usr/lib/aarch64-linux-gnu/libgdcmMSFF.so.2.8
/home/gwb/Fang/opencv_local/usr/lib/aarch64-linux-gnu/libgdcmMSFF.so.2.8.4
/home/gwb/Fang/opencv_local/usr/lib/aarch64-linux-gnu/libhdf5_serial.so.100
/home/gwb/Fang/opencv_local/usr/lib/aarch64-linux-gnu/libhdf5_serial.so.100.0.1
/home/gwb/Fang/opencv_local/usr/lib/aarch64-linux-gnu/libtbb.so.2
/home/gwb/Fang/opencv_local/usr/lib/aarch64-linux-gnu/libtesseract.so.4
/home/gwb/Fang/opencv_local/usr/lib/aarch64-linux-gnu/libtesseract.so.4.0.0
/home/gwb/Fang/opencv_local/usr/lib/libgdal.so.20
/home/gwb/Fang/opencv_local/usr/lib/libgdal.so.20.3.2
```

思考：`libgdal.so.20` 位于 `opencv_local/usr/lib`，需要把该目录也加入 `LD_LIBRARY_PATH`。

#### 命令 39

```bash
PYTHONPATH=/home/gwb/Fang/opencv_local/usr/lib/python3/dist-packages LD_LIBRARY_PATH=/home/gwb/Fang/opencv_local/usr/lib/aarch64-linux-gnu:/home/gwb/Fang/opencv_local/usr/lib python3 -c "import cv2; print(cv2.__version__)"
```

返回码：1

输出：

```text
Traceback (most recent call last):
  File "<string>", line 1, in <module>
ImportError: libsz.so.2: cannot open shared object file: No such file or directory
```

问题：继续暴露更多二级依赖缺失。

#### 命令 40

```bash
PYTHONPATH=/home/gwb/Fang/opencv_local/usr/lib/python3/dist-packages LD_LIBRARY_PATH=/home/gwb/Fang/opencv_local/usr/lib/aarch64-linux-gnu:/home/gwb/Fang/opencv_local/usr/lib ldd /home/gwb/Fang/opencv_local/usr/lib/python3/dist-packages/cv2.cpython-36m-aarch64-linux-gnu.so | grep 'not found'
```

返回码：0

输出：

```text
	libsz.so.2 => not found
	liblept.so.5 => not found
	libCharLS.so.1 => not found
	libarmadillo.so.8 => not found
	libproj.so.12 => not found
	libfreexl.so.1 => not found
	libqhull.so.7 => not found
	libgeos_c.so.1 => not found
	libepsilon.so.1 => not found
	libodbc.so.2 => not found
	libodbcinst.so.2 => not found
	libkmlbase.so.1 => not found
	libkmldom.so.1 => not found
	libkmlengine.so.1 => not found
	libxerces-c-3.2.so => not found
	libnetcdf.so.13 => not found
	libmfhdfalt.so.0 => not found
	libdfalt.so.0 => not found
	libogdi.so.3.2 => not found
	libgeotiff.so.2 => not found
	libpq.so.5 => not found
	libdapclient.so.6 => not found
	libdap.so.25 => not found
	libspatialite.so.7 => not found
	libfyba.so.0 => not found
```

思考：这些主要来自 `libgdal20`、`libgdcm2.8`、`libhdf5-100`、`libtesseract4` 的依赖，继续按包名补齐。

#### 命令 41

```bash
apt-cache depends libgdal20 libgdcm2.8 libhdf5-100 libtesseract4
```

返回码：0

输出：

```text
libgdal20
  依赖: gdal-data
  依赖: libarmadillo8
  依赖: libc6
  依赖: libcurl3-gnutls
  依赖: libdap25
  依赖: libdapclient6v5
  依赖: libepsilon1
  依赖: libexpat1
  依赖: libfreexl1
  依赖: libfyba0
  依赖: libgcc1
    libgcc-s1
  依赖: libgeos-c1v5
  依赖: libgeotiff2
  依赖: libgif7
  依赖: libhdf4-0-alt
  依赖: libhdf5-100
  依赖: libjpeg8
  依赖: libjson-c3
  依赖: libkmlbase1
  依赖: libkmldom1
  依赖: libkmlengine1
  依赖: libmysqlclient20
  依赖: libnetcdf13
  依赖: libodbc1
  依赖: libogdi3.2
  依赖: libopenjp2-7
  依赖: libpcre3
  依赖: libpng16-16
  依赖: libpoppler73
  依赖: libpq5
  依赖: libproj12
  依赖: libqhull7
  依赖: libspatialite7
  依赖: libstdc++6
  依赖: libtiff5
  依赖: libwebp6
  依赖: libxerces-c3.2
  依赖: libxml2
  依赖: odbcinst1debian2
  依赖: zlib1g
  破坏: <libgdal1h>
  推荐: proj-bin
libgdcm2.8
  依赖: libc6
  依赖: libcharls1
  依赖: libexpat1
  依赖: libgcc1
    libgcc-s1
  依赖: libjson-c3
  依赖: libopenjp2-7
  依赖: libsocket++1
  依赖: libssl1.1
  依赖: libstdc++6
  依赖: libuuid1
  依赖: zlib1g
libhdf5-100
  依赖: libc6
  依赖: libgcc1
    libgcc-s1
  依赖: libgfortran4
  依赖: libsz2
  依赖: zlib1g
libtesseract4
  依赖: libc6
  依赖: libgcc1
    libgcc-s1
  依赖: libgomp1
  依赖: liblept5
  依赖: libstdc++6
  破坏: <ocropus>
  破坏: <tesseract-afr>
  破坏: <tesseract-ara>
  破坏: <tesseract-aze>
  破坏: <tesseract-bel>
  破坏: <tesseract-ben>
  破坏: <tesseract-bul>
  破坏: <tesseract-cat>
  破坏: <tesseract-ces>
  破坏: <tesseract-chi-sim>
  破坏: <tesseract-chi-tra>
  破坏: <tesseract-chr>
  破坏: <tesseract-dan>
  破坏: <tesseract-deu>
  破坏: <tesseract-deu-frak>
  破坏: <tesseract-ell>
  破坏: <tesseract-eng>
  破坏: <tesseract-enm>
  破坏: <tesseract-epo>
  破坏: <tesseract-equ>
  破坏: <tesseract-est>
  破坏: <tesseract-eus>
  破坏: <tesseract-fin>
  破坏: <tesseract-fra>
  破坏: <tesseract-frk>
  破坏: <tesseract-frm>
  破坏: <tesseract-glg>
  破坏: <tesseract-grc>
  破坏: <tesseract-heb>
  破坏: <tesseract-hin>
  破坏: <tesseract-hrv>
  破坏: <tesseract-hun>
  破坏: <tesseract-ind>
  破坏: <tesseract-isl>
  破坏: <tesseract-ita>
  破坏: <tesseract-ita-old>
  破坏: <tesseract-jpn>
  破坏: <tesseract-kan>
  破坏: <tesseract-kor>
  破坏: <tesseract-lav>
  破坏: <tesseract-lit>
  破坏: <tesseract-mal>
  破坏: <tesseract-mkd>
  破坏: <tesseract-mlt>
  破坏: <tesseract-msa>
  破坏: <tesseract-nld>
  破坏: <tesseract-nor>
  破坏: tesseract-ocr
  破坏: tesseract-ocr-amh
  破坏: tesseract-ocr-asm
  破坏: tesseract-ocr-aze-cyrl
  破坏: tesseract-ocr-bod
  破坏: tesseract-ocr-bos
  破坏: tesseract-ocr-ceb
  破坏: tesseract-ocr-cym
  破坏: tesseract-ocr-dzo
  破坏: tesseract-ocr-fas
  破坏: tesseract-ocr-gle
  破坏: <tesseract-ocr-gle-uncial>
  破坏: <tesseract-ocr-grc>
  破坏: tesseract-ocr-guj
  破坏: tesseract-ocr-hat
  破坏: tesseract-ocr-iku
  破坏: tesseract-ocr-jav
  破坏: tesseract-ocr-kat
  破坏: tesseract-ocr-kat-old
  破坏: tesseract-ocr-kaz
  破坏: tesseract-ocr-khm
  破坏: tesseract-ocr-kir
  破坏: <tesseract-ocr-kur>
  破坏: tesseract-ocr-lao
  破坏: tesseract-ocr-lat
  破坏: tesseract-ocr-mar
  破坏: tesseract-ocr-mya
  破坏: tesseract-ocr-nep
  破坏: tesseract-ocr-ori
  破坏: tesseract-ocr-pan
  破坏: tesseract-ocr-pus
  破坏: tesseract-ocr-san
  破坏: tesseract-ocr-sin
  破坏: tesseract-ocr-srp-latn
  破坏: tesseract-ocr-syr
  破坏: tesseract-ocr-tgk
  破坏: tesseract-ocr-tir
  破坏: tesseract-ocr-uig
  破坏: tesseract-ocr-urd
  破坏: tesseract-ocr-uzb
  破坏: tesseract-ocr-uzb-cyril
  破坏: tesseract-ocr-yid
  破坏: <tesseract-pol>
  破坏: <tesseract-por>
  破坏: <tesseract-ron>
  破坏: <tesseract-rus>
  破坏: <tesseract-slk>
  破坏: <tesseract-slk-frak>
  破坏: <tesseract-slv>
  破坏: <tesseract-spa>
  破坏: <tesseract-spa-old>
  破坏: <tesseract-sqi>
  破坏: <tesseract-srp>
  破坏: <tesseract-swa>
  破坏: <tesseract-swe>
  破坏: <tesseract-tam>
  破坏: <tesseract-tel>
  破坏: <tesseract-tgl>
  破坏: <tesseract-tha>
  破坏: <tesseract-tur>
  破坏: <tesseract-ukr>
  破坏: <tesseract-vie>
  替换: tesseract-ocr
```

说明：实际终端输出中该段很长；关键是定位缺失库对应的软件包。

#### 命令 42

```bash
apt-get download libsz2 liblept5 libcharls1 libarmadillo8 libproj12 libfreexl1 libqhull7 libgeos-c1v5 libepsilon1 libodbc1 odbcinst1debian2 libkmlbase1 libkmldom1 libkmlengine1 libxerces-c3.2 libnetcdf13 libhdf4-0-alt libogdi3.2 libgeotiff2 libpq5 libdapclient6v5 libdap25 libspatialite7 libfyba0
```

工作目录：`/home/gwb/Fang/local_debs`

返回码：0

输出：

```text
获取:1 http://ports.ubuntu.com/ubuntu-ports bionic/main arm64 libodbc1 arm64 2.3.4-1.1ubuntu3 [148 kB]
获取:2 http://ports.ubuntu.com/ubuntu-ports bionic/main arm64 odbcinst1debian2 arm64 2.3.4-1.1ubuntu3 [37.6 kB]
获取:3 http://ports.ubuntu.com/ubuntu-ports bionic/universe arm64 libarmadillo8 arm64 1:8.400.0+dfsg-2 [81.6 kB]
获取:4 http://ports.ubuntu.com/ubuntu-ports bionic/universe arm64 libcharls1 arm64 1.1.0+dfsg-2 [50.6 kB]
获取:5 http://ports.ubuntu.com/ubuntu-ports bionic/universe arm64 libdap25 arm64 3.19.1-2build1 [400 kB]
获取:6 http://ports.ubuntu.com/ubuntu-ports bionic/universe arm64 libdapclient6v5 arm64 3.19.1-2build1 [82.5 kB]
获取:7 http://ports.ubuntu.com/ubuntu-ports bionic/universe arm64 libepsilon1 arm64 0.9.2+dfsg-2 [35.7 kB]
获取:8 http://ports.ubuntu.com/ubuntu-ports bionic/universe arm64 libfreexl1 arm64 1.0.5-1 [31.6 kB]
获取:9 http://ports.ubuntu.com/ubuntu-ports bionic/universe arm64 libfyba0 arm64 4.1.1-3 [101 kB]
获取:10 http://ports.ubuntu.com/ubuntu-ports bionic/universe arm64 libgeos-c1v5 arm64 3.6.2-1build2 [52.9 kB]
获取:11 http://ports.ubuntu.com/ubuntu-ports bionic/universe arm64 libgeotiff2 arm64 1.4.2-2build1 [51.1 kB]
获取:12 http://ports.ubuntu.com/ubuntu-ports bionic/universe arm64 libhdf4-0-alt arm64 4.2.13-2 [219 kB]
获取:13 http://ports.ubuntu.com/ubuntu-ports bionic/universe arm64 libkmlbase1 arm64 1.3.0-5 [43.9 kB]
获取:14 http://ports.ubuntu.com/ubuntu-ports bionic/universe arm64 libkmldom1 arm64 1.3.0-5 [161 kB]
获取:15 http://ports.ubuntu.com/ubuntu-ports bionic/universe arm64 libkmlengine1 arm64 1.3.0-5 [71.4 kB]
获取:16 http://ports.ubuntu.com/ubuntu-ports bionic/universe arm64 liblept5 arm64 1.75.3-3 [775 kB]
获取:17 http://ports.ubuntu.com/ubuntu-ports bionic/universe arm64 libnetcdf13 arm64 1:4.6.0-2build1 [282 kB]
获取:18 http://ports.ubuntu.com/ubuntu-ports bionic/universe arm64 libogdi3.2 arm64 3.2.0+ds-2 [177 kB]
获取:19 http://ports.ubuntu.com/ubuntu-ports bionic/universe arm64 libproj12 arm64 4.9.3-2 [131 kB]
获取:20 http://ports.ubuntu.com/ubuntu-ports bionic/universe arm64 libqhull7 arm64 2015.2-4 [131 kB]
获取:21 http://ports.ubuntu.com/ubuntu-ports bionic/universe arm64 libspatialite7 arm64 4.3.0a-5build1 [1,114 kB]
获取:22 http://ports.ubuntu.com/ubuntu-ports bionic/universe arm64 libsz2 arm64 0.3.2-2 [4,926 B]
获取:23 http://ports.ubuntu.com/ubuntu-ports bionic/universe arm64 libxerces-c3.2 arm64 3.2.0+debian-2 [792 kB]
获取:24 http://ports.ubuntu.com/ubuntu-ports bionic-updates/main arm64 libpq5 arm64 10.23-0ubuntu0.18.04.2 [93.2 kB]
已下载 5,067 kB，耗时 4秒 (1,295 kB/s)
```

#### 命令 43

```bash
for f in *.deb; do dpkg-deb -x "$f" ../opencv_local; done
```

工作目录：`/home/gwb/Fang/local_debs`

返回码：0

输出：

```text

```

#### 命令 44

```bash
PYTHONPATH=/home/gwb/Fang/opencv_local/usr/lib/python3/dist-packages LD_LIBRARY_PATH=/home/gwb/Fang/opencv_local/usr/lib/aarch64-linux-gnu:/home/gwb/Fang/opencv_local/usr/lib python3 -c "import cv2; print(cv2.__version__)"
```

返回：进程收到信号 11，段错误。

输出：

```text
execution error: Sandbox(Signal(11))
```

问题：导入 `cv2` 时段错误。

思考：段错误通常意味着动态库链仍不完整，或加载到了不完整依赖。继续使用 `ldd` 查剩余缺失库。

#### 命令 45

```bash
PYTHONPATH=/home/gwb/Fang/opencv_local/usr/lib/python3/dist-packages LD_LIBRARY_PATH=/home/gwb/Fang/opencv_local/usr/lib/aarch64-linux-gnu:/home/gwb/Fang/opencv_local/usr/lib ldd /home/gwb/Fang/opencv_local/usr/lib/python3/dist-packages/cv2.cpython-36m-aarch64-linux-gnu.so | grep 'not found'
```

返回码：0

输出：

```text
	libaec.so.0 => not found
	libarpack.so.2 => not found
	libsuperlu.so.5 => not found
	libgeos-3.6.2.so => not found
	libminizip.so.1 => not found
	liburiparser.so.1 => not found
```

思考：还有 6 个依赖缺失；继续下载对应包。

#### 命令 46

```bash
apt-get download libaec0 libarpack2 libsuperlu5 libgeos-3.6.2 libminizip1 liburiparser1
```

工作目录：`/home/gwb/Fang/local_debs`

返回码：0

输出：

```text
获取:1 http://ports.ubuntu.com/ubuntu-ports bionic/universe arm64 libaec0 arm64 0.3.2-2 [16.7 kB]
获取:2 http://ports.ubuntu.com/ubuntu-ports bionic/universe arm64 libarpack2 arm64 3.5.0+real-2 [71.0 kB]
获取:3 http://ports.ubuntu.com/ubuntu-ports bionic/universe arm64 libgeos-3.6.2 arm64 3.6.2-1build2 [460 kB]
获取:4 http://ports.ubuntu.com/ubuntu-ports bionic/universe arm64 libminizip1 arm64 1.1-8build1 [17.5 kB]
获取:5 http://ports.ubuntu.com/ubuntu-ports bionic-updates/universe arm64 liburiparser1 arm64 0.8.4-1+deb9u2ubuntu0.1 [30.4 kB]
获取:6 http://ports.ubuntu.com/ubuntu-ports bionic/universe arm64 libsuperlu5 arm64 5.2.1+dfsg1-3 [119 kB]
已下载 715 kB，耗时 2秒 (454 kB/s)
```

#### 命令 47

```bash
for f in *.deb; do dpkg-deb -x "$f" ../opencv_local; done
```

工作目录：`/home/gwb/Fang/local_debs`

返回码：0

输出：

```text

```

#### 命令 48

```bash
PYTHONPATH=/home/gwb/Fang/opencv_local/usr/lib/python3/dist-packages LD_LIBRARY_PATH=/home/gwb/Fang/opencv_local/usr/lib/aarch64-linux-gnu:/home/gwb/Fang/opencv_local/usr/lib ldd /home/gwb/Fang/opencv_local/usr/lib/python3/dist-packages/cv2.cpython-36m-aarch64-linux-gnu.so | grep 'not found'
```

返回码：1

输出：

```text

```

说明：`grep 'not found'` 无匹配时返回 1，表示没有缺失库。

#### 命令 49

```bash
PYTHONPATH=/home/gwb/Fang/opencv_local/usr/lib/python3/dist-packages LD_LIBRARY_PATH=/home/gwb/Fang/opencv_local/usr/lib/aarch64-linux-gnu:/home/gwb/Fang/opencv_local/usr/lib python3 -c "import cv2; print(cv2.__version__)"
```

返回码：0

输出：

```text
3.2.0
```

思考：OpenCV 已能在当前用户环境下运行。

### 8. 编写 OpenCV 摄像头实时预览脚本

目的：创建一个简单 Python 脚本，实时显示摄像头画面，同时适配本地解包的 OpenCV。

操作：使用文件编辑工具新增 `/home/gwb/Fang/camera_viewer.py`。

主要设计：

1. 启动时自动检测旁边是否存在 `opencv_local`。
2. 如果存在，重新执行自身并设置 `PYTHONPATH` 和 `LD_LIBRARY_PATH`，使 Python 能导入本地解包的 OpenCV。
3. 默认打开摄像头编号 `0`。
4. 默认请求 640x480。
5. 实时窗口中按 `q` 或 `Esc` 退出。
6. 增加 `--test-only` 参数，便于无交互测试摄像头是否能取帧。

#### 命令 50

```bash
chmod +x camera_viewer.py
```

返回码：0

输出：

```text

```

思考：让脚本可以直接用 `./camera_viewer.py` 运行。

### 9. 第一次运行脚本并修复 OpenCV 版本兼容问题

目的：用脚本自身执行 OpenCV 取帧测试。

#### 命令 51

```bash
./camera_viewer.py --test-only --save-frame opencv_test.jpg
```

返回码：1

输出：

```text
Traceback (most recent call last):
  File "./camera_viewer.py", line 124, in <module>
    main()
  File "./camera_viewer.py", line 91, in main
    cap = open_camera(device, args.width, args.height)
  File "./camera_viewer.py", line 53, in open_camera
    cap = cv2.VideoCapture(device, backend)
TypeError: VideoCapture() takes at most 1 argument (2 given)
```

问题：OpenCV 3.2 的 Python 绑定不支持较新版 OpenCV 的 `VideoCapture(device, backend)` 双参数写法。

思考：为了兼容当前系统的 OpenCV 3.2，需要在代码中捕获 `TypeError`，失败时退回 `cv2.VideoCapture(device)` 单参数写法。

操作：修改 `camera_viewer.py` 中打开摄像头逻辑，加入旧版兼容 fallback。

#### 命令 52

```bash
./camera_viewer.py --test-only --save-frame opencv_test.jpg
```

返回码：0

输出：

```text
OpenCV 3.2.0 opened camera 0 and read frame 640x480.
```

思考：OpenCV 脚本已能打开摄像头并读取一帧。

#### 命令 53

```bash
file opencv_test.jpg
```

返回码：0

输出：

```text
opencv_test.jpg: JPEG image data, JFIF standard 1.01, aspect ratio, density 1x1, segment length 16, baseline, precision 8, 640x480, frames 3
```

#### 命令 54

```bash
ls -l opencv_test.jpg
```

返回码：0

输出：

```text
-rw-rw-r-- 1 gwb gwb 76390 6月   1 11:02 opencv_test.jpg
```

思考：OpenCV 保存的测试图存在，尺寸 640x480。随后通过图像查看工具人工查看，确认图像有效。

### 10. 测试实时窗口

目的：确认脚本不仅能取帧，还能通过 OpenCV 窗口实时显示。

#### 命令 55

```bash
timeout 5s ./camera_viewer.py
```

返回码：124

输出：

```text

```

问题：没有看到脚本提示输出。

思考：脚本在被 `timeout` 终止前，标准输出可能因为缓冲未刷新。实时窗口路径本身没有报错，但为了记录更清楚，需要让提示输出 `flush=True`。

操作：修改 `camera_viewer.py`，给运行提示和测试输出加 `flush=True`。

#### 命令 56

```bash
./camera_viewer.py --test-only --save-frame opencv_test.jpg
```

返回码：0

输出：

```text
OpenCV 3.2.0 opened camera 0 and read frame 640x480.
```

思考：修改输出刷新后，再次确认取帧功能仍正常。

#### 命令 57

```bash
timeout 5s ./camera_viewer.py
```

返回码：124

输出：

```text
Showing camera 0. Press q or Esc in the window to exit.
```

说明：返回码 124 是 `timeout 5s` 到时自动结束，不是脚本错误。

思考：脚本已进入实时预览循环并输出提示，说明窗口路径可启动。

### 11. 编写桌面使用说明

目的：按要求在桌面留下 Markdown 使用说明。

操作：新增 `/home/gwb/Desktop/camera_viewer_usage.md`。

内容包括：

1. 脚本路径。
2. 实时预览运行命令。
3. 退出方式：按 `q` 或 `Esc`。
4. 一次性测试命令。
5. 常用参数说明。
6. 本次摄像头检查结论。

### 12. 编写本记录文件

目的：按要求把任务过程、终端输入命令、终端返回、问题、思考和采取措施整理为 Markdown。

操作：新增 `/home/gwb/Fang/RECORD.md`。

## 最终文件清单

- `/home/gwb/Fang/camera_viewer.py`：OpenCV 摄像头实时预览脚本。
- `/home/gwb/Fang/RECORD.md`：完整任务过程记录。
- `/home/gwb/Desktop/camera_viewer_usage.md`：桌面使用说明。
- `/home/gwb/Fang/opencv_local/`：免 root 解包的 OpenCV 运行依赖。
- `/home/gwb/Fang/local_debs/`：下载的 `.deb` 包。
- `/home/gwb/Fang/camera_test.jpg`：ffmpeg 摄像头抓图测试文件。
- `/home/gwb/Fang/opencv_test.jpg`：OpenCV 摄像头抓图测试文件。

## 脚本使用方法

实时预览：

```bash
cd /home/gwb/Fang
./camera_viewer.py
```

一次性取帧测试：

```bash
cd /home/gwb/Fang
./camera_viewer.py --test-only --save-frame opencv_test.jpg
```

退出实时窗口：在窗口中按 `q` 或 `Esc`。
