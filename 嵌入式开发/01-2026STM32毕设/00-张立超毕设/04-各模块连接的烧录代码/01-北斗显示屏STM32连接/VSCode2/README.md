# STM32F103C8T6 北斗 BDS 经纬度 OLED 显示实验

本工程基于 STM32F103C8T6 小系统板，实现北斗/GNSS 模块 NMEA 数据接收、RMC 定位语句解析、经纬度格式转换、北京时间显示，并通过 7 针 SPI OLED 显示结果。

当前固件已经按实际硬件接线重构为模块化结构：

- `main.c` 只负责 MCU 初始化、USART2 中断接收启动、主循环调度。
- `bds_nmea.c/h` 负责 NMEA 字节流捕获、RMC 帧解析、经纬度和时间格式化。
- `bds_display.c/h` 负责 OLED 页面布局和业务显示内容。
- `oled_spi.c/h` 负责 SSD1306 类 7 针 SPI OLED 的底层 GPIO 模拟 SPI 驱动。

底部固定显示作者字样：`作者：张立超`。

## 1. 实验目标

本实验的目标不是单纯点亮 OLED，而是完成一个完整的小型嵌入式项目闭环：

1. STM32F103C8T6 通过 USART2 接收北斗/GPS 模块输出的 NMEA 数据。
2. 从 NMEA 数据中筛选 RMC 语句。
3. 解析 RMC 的 UTC 时间、定位状态、纬度、经度和方向字段。
4. 将 NMEA 原始坐标格式转换成普通人更容易理解的十进制度数。
5. 将 UTC 时间换算为北京时间。
6. 通过 7 针 SPI OLED 显示定位状态、时间、经纬度和作者信息。
7. 保持代码结构清晰，让硬件驱动、协议解析和显示业务互不混杂。

## 2. 硬件材料

实验使用以下硬件：

| 硬件 | 用途 |
| --- | --- |
| STM32F103C8T6 小系统板 | 主控 MCU |
| ST-Link 下载器 | 下载、调试、供电辅助 |
| 7 针 SPI OLED 液晶 | 显示 BDS 状态、时间和坐标 |
| 北斗/GPS 模块 | 输出 NMEA 定位数据 |
| 北斗天线套装 | 增强卫星信号接收 |
| 面包板、杜邦线 | 搭建实验电路 |
| USB 数据线 | 给小系统板或下载器供电 |

## 3. 硬件接线

### 3.1 OLED 接线

OLED 使用 4 线 SPI 方式，软件通过 GPIO 模拟 SPI，不依赖 STM32 的硬件 SPI 外设。

| STM32F103C8T6 小系统板 | OLED 引脚 | 说明 |
| --- | --- | --- |
| G | GND | OLED 地 |
| 3.3 | VDD | OLED 3.3V 电源 |
| B8 | D0 / SCK | SPI 时钟 |
| B9 | D1 / SDA | SPI 数据 |
| R / RESET | RES | OLED 复位，接小系统板复位脚 |
| B12 | DC | 命令/数据选择 |
| B13 | CS | 片选 |

注意：原实验文档最后两行可能因为表格排版显示为 `DC -> B12`、`CS -> B13`，本工程按实际电路含义理解为 `B12 -> DC`、`B13 -> CS`。

### 3.2 北斗 GPS 模块接线

GPS 模块接 USART2。

| STM32F103C8T6 小系统板 | 北斗 GPS 模块 | 说明 |
| --- | --- | --- |
| 3.3 | VCC | 模块供电 |
| G | GND | 共地 |
| A2 / PA2 / USART2_TX | RX | MCU 发给模块，当前主要用于保持标准串口连接 |
| A3 / PA3 / USART2_RX | TX | 模块输出 NMEA，MCU 接收 |

串口参数：

```text
baudrate: 9600
data:     8 bit
parity:   none
stop:     1 bit
flow:     none
```

## 4. 软件架构

重构后工程按职责拆分，而不是把所有逻辑都写在 `main.c` 中。

```text
Core/
  Inc/
    bds_display.h      显示层接口
    bds_nmea.h         NMEA/RMC 解析接口
    oled_spi.h         OLED 底层驱动接口
    main.h             CubeMX/HAL 主头文件
  Src/
    main.c             系统初始化、主循环、UART 回调胶水
    bds_display.c      OLED 页面布局和业务显示
    bds_nmea.c         NMEA 捕获、RMC 解析、坐标/时间格式化
    oled_spi.c         GPIO 模拟 SPI + OLED 初始化 + 点阵输出
    stm32f1xx_hal_msp.c USART2 GPIO/NVIC 初始化
    stm32f1xx_it.c     USART2 中断入口
```

### 4.1 `main.c`

`main.c` 只做三类事情：

1. 初始化 HAL、系统时钟、GPIO 和 USART2。
2. 初始化显示和 NMEA 捕获。
3. 在主循环中取出完整 RMC 帧，交给解析模块和显示模块。

核心流程：

```c
BDS_Display_Init();
BDS_NMEA_ResetCapture();
HAL_UART_Receive_IT(&huart2, &bds_rx_byte, 1);

while (1)
{
  if (BDS_NMEA_TakeFrame(frame, sizeof(frame)) != 0U)
  {
    if (BDS_NMEA_ParseFrame(frame, &fix) != 0U)
    {
      BDS_Display_ShowFix(&fix);
    }
    else
    {
      BDS_Display_ShowParseError();
    }
  }
}
```

USART2 中断回调只做非常小的事情：把收到的字节送给 NMEA 捕获器，然后继续启动下一字节接收。

### 4.2 `bds_nmea.c/h`

这个模块负责协议和数据格式。

职责：

- 从串口字节流中捕获 `$...\\r\\n` 格式的 NMEA 帧。
- 只保留 RMC 语句，避免显示层处理大量无关 NMEA 语句。
- 支持以下 RMC talker：
  - `$GPRMC`
  - `$GNRMC`
  - `$BDRMC`
- 解析字段：
  - UTC 时间
  - 定位状态 `A` / `V`
  - 纬度
  - 南北纬标志
  - 经度
  - 东西经标志
- 将坐标转换成十进制度数。
- 将 UTC 时间转换成北京时间。

### 4.3 `bds_display.c/h`

这个模块只关心“显示什么”，不关心 UART 如何接收，也不关心 OLED SPI 时序。

页面布局：

| OLED 行 | 内容 |
| --- | --- |
| 第 0 行 | `BDS FIX` / `BDS NO FIX` / `BDS WAITING` |
| 第 1 行 | `TIME:HH:MM:SS`，北京时间 |
| 第 2 行 | `LAT:xx.xxxxxxN/S` 或 `LAT:--` |
| 第 3 行 | `LON:xxx.xxxxxxE/W` 或 `LON:--` |
| 第 6-7 行 | `作者：张立超`，16x16 中文点阵 |

### 4.4 `oled_spi.c/h`

OLED 模块按 SSD1306 类控制器初始化，使用 GPIO 模拟 SPI。

引脚定义：

```c
#define OLED_SCK_GPIO_Port GPIOB
#define OLED_SCK_Pin       GPIO_PIN_8

#define OLED_SDA_GPIO_Port GPIOB
#define OLED_SDA_Pin       GPIO_PIN_9

#define OLED_DC_GPIO_Port  GPIOB
#define OLED_DC_Pin        GPIO_PIN_12

#define OLED_CS_GPIO_Port  GPIOB
#define OLED_CS_Pin        GPIO_PIN_13
```

为什么使用 GPIO 模拟 SPI：

- 当前工程由 CubeMX 生成，原始 `.ioc` 未配置 SPI 外设。
- 用户给出的 OLED 接线固定在 PB8/PB9/PB12/PB13，不是默认 SPI1 引脚。
- 模拟 SPI 对 OLED 这种低速显示足够稳定。
- 避免为了一个显示屏重配 CubeMX 生成大量外设代码。

## 5. 数据格式说明

### 5.1 RMC 语句

RMC 是 NMEA 中常用的推荐最小定位信息语句，典型格式：

```text
$GNRMC,hhmmss.sss,A,ddmm.mmmm,N,dddmm.mmmm,E,...
```

关键字段：

| 字段 | 含义 |
| --- | --- |
| `hhmmss.sss` | UTC 时间 |
| `A` / `V` | 定位有效 / 定位无效 |
| `ddmm.mmmm` | 纬度，度分格式 |
| `N` / `S` | 北纬 / 南纬 |
| `dddmm.mmmm` | 经度，度分格式 |
| `E` / `W` | 东经 / 西经 |

### 5.2 为什么原来显示 `3000.xxxx`

NMEA 的经纬度不是十进制度数，而是“度 + 分”的混合格式。

纬度格式：

```text
ddmm.mmmm
```

例如：

```text
3000.1234
```

含义不是 `3000.1234` 度，而是：

```text
30 度 00.1234 分
```

十进制度数换算公式：

```text
decimal_degree = degree + minute / 60
```

经度格式类似，但度数部分是 3 位：

```text
dddmm.mmmm
```

例如：

```text
12030.0000 = 120 度 30.0000 分 = 120.500000 度
```

当前工程在 `bds_nmea.c` 中使用整数定点算法转换为百万分之一度，避免依赖嵌入式 `printf` 的浮点格式化。

### 5.3 时间显示

RMC 输出的是 UTC 时间。本项目显示北京时间：

```text
Beijing time = UTC + 8 hours
```

只处理时分秒，不处理日期跨天显示。也就是说 23:30 UTC 会显示为 07:30:00，但不额外显示日期已经加一天。

## 6. 构建和烧录

当前工程使用 CMake + Ninja。

### 6.1 VS Code 构建

在 VS Code 中可以使用：

```text
CMake: Configure
CMake: Build
```

或者使用本工程 preset：

```powershell
cube-cmake --preset Debug
cube-cmake --build --preset Debug
```

### 6.2 命令行烧录

通过 STM32CubeProgrammer CLI 和 ST-Link 烧录：

```powershell
STM32_Programmer_CLI.exe -c port=SWD -w build\Debug\VSCode.elf -v -rst
```

成功标志：

```text
Download verified successfully
MCU Reset
Software reset is performed
```

## 7. 已解决的问题

### 7.1 VS Code 插件找不到构建工具

现象：

```text
One or more build tools have not been found
```

原因：

- 全局 `stm32-for-vscode.armToolchainPath` 曾指向错误目录。
- 工作区 `.vscode/settings.json` 曾把 `stm32-for-vscode.*Path` 覆盖为 `false`。
- CMake/Ninja 构建环境缺少 ST bundle 中的 Ninja 和 GNU Arm 工具链 PATH。

处理：

- 修正 ARM GCC 工具链路径。
- 修正工作区 `stm32-for-vscode` 三个工具路径。
- 补齐 CMake 环境中的 `cube-cmake`、`ninja`、`gnu-tools-for-stm32` 路径。

### 7.2 屏幕不亮

最初代码使用 LCD1602 并口驱动，但实际硬件是 7 针 SPI OLED。

原因：

- LCD1602 和 SPI OLED 是完全不同的显示设备。
- LCD1602 使用 RS/EN/D4-D7 并口控制。
- 当前 OLED 使用 SCK/SDA/DC/CS/RES 的 SPI 控制。

处理：

- 删除 LCD1602 模块。
- 新增 `oled_spi.c/h`。
- 按 PB8/PB9/PB12/PB13 实际接线实现 GPIO 模拟 SPI。

### 7.3 显示 `GPS NO FIX`

现象：

OLED 能亮，但显示无定位。

说明：

- 串口已经收到 RMC 数据。
- RMC 状态字段为 `V`，表示模块尚未定位成功。

处理：

- 文案改为 `BDS NO FIX`。
- 支持 `$BDRMC`，避免北斗模块输出北斗 talker 时被忽略。
- 保留 `BDS NO FIX` 状态，让用户能区分“串口没数据”和“有数据但未定位”。

硬件/环境建议：

- 天线接好并放到窗边或室外。
- 冷启动可能需要几分钟到十几分钟。
- 确认模块供电稳定。
- 确认 PA3 接 GPS TX。

### 7.4 经纬度显示为 `3000.xxxx`

原因：

直接显示了 NMEA 原始 `ddmm.mmmm` 字段。

处理：

- 增加坐标转换。
- 显示为 `LAT:xx.xxxxxxN/S` 和 `LON:xxx.xxxxxxE/W`。

## 8. 当前显示效果

未定位时：

```text
BDS NO FIX
TIME:HH:MM:SS
LAT:--
LON:--


作者：张立超
```

定位成功时：

```text
BDS FIX
TIME:HH:MM:SS
LAT:30.xxxxxxN
LON:120.xxxxxxE


作者：张立超
```

## 9. 设计取舍

### 9.1 不使用浮点格式化

STM32 裸机工程常使用 newlib-nano，默认不一定启用浮点 `printf`。如果直接写：

```c
snprintf(line, sizeof(line), "%.6f", value);
```

可能导致输出异常或增加固件体积。

本工程使用整数定点：

```text
度数 * 1000000
```

最后用整数拆成：

```text
整数部分.小数部分
```

这样更稳定，也更适合小型 MCU。

### 9.2 不在中断里解析整帧

USART 中断只做一件事：收字节并追加到 NMEA 捕获器。

完整解析放在主循环中做，原因：

- 中断执行越短越稳定。
- 避免在中断中调用复杂字符串函数。
- 避免显示刷新阻塞串口接收。

### 9.3 显示层和协议层解耦

`bds_nmea` 不依赖 OLED。

`bds_display` 不关心 USART。

这样以后如果换成串口屏、LCD、I2C OLED，只需要替换显示层；如果换解析 GGA/GLL/VTG，也只需要扩展协议层。

## 10. 常见排查

### OLED 没显示

优先检查：

1. VDD 是否接 3.3V。
2. GND 是否共地。
3. PB8 是否接 D0/SCK。
4. PB9 是否接 D1/SDA。
5. PB12 是否接 DC。
6. PB13 是否接 CS。
7. RES 是否接小系统板 R/RESET。
8. OLED 是否是 SSD1306 兼容控制器；如果是 SH1106，可能需要调整列偏移。

### 一直 `BDS WAITING`

说明没有收到可识别的 RMC 帧。

检查：

1. GPS TX 是否接 PA3。
2. GPS 模块波特率是否为 9600。
3. GPS 模块是否正在输出 NMEA。
4. 模块输出是否包含 `$GPRMC` / `$GNRMC` / `$BDRMC`。

### 一直 `BDS NO FIX`

说明已经收到 RMC 帧，但状态为无定位。

检查：

1. 天线是否接好。
2. 是否放在室外或靠窗。
3. 是否等待足够长时间。
4. GPS 模块供电是否稳定。

### 经纬度为空

通常是 RMC 状态仍为 `V`，模块没有给出有效经纬度。

### 时间不对

本工程显示的是 RMC UTC + 8 小时，不显示日期。如果 UTC 跨天，日期不会显示。

## 11. 后续可扩展方向

可以继续扩展：

1. 支持 GGA 语句，显示卫星数量和海拔。
2. 支持 HDOP，判断定位质量。
3. 支持 SH1106 OLED 的列偏移配置。
4. 增加 UART 日志输出，便于串口调试。
5. 增加状态页，显示是否收到 RMC、最后一帧 talker、定位状态和原始 UTC。
6. 将 OLED 字体拆成独立字体模块。
7. 将 `.ioc` 重新配置为 USART2 + OLED SPI 相关 GPIO，减少手写 MSP 的维护成本。

## 12. 当前验证记录

重构后已完成构建和烧录验证：

```text
FLASH: 14940 B / 64 KB
RAM:   2264 B / 20 KB
Download verified successfully
MCU Reset
Software reset is performed
```

这说明当前固件已经成功写入 STM32F103C8T6，并通过 STM32CubeProgrammer 校验。
