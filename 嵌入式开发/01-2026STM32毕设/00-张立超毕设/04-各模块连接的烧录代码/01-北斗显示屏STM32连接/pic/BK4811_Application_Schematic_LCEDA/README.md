# BK4811 Application Schematic

本目录是根据用户给出的 `Figure 6 Application Schematic of BK4811` 图片重建的可编辑原理图工程。

当前版本 `Rev 1.2` 优先修正连线质量：所有导线改为水平/垂直正交线，并尽量使用可见直接连接，减少悬空标签造成的断联观感。

## 打开方式

- 嘉立创 EDA 标准版：使用“文件 / 导入 / KiCad”导入 `BK4811_Application_Schematic_LCEDA.zip`。
- 嘉立创 EDA 专业版：开始页或顶部菜单选择 KiCad 导入，选择 `BK4811_Application_Schematic_LCEDA.zip`。
- KiCad：直接打开 `BK4811_Application_Schematic.pro`。

## 重要限制

- 本工程只还原原理图连接和图片中可见的元件值。
- 图片没有提供封装、厂家料号、PCB 布局、走线阻抗和射频匹配实测参数，因此所有封装字段留空。
- 导入嘉立创 EDA 后，请先核对 BK4811 数据手册、封装和实际模块接口，再继续 PCB 设计。
- L2、L3、L5、C5、C16、C17、C18 按图片标注为 `NL`，表示不贴/预留。

## 文件说明

- `BK4811_Application_Schematic.pro`：KiCad 5 工程文件。
- `BK4811_Application_Schematic.sch`：原理图文件。
- `BK4811_App.lib` / `BK4811_App.dcm`：本工程自定义符号库。
