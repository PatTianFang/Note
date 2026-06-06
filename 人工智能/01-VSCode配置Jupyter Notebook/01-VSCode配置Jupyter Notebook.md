---
title: 01-VSCode配置Jupyter Notebook
date: 2025-05-01 17:01:26
categories: AI
tags:
  - VSCode
  - JupyterNotebook
  - AI
---
>Jupyter Notebook是基于网页的用于交互计算的应用程序。其可被应用于全过程计算：开发、文档编写、运行代码和展示结果。——[Jupyter Notebook官方介绍](https://jupyter-notebook.readthedocs.io/en/stable/notebook.html)

Jupyter Notebook文档是保存为后缀名为`.ipynb`的`JSON`格式文件，不仅便于版本控制，也方便与他人共享，文档可以导出为：HTML、LaTeX、PDF等格式。

# 一、VS Code 配置Jupyter

## 1-安装扩展

打开VS Code

![](Note/人工智能/image/Pasted%20image%2020250501160042.png)

安装jupyter扩展

![](Note/人工智能/image/Pasted%20image%2020250501160214.png)

在使用前你需要确保你的python环境以及安装了Jupyter内核

```CMD
conda install jupyter
```

或者

```CMD
pip install jupyter
```

## 2-创建Jupyter笔记本

### 方法一

按下 `Ctrl + Shift + P` 打开命令面板，输入 `Jupyter: Create New Blank Notebook`来创建一个新的 Jupyter 笔记本

![](Pasted%20image%2020250501161102%201.png)

### 方法二

点击VS Code上方的搜索框

![](Pasted%20image%2020250501161556%201.png)

点击`显示并运行命令`

![](Pasted%20image%2020250501161650%201.png)

输入 `Jupyter: Create New Blank Notebook`来创建一个新的 Jupyter 笔记本

![](Pasted%20image%2020250501162904%201.png)

### 方法三

直接新建文件

![](Pasted%20image%2020250501163546%201.png)

将文件后缀名设置为`ipynb`

![](Pasted%20image%2020250501163647%201.png)
### 创建完成

创建或者打开现有的 `.ipynb` 文件后显示

![](Pasted%20image%2020250501163127%201.png)

## 3-选择和配置内核

点击右上角的内核选择器选择内核

![](Pasted%20image%2020250501165055%201.png)

选择想要使用的Python环境

![](Pasted%20image%2020250501165305%201.png)

**注意：对应环境中要安装Jupyter，否则会报错**

![](Pasted%20image%2020250501165529%201.png)

### Python环境没有Jupyter的解决方法

这时可以使用conda进入到对应的环境，安装Jupyter

使用命令激活对应环境

```conda
conda activate UI
```

![](Pasted%20image%2020250501165819%201.png)

运行以下命令来安装 Jupyter

```conda
pip install jupyter
```

![](Pasted%20image%2020250501165915%201.png)

安装完毕

![](Pasted%20image%2020250501170040%201.png)

再次运行代码，发现能够正常运行

![](Pasted%20image%2020250501170120%201.png)

#### 只安装ipykernel内核（更快）

![](Pasted%20image%2020250501170542%201.png)

也可以在激活环境后直接输入只安装内核

```
pip install ipykernel
```

![](Pasted%20image%2020250501170434%201.png)

![](Pasted%20image%2020250501170504%201.png)

代码正常输出

![](Pasted%20image%2020250501170610%201.png)



---
#VSCode #JupyterNotebook  #AI 