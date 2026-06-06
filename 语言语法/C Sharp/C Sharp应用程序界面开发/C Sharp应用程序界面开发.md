# C Sharp应用程序界面开发

文章主要参考CSDN DXB2021的文章《C#应用程序界面开发基础——窗体控制（1）——Form窗体（删除事件部分，没看懂）》^[https://blog.csdn.net/DXB2021/article/details/125558570]

# 项目的创建

打开Visual Studio（以Visual Studio 2022为例）

![Alt](file-20250510104246043.png)

依次点击`文件`>`新建`>`项目`

使用快捷键`Ctrl`+`Shift`+`N`可以快速地打开“新建项目”对话框

![Alt](file-20250510104346189.png)

![Alt](file-20250510104548248.png)

在拉下菜单中选择`C#`、`Windows`、`桌面`中，在列表框选择`Windows窗体应用（.NET Framework）`，点击`下一步`。

![Alt](file-20250510105214683.png)

PS：VS下的`Windows 窗体应用(.NET Framework)`与`Windows 窗体应用`之间的区别可以参考下面的网站

1. [VS下的 Windows 窗体应用(.NET Framework) 与 Windows 窗体应用之间的区别_vs 窗体应用和窗体应用库选哪个-CSDN博客](https://blog.csdn.net/qq_45040187/article/details/137717793)
2. [(求教windows窗体应用和windows窗体应用(.net framework)的区别? - 知乎](https://www.zhihu.com/question/451320245)

设置完毕后点击`创建`。

![Alt](file-20250510105340463.png)

进入界面

![Alt](file-20250510105458898.png)

# 窗体初始代码

在解决方案资源管理器，双击Program.cs文件，会跳转到Windows控制台应用界面，在“编辑”窗口中是一段自动生成的WinForm程序。

![Alt](file-20250510105547505.png)

代码如下：

```CSharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using System.Windows.Forms;

namespace WindowsFormsApp1
{
    internal static class Program
    {
        /// <summary>
        /// 应用程序的主入口点。
        /// </summary>
        [STAThread]
        static void Main()
        {
            Application.EnableVisualStyles();
            Application.SetCompatibleTextRenderingDefault(false);
            Application.Run(new Form1());
        }
    }
}

```

再单击“启动”按键，会弹出一个空白的窗体。

![Alt](file-20250510105656278.png)


右击“窗体”后，选择“查看代码”命令，就会跳转到Form1.cs文件

![Alt](file-20250510105811703.png)

![Alt](file-20250510105842834.png)

代码如下： 

```cs
using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Data;
using System.Drawing;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Windows.Forms;

namespace WindowsFormsApp1
{
    public partial class Form1 : Form
    {
        public Form1()
        {
            InitializeComponent();
        }
    }
}
```

# 窗体的添加与删除

## 添加窗体

右击项目名称WindowsFormsApp1，在弹出的快捷菜单中选择“添加”-“Windows窗体”或者“添加”-“新建项”命令

![Alt](file-20250510110048814.png)

![Alt](file-20250510110104317.png)

最后在“添加新项”框中选择“窗体（Windows窗体）”

![Alt](file-20250510110138688.png)

添加窗体成功

![Alt](file-20250510110214969.png)

## 删除窗体

删除窗体，只需要在解决方案管理器中，选中要删除的窗体名称，右击，在弹出的快捷菜单选择“删除”命令即可

![Alt](file-20250510110309636.png)

# 窗体的属性

## 1-“属性”面板

打开“属性”面板有三种方法：

### 01-右键命令

![Alt](file-20250510110537190.png)

### 02-“视图”命令

依次点击`视图`>`属性窗口`

![Alt](file-20250510110623693.png)

### 03-快捷键

同时按下`Alt`+`Enter`

![Alt](file-20250510110743416.png)

## 2-C# WinForm窗体基础属性

### 01-窗口样式中的属性值

|         属性值         |      说明       |
| :-----------------: | :-----------: |
|        Icon         | 更改图标样式（左上角图标） |
|  MaximizeBox:true;  |  显示右上角最大化按钮   |
|  MinimizeBox:true;  |  显示右上角最小化按钮   |
|   ShowInco:true;    |   显示左上角小图标    |
| ShowInTaskbar:ture; |   窗体显示在任务栏    |
|    TopMost:ture;    |    窗口置顶显示     |
|    Opacity:100%     |    整个窗口透明度    |
### 02-布局中的属性值

|             属性值             |        说明         |
| :-------------------------: | :---------------: |
|   AutoScroll:true/false;    | 如果控件超出窗口是否自动显示滚动条 |
|    AutoSize:true/false;     |  窗口的范围是否会超出控件的大小  |
|      MaximumSize:0,0;       |   窗口可以拖曳的最大的大小    |
|      MinimumSize:0,0;       |   窗口可以拖曳的最小的大小    |
|        Size:300,300;        |    窗口打开时默认的大小     |
| StartPosition:centerScreen; |  窗口打开时默认桌面位置，居中   |
|   WindowState:Maximized;    |     默认打开窗口最大化     |

### 03-外观的属性值

|             属性值              |        说明         |
| :--------------------------: | :---------------: |
|         Font:宋体,9pt；         | 可以修改字体大小，字体越大控件越大 |
|            Text；             |       输入文本        |
|          TextAlign；          |       文字位置        |
| FormBorderStyle:FixedSingle； |     窗口不可拖曳大小      |
|    FormBorderStyle:None;     |      隐藏窗口的边框      |
| DropDownStyle:DropDownList；  |    窗让下拉框无法输入文本    |
## 3-设置窗体属性

窗体的图标是系统默认的图标

![Alt](file-20250510111641284.png)

更改图标，在“属性”面板中，选择Icon性格

![Alt](file-20250510111711089.png)

窗体的颜色和背景，通过BackgroundImage属性进行设置。选择“属性”面板中的BackgroundImage属性。

![Alt](file-20250510111748897.png)

# 窗体的常用事件

所谓事件，就是指要发生的事情，可以简单地理解为用户的操作，它是由对象引发的。窗体的所有事件，都可以在“属性”面板中进行查看。

## 01-添加事件

为窗体添加一件事件，只要在事件面板里选择要添加的事件

![Alt](file-20250510111920303.png)

以Load为例，在Load后面的空格里双击，相应的事件将会自动生成。

![Alt](file-20250510112038593.png)

![Alt](file-20250510111939732.png)

运行

![Alt](file-20250510112128383.png)

## 02-删除事件

直接删除对应的代码即可

```Cs
private void Form1_Load(object sender, EventArgs e)
{
	MessageBox.Show("窗体加载完成");
}
```

![Alt](file-20250510112323826.png)


## 03-窗体的显示与隐藏

窗体标识符`.Show()`

窗体标识符`.Hide()`