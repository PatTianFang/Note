- 基础概述
	- 课程介绍
	- Linux系统介绍及安装
	- Linux系统基础操作
	- C++/Python极简基础
	- 安装ROS系统


# C++/Python极简基础

## 安装编译器和解析器

通过命令行形式对代码进行编译

先将编译器作为软件进行安装

在终端输入

```Linux
sudo apt-get install g++
```

和

```Linux
sudo apt-get install python
```

在桌面创建名为`c++_for.cpp`的文件

![[Pasted image 20250329192837.png]]

编译格式为

```Linux
g++ 编译的代码的文件名 -o 输出的可执行文件的名字
```

输入

```Linux
g++ c++_for.cpp -o c++_for
```

将`c++_for.cpp`文件进行编译

![[Pasted image 20250329193328.png]]

按照运行文件的方式

```Linux
./编译完的文件名
```

输入

```Linux
./c++_for
```

![[Pasted image 20250329195535.png]]

在桌面创建名为`python_for.python`的文件

![[Pasted image 20250329194744.png]]

输入

```Linux
python python_for.py
```

![[Pasted image 20250329195852.png]]

#

#Linux #ROS #rm #指令 


