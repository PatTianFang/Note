# 03-2-1-手动安装Gradle

本小结与[03-2-2-javax.net.ssl.SSLHandshakeException：PKIX path building failed问题的解决](Note/QT/03-2-2-javax.net.ssl.SSLHandshakeException：PKIX%20path%20building%20failed问题的解决.md)相同

## 下载Gradle

访问 Gradle 官方下载页面：https://gradle.org/releases/

![Alt](Pasted%20image%2020250413132618.png)

或者国内的镜像网站：https://mirrors.cloud.tencent.com/gradle/

![Alt](Pasted%20image%2020250413132602.png)

选择适合你需求的 Gradle 版本，下载

![Alt](Pasted%20image%2020250413132655.png)

![Alt](Pasted%20image%2020250413132729.png)

## 手动添加Gradle

进入到自己工程目录下的wrapper文件夹

![Alt](Pasted%20image%2020250413132837.png)

![Alt](Pasted%20image%2020250413132848.png)

![Alt](Pasted%20image%2020250413132930.png)

![Alt](Pasted%20image%2020250413132945.png)

![Alt](Pasted%20image%2020250413132959.png)

![Alt](Pasted%20image%2020250413133012.png)

将刚刚下载的压缩包移动到该文件夹中

![Alt](Pasted%20image%2020250413133059.png)

编辑gradle-wrapper文件

![Alt](Pasted%20image%2020250413133155.png)

修改为下载文件名称（缺少的文件）

![Alt](Pasted%20image%2020250413133351.png)

然后再次运行

![Alt](Pasted%20image%2020250413133411.png)

正常编译

![Alt](Pasted%20image%2020250413133454.png)

输出的APK文件在之前文件夹中build目录下的output文件夹中

![Alt](Pasted%20image%2020250413133817.png)

![Alt](Pasted%20image%2020250413133858.png)

![Alt](Pasted%20image%2020250413133914.png)




---
#QT #Android #Gradle