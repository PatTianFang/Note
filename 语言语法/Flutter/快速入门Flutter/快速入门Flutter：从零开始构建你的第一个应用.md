# 快速入门Flutter：从零开始构建你的第一个应用

文章主要参考CSDN孤影过客的文章《快速入门Flutter：从零开始构建你的第一个应用》^[https://blog.csdn.net/gygkhd/article/details/139646716]

# 一、环境准备

## 1-安装 Flutter SDK

在官网[Flutter - Build apps for any screen](https://flutter.dev/)下载Flutter SDK。

![Alt](file-20250729080900666.png)

可以参考文档[开发 Windows 桌面应用 | Flutter 中文文档 - Flutter 中文开发者网站 - Flutter](https://docs.flutter.cn/get-started/install/windows/desktop)

![Alt](file-20250729081105457.png)

安装 [Visual Studio Code](https://code.visualstudio.com/docs/setup/windows) 以及 [Flutter extension for VS Code](https://marketplace.visualstudio.com/items?itemName=Dart-Code.flutter)

打开VS Code，下载Flutter拓展

![Alt](file-20250729081326723.png)

## 1-1-安装 Flutter

1. 打开 **命令面板 (Command Palette)**，按下快捷键 Control + Shift + P。

![Alt](file-20250729081558544.png)

2. 在 **命令面板 (Command Palette)** 中输入 `flutter`。

![Alt](file-20250729081620184.png)

3. 选择 **Flutter: New Project**。

![Alt](file-20250729081650482.png)

4. 显示没有安装 Flutter SDK, 单击 **Download SDK**。当对话框 Select Folder for Flutter SDK 显示时，选择你想要安装 Flutter 的位置。

![Alt](file-20250729081846070.png)

5. 等待下载完成

![Alt](file-20250729081926326.png)

**注意**

不要将 Flutter 安装到以下情况的目录或路径中：

- 路径包含特殊字符或空格。
    
- 路径需要较高的权限。

## 1-2-解决Resolving dependencies...问题

发现卡在Resolving dependencies...这一步

```
Running pub upgrade...

Resolving dependencies...

Got socket error trying to find package file at https://pub.dev.
```

打开环境变量

![Alt](file-20250729083104049.png)

在用户变量下添加下面两个变量名：

```
PUB_HOSTED_URL https://pub.flutter-io.cn
```

```
FLUTTER_STORAGE_BASE_URL https://storage.flutter-io.cn
```

![Alt](file-20250729092009444.png)

成功后会显示通知：

```
The Flutter SDK was added to your PATH
```

![Alt](file-20250729095500703.png)

## 1-3-检查安装是否成功

打开PowerShell。

![Alt](file-20250729095647145.png)

运行以下指令。

```
flutter doctor
```

![Alt](file-20250729105120549.png)

安装成功

# 二、第一个应用

## 2-1-创建应用

打开你想要创建Flutter应用的位置

![Alt](file-20250729105530746.png)

打开终端

![Alt](file-20250729105553562.png)

在终端中，运行以下命令创建一个新的 Flutter 项目：

```
flutter create my_first_app
```

创建完毕

![Alt](file-20250729110128819.png)

![Alt](file-20250729110153956.png)

进入项目目录：

```bash
cd my_first_app
```

## 2-2-运行应用

运行以下命令启动应用：

```bash
flutter run
```
![Alt](file-20250729110642500.png)

选择设备后，弹出测试程序

![Alt](file-20250729110714236.png)

在创建的项目中，目录结构如下：
```
my_first_app/
├── android/
├── ios/
├── lib/
│   └── main.dart
├── test/
├── pubspec.yaml
└── README.md
```
其中：

`lib/`：主要的 Dart 代码目录，`main.dart`是应用的入口文件。

`pubspec.yaml`：项目的配置文件，用于管理依赖包。

# 三、编译应用

## 3-1-生成apk文件

在对应目录下打开终端，输入

```
flutter build apk
```

![Alt](file-20250729122208682.png)

但总是卡在

```
Running Gradle task 'assembleRelease'...  
```

这一步，网上的各种教程都是说修改`/android/build.gradle`、`flutter/packages/flutter_tools/gradle/flutter.gradle`这些`.gradle`结尾的文件，但实际打开目录时发现这些文件后面多了`kts`后缀。

![Alt](file-20250729131621144.png)

这种情况通常发生在较新的Android项目中，因为Google正在推动从Groovy转向Kotlin DSL作为构建脚本语言。新版Android Studio的gradle文件已经默认使用Kotlin脚本(KTS)。

.gradle使用Groovy语言编写，而.gradle.kts使用Kotlin脚本。

所以应该修改`settings.gradle.kts`文件和`build.gradle.kts`文件。

对于`settings.gradle.kts`文件

![Alt](file-20250729131918996.png)

在此处插入国内镜像源

![Alt](file-20250729132012299.png)

```
// Google 专属仓库（Android 项目必备）
maven(url = "https://maven.aliyun.com/repository/google")

// 公共中央仓库（代理 Maven Central）
maven(url = "https://maven.aliyun.com/repository/public")

// Gradle 插件仓库
maven(url = "https://maven.aliyun.com/repository/gradle-plugin")

// JCenter 仓库（兼容旧项目）
maven(url = "https://maven.aliyun.com/repository/jcenter")

// 通用仓库（覆盖 Google + Central）
maven(url = "https://repo.huaweicloud.com/repository/maven")

// 公共仓库（代理 Central）
maven(url = "https://mirrors.cloud.tencent.com/nexus/repository/maven-public")

// 清华大学镜像
maven(url = "https://mirrors.tuna.tsinghua.edu.cn/maven")

// 中国科学技术大学镜像
maven(url = "https://mirrors.ustc.edu.cn/maven")
```

完整配置如下

```
pluginManagement {
    val flutterSdkPath = run {
        val properties = java.util.Properties()
        file("local.properties").inputStream().use { properties.load(it) }
        val flutterSdkPath = properties.getProperty("flutter.sdk")
        require(flutterSdkPath != null) { "flutter.sdk not set in local.properties" }
        flutterSdkPath
    }

    includeBuild("$flutterSdkPath/packages/flutter_tools/gradle")

    repositories {

	// Google 专属仓库（Android 项目必备）
	maven(url = "https://maven.aliyun.com/repository/google")

	// 公共中央仓库（代理 Maven Central）
	maven(url = "https://maven.aliyun.com/repository/public")

	// Gradle 插件仓库
	maven(url = "https://maven.aliyun.com/repository/gradle-plugin")

	// JCenter 仓库（兼容旧项目）
	maven(url = "https://maven.aliyun.com/repository/jcenter")

	// 通用仓库（覆盖 Google + Central）
	maven(url = "https://repo.huaweicloud.com/repository/maven")
	
	// 公共仓库（代理 Central）
	maven(url = "https://mirrors.cloud.tencent.com/nexus/repository/maven-public")

	// 清华大学镜像
	maven(url = "https://mirrors.tuna.tsinghua.edu.cn/maven")

	// 中国科学技术大学镜像
	maven(url = "https://mirrors.ustc.edu.cn/maven")

        google()
        mavenCentral()
        gradlePluginPortal()
    }
}

plugins {
    id("dev.flutter.flutter-plugin-loader") version "1.0.0"
    id("com.android.application") version "8.7.3" apply false
    id("org.jetbrains.kotlin.android") version "2.1.0" apply false
}

include(":app")

```

对于`build.gradle.kts`文件

![Alt](file-20250729142010643.png)

添加

```
// Google 专属仓库（Android 项目必备）
maven(url = "https://maven.aliyun.com/repository/google")

// 公共中央仓库（代理 Maven Central）
maven(url = "https://maven.aliyun.com/repository/public")

// Gradle 插件仓库
maven(url = "https://maven.aliyun.com/repository/gradle-plugin")

// JCenter 仓库（兼容旧项目）
maven(url = "https://maven.aliyun.com/repository/jcenter")

// 通用仓库（覆盖 Google + Central）
maven(url = "https://repo.huaweicloud.com/repository/maven")

// 公共仓库（代理 Central）
maven(url = "https://mirrors.cloud.tencent.com/nexus/repository/maven-public")

// 清华大学镜像
maven(url = "https://mirrors.tuna.tsinghua.edu.cn/maven")

// 中国科学技术大学镜像
maven(url = "https://mirrors.ustc.edu.cn/maven")
```

完整配置如下

```
allprojects {
    repositories {
	// Google 专属仓库（Android 项目必备）
	maven(url = "https://maven.aliyun.com/repository/google")

	// 公共中央仓库（代理 Maven Central）
	maven(url = "https://maven.aliyun.com/repository/public")

	// Gradle 插件仓库
	maven(url = "https://maven.aliyun.com/repository/gradle-plugin")

	// JCenter 仓库（兼容旧项目）
	maven(url = "https://maven.aliyun.com/repository/jcenter")

	// 通用仓库（覆盖 Google + Central）
	maven(url = "https://repo.huaweicloud.com/repository/maven")
	
	// 公共仓库（代理 Central）
	maven(url = "https://mirrors.cloud.tencent.com/nexus/repository/maven-public")

	// 清华大学镜像
	maven(url = "https://mirrors.tuna.tsinghua.edu.cn/maven")

	// 中国科学技术大学镜像
	maven(url = "https://mirrors.ustc.edu.cn/maven")

        google()
        mavenCentral()
    }
}

val newBuildDir: Directory = rootProject.layout.buildDirectory.dir("../../build").get()
rootProject.layout.buildDirectory.value(newBuildDir)

subprojects {
    val newSubprojectBuildDir: Directory = newBuildDir.dir(project.name)
    project.layout.buildDirectory.value(newSubprojectBuildDir)
}
subprojects {
    project.evaluationDependsOn(":app")
}

tasks.register<Delete>("clean") {
    delete(rootProject.layout.buildDirectory)
}

```

![Alt](file-20250729142028835.png)

添加镜像源后速度恢复正常。

![Alt](file-20250729131000344.png)

编译apk成功

![Alt](file-20250729142645613.png)

输出路径在`项目名\build\app\outputs\apk\release`内

![Alt](file-20250729142739490.png)

## 3-2-生成exe文件

项目目录下，在命令行输入

```
flutter build windows
```

![Alt](file-20250729143527833.png)

生成exe成功

![Alt](file-20250729143807058.png)