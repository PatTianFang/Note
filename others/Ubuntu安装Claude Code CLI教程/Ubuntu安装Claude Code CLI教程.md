# Ubuntu安装Claude Code CLI教程

按照下面的步骤操作，即可完成相同流程。

## 操作步骤

1）在桌面上右键，点击从终端中打开

![步骤 1](images/step_001_click.png)

2）输入
```
sudo apt update && sudo apt upgrade -y
```
然后按下回车键，更新系统环境

![步骤 2](images/step_004.png)

3）然后输入你的密码，按下回车键。

![步骤 3](images/step_005.png)

4）在当前界面中向上滚动。

![步骤 4](images/step_020.png)

5）输入代码安装nvm，然后按下回车键。
```
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
```

![步骤 5](images/step_025_click.png)

6）输入指令，重新加载 shell 配置，然后按下回车键。
```
source ~/.bashrc
```

![步骤 6](images/step_027.png)

7）输入下面指令，安装Node.js（会自动配置好权限），然后按下回车键。
```
nvm install node
```

![步骤 7](images/step_029.png)

8）右键点击当前界面中的目标位置，然后按下回车键。

![步骤 8](images/step_030.png)

9）输入命令，全局安装claude code CLI

![步骤 9](images/step_031.png)

10）点击当前界面中的目标位置，然后输入“claude”，按下回车键。出现图片表明安装成功。

![步骤 10](images/step_032.png)

11）在终端中输入命令，按下回车键
```
cd ~/.claude
```
输入下面命令
```
nano settings.json
```

![步骤 11](images/step_036.png)

![步骤 11-2](images/cb8a87336fbe4b696a0504b2d01aef66_2.png)

12）打开 settings.json，添加以下配置：
```
{
"apiKey": "sk-ant-your-api-key-here",
"apiBaseUrl": "https://api.anthropic.com/v1"
}
```

![步骤 12](images/step_037.png)

13）再次在终端中输入claude，如下显示表面安装配置成功

![步骤 13](images/step_040.png)

![步骤 14](images/step_041.png)

