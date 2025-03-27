Anaconda 虚拟环境
Anaconda 提供了一个称为 environments 的强大功能，它允许创建具有特定配置的隔离 Python 环境。

Anaconda 中的环境是虚拟空间，可以在其中安装包并管理依赖项，这与系统的 Python 安装分开。这在处理具有不同包依赖项的不同项目时特别有用。

可以使用以下命令创建环境：

conda create --name myenv

这将创建一个名为“myenv”的新环境。

创建虚拟环境后，可能需要根据需要激活、停用或删除它们。以下是在 Anaconda 中管理环境的基本命令：

激活环境：

conda activate myenv

停用当前环境：

conda deactivate

删除环境：

conda remove --name myenv --all

Anaconda Navigator
Anaconda Navigator 是 Anaconda 附带的图形用户界面 （GUI），它为管理包、环境和应用程序提供了一个易于使用的界面。

Anaconda Navigator 是一个桌面应用程序，提供了一种方便的方式来启动和管理应用程序，例如 Jupyter Notebook、Spyder 等。方便的进行环境、包管理。



Anaconda 内置多种工具，可以方便的进行安装。比较经典的 Jupyter Notebook、Spyder 可以直接打开使用。



图形界面也可以进行环境管理、安装包管理，更加直观简单。

Anaconda 命令行界面
除了 Anaconda Navigator 提供的图形界面外，Anaconda 还提供了一个强大的命令行界面 （CLI），用于管理 Python 环境和包。

常见的 Anaconda 命令

更新 Anaconda

要将 Anaconda 安装更新到最新版本，请运行以下命令：

conda update anaconda

安装包

若要使用 conda 安装包，请使用以下命令：

conda install package_name

删除包

conda remove package_name

更新包

conda update package_name

搜索包

conda search package_name