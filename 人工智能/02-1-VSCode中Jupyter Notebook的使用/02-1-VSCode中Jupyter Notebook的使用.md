# 02-1-VSCode中Jupyter Notebook的使用

可以参考VS Code官方给出的文档[在 Visual Studio Code 中使用 Jupyter Notebook_Vscode中文网](https://vscode.github.net.cn/docs/datascience/jupyter-notebooks)

# 一、切换模式

存在两种模式，分别为**命令模式**和**编辑模式**

## 命令模式

当处于命令模式时，单元格左侧将出现一个实心垂直条，表示当前选中的单元格。该单元可以进行操作并接受键盘命令。

![Alt](Pasted%20image%2020250502085848.png)

![Alt](Pasted%20image%2020250502085740.png)

![Alt](Pasted%20image%2020250502085812.png)

在键盘上，按`Esc`键可进入命令模式

## 编辑模式

在编辑模式下，单元格编辑器周围有一个实心垂直条由边框连接起来。单元格的内容（代码或 Markdown）可以修改。

![Alt](Pasted%20image%2020250502090507.png)

![Alt](Pasted%20image%2020250502090521.png)

在键盘上，按`Enter`键可进入编辑模式

## 模式的切换

要切换模式，可以使用键盘或鼠标。

- 使用键盘时，按`Enter`键可进入编辑模式，按`Esc`键可进入命令模式。
- 使用鼠标时，单击单元格红框以外的区域进入命令模式，单元格红框以内的区域进入编辑模式。

![Alt](Pasted%20image%2020250502090843.png)

![Alt](Pasted%20image%2020250502090853.png)

# 二、运行代码


## 运行当前单元格

可以点击单元左侧的**“运行”图标运行代码单元，输出将直接显示在代码单元下方。**

![Alt](Pasted%20image%2020250502091406.png)

有输出和无输出的对比

![Alt](Pasted%20image%2020250502091456.png)

使用键盘运行当前单元格时，
- 使用`Ctrl`+`Enter`运行当前单元格。
- 使用`Shift`+`Enter`运行当前单元格并前进到下一个单元格。

`Ctrl`+`Enter`

![Alt](Pasted%20image%2020250502092138.png)

`Shift`+`Enter`

![Alt](Pasted%20image%2020250502092154.png)

## 按行运行

按行运行能够按照一行一行的顺序对单元格内的代码进行运行和输出

运行前

![Alt](Pasted%20image%2020250502092316.png)

运行后

![Alt](Pasted%20image%2020250502092345.png)

![Alt](Pasted%20image%2020250502092435.png)

![Alt](Pasted%20image%2020250502092505.png)

## 运行多个单元格

可以通过选择**运行全部**、**运行上方全部**或**运行下方全部**来运行多个单元格

![Alt](Pasted%20image%2020250502152733.png)


# 三、基础操作

## 保存文件

在键盘按下`Ctrl`+`S`或点击文件>保存

![Alt](Pasted%20image%2020250502153156.png)

![Alt](Pasted%20image%2020250502153135.png)

## 导出Jupyter笔记本

选择界面上方的导出可以导出文件

![Alt](Pasted%20image%2020250502154545.png)

选择要导出的格式即可

![Alt](Pasted%20image%2020250502154637.png)

**当存在LaTeX公式时，导出 PDF必须安装 TeX**

![Alt](Pasted%20image%2020250502154817.png)

可以进入[Get LaTeX - Mac OS, Windows, Linux](https://www.latex-project.org/get/)下载

LaTex的安装可以参考[01-LaTex的下载与安装](Note/语言语法/LaTeX/01-LaTex的下载与安装.md)

安装完成后能够正常导出

![Alt](file-20250503082109530.png)

![Alt](file-20250503082216209.png)