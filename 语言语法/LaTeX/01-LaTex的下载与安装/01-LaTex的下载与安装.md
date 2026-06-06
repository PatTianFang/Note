---
title: 01-LaTex的下载与安装
date: 2025-05-02 15:50:49
categories: LaTex
tags:
  - Latex
---
文章主要参考CSDN Nicolecocol的文章《【LaTex】LaTex的下载与安装（2024新手小白超详细、超简洁 Windows系统）》^[https://blog.csdn.net/Nicolecocol/article/details/136968456]

### 下载Texlive和TexStudio

可以从镜像站下载，速度较快

Texlive
[Index of /CTAN/systems/texlive/Images/ | 清华大学开源软件镜像站 | Tsinghua Open Source Mirror](https://mirrors.tuna.tsinghua.edu.cn/CTAN/systems/texlive/Images/)
![](Blog/source/_posts/01-LaTex的下载与安装/Pasted%20image%2020250502155417.png)

TexStudio
[Index of /github-release/texstudio-org/texstudio/LatestRelease/ | 清华大学开源软件镜像站 | Tsinghua Open Source Mirror](https://mirrors.tuna.tsinghua.edu.cn/github-release/texstudio-org/texstudio/LatestRelease/)

![](Blog/source/_posts/01-LaTex的下载与安装/Pasted%20image%2020250502162529.png)

**先安装texlive再安装TexStudio**


## 安装Texlive

下载完成后打开

![](Blog/source/_posts/01-LaTex的下载与安装/Pasted%20image%2020250502162347.png)

使用管理员身份打开其中的`install-tl-windows.bat`文件

![](Blog/source/_posts/01-LaTex的下载与安装/Pasted%20image%2020250502162736.png)

短暂出现下面界面后进入安装界面

![](Blog/source/_posts/01-LaTex的下载与安装/Pasted%20image%2020250502163007.png)

![](Blog/source/_posts/01-LaTex的下载与安装/Pasted%20image%2020250502163354.png)

**如果不使用管理员身份会卡在第一个界面**

根据个人需要修改安装路径，可以点击`Advanced`进一步设置

![](image/Pasted%20image%2020250502163633.png)

设置完成后点击安装

![](Blog/source/_posts/01-LaTex的下载与安装/Pasted%20image%2020250502163655.png)

**不要按Abort否则会退出**

有事会卡在`running package-specific postactions`这一步

![](Blog/source/_posts/01-LaTex的下载与安装/Pasted%20image%2020250502184355.png)

打开对应文件夹下的`install-tl.log`文件

![](Blog/source/_posts/01-LaTex的下载与安装/Pasted%20image%2020250502184456.png)

发现实际已经输出安装成功的提示

![](Blog/source/_posts/01-LaTex的下载与安装/Pasted%20image%2020250502184543.png)

此时点击关闭即可

![](Blog/source/_posts/01-LaTex的下载与安装/Pasted%20image%2020250502184705.png)

此时打开CMD，输入`latex -v`

![](Blog/source/_posts/01-LaTex的下载与安装/Pasted%20image%2020250502184843.png)

如图显示如上图，则安装成功。

## 安装TexStudio

双击打开安装包

![](Blog/source/_posts/01-LaTex的下载与安装/Pasted%20image%2020250502185012.png)

下一步

![](Blog/source/_posts/01-LaTex的下载与安装/Pasted%20image%2020250502185050.png)

设置安装目录

![](Blog/source/_posts/01-LaTex的下载与安装/Pasted%20image%2020250502185132.png)

安装完成，点击关闭

![](Blog/source/_posts/01-LaTex的下载与安装/Pasted%20image%2020250502185149.png)

![](Blog/source/_posts/01-LaTex的下载与安装/Pasted%20image%2020250502185219.png)


## 测试LaTex是否正常安装

打开TeXstudio

![](Blog/source/_posts/01-LaTex的下载与安装/Pasted%20image%2020250502185323.png)

![](Blog/source/_posts/01-LaTex的下载与安装/Pasted%20image%2020250502185426.png)

点击**文件**>**新建**

![](Blog/source/_posts/01-LaTex的下载与安装/Pasted%20image%2020250502185516.png)

**输入以下测试代码**

```LaTex
\documentclass{article}
 
\begin{document}
 
Hello, world!
 
\end{document}
```

![](Blog/source/_posts/01-LaTex的下载与安装/Pasted%20image%2020250502185630.png)

点击运行，如果右侧出现Hello, world!说明正常安装完成

![](Blog/source/_posts/01-LaTex的下载与安装/Pasted%20image%2020250502185705.png)

