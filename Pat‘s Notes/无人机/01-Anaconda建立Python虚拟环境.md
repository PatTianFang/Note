文章主要参考CSDN风吹过得夏天的文章《使用 Anaconda 创建 Python 虚拟环境》^[https://blog.csdn.net/u011385476/article/details/105277426]

Anaconda创建Python虚拟环境主要有两种方法，分别是使用CMD创建虚拟环境和使用Anaconda客户端创建虚拟环境。

# 一、使用CMD创建虚拟环境

## 1-创建虚拟环境

#### 打开CMD
同时按下WIN键和R键打开运行
![[Pasted image 20250315133717.png]]
输入CMD点击确定，进入CMD
![[Pasted image 20250315133813.png]]
![[Pasted image 20250315133825.png]]


#### 创建虚拟环境
创建Python虚拟环境的命令是：
```conda
conda create -n your_env_name   # your_env_name 为你虚拟环境名
```
比如说创建一个名字是text的虚拟环境
```conda
conda create -n text
```
![[Pasted image 20250315133908.png]]
输入命令后，再输入一个y进行确认
![[Pasted image 20250315133958.png]]
名叫text的Python虚拟环境就创建完毕了

#### 创建时指定版本
在创建新的虚拟环境的时候，可以指定Python的版本
```conda
conda create -n text python=X.x
```
比如说要指定Python的版本为3.8，可以输入
```conda
conda create -n text python=3.8
```

## 2-查看虚拟环境

需要查看虚拟环境时，在CMD中输入
```conda
conda env list
```
即可查看所有的Python虚拟环境
![[Pasted image 20250315161423.png]]



## 3-激活虚拟环境
如果你想要使用你所创建的 python 虚拟环境，首先你就要激活你想用的虚拟环境，激活虚拟环境的命令是：
```conda
conda activate your_env_name
```
比如想要激活刚刚创建的text环境，可以输入
```conda
conda activate test
```
激活虚拟环境以后，在CMD的前面会显示所激活的虚拟环境的名称
![[Pasted image 20250315163306.png]]

#### 常见错误
出现“conda-script.py: error: argument COMMAND: invalid choice: 'activate'”错误
![[Pasted image 20250315163052.png]]
需要先执行`conda init`才行
输入代码
```conda
conda init
```
然后重新启动CMD
![[Pasted image 20250315163200.png]]
## 4-为虚拟环境安装包
创建新的虚拟环境以后，可以输入`list`指令来查看安装的包
```conda
conda list
```
![[Pasted image 20250315163612.png]]
安装其他包有两种方法
#### 激活虚拟环境后安装
使用`pip install`或者`conda install`
```pip
pip install xxx
```
或者使用
```conda
conda install xxx
```
比如安装Numpy，可以输入
```pip
pip install Numpy
```
或者使用
```conda
conda install Numpy
```

#### 未激活虚拟环境直接安装
使用
```conda
python conda install -n your_env_name [package]
```
比如安装Numpy库，可以输入
```conda
conda install -n test numpy
```

## 5-退出以及删除虚拟软件
删除当前虚拟环境的命令是：
```conda
conda deactivate
```
删除虚拟环境的命令是：
```conda
conda remove -n your_env_name --all
 ```
如果我们想要删除刚创建的 test 虚拟环境，只需要输入：
```conda
conda remove -n test --all
```

# 二、使用Anaconda客户端创建虚拟环境

## 1-创建Python虚拟环境

使用 Anaconda 客户端创建 python 虚拟环境，我们只要打开 anaconda 客户端，然后点击 **Environment** ，再点击 **Create** ，填写相应的东西，然后点击 **Create** 就可以了。
![[Pasted image 20250315165422.png]]
创建完毕
![[Pasted image 20250315165841.png]]

## 2-安装包
在anaconda里面安装包，我们首先要选择虚拟环境，然后点击**Not installed**然后搜索想安装的包，最后点击**Apply**就可以了
![[Pasted image 20250315165926.png]]
![[Pasted image 20250315165944.png]]

## 3-激活虚拟环境
想要激活虚拟环境，我们只要点击选择的虚拟环境的**停止符号**，然后点击**Open Terminal**就可以了

## 4-删除虚拟环境
想要删除创建的虚拟环境，我们只要先选择想要的删除的虚拟环境，然后点击**Remove**就可以了

# 



#CMD #Anaconda #Conda #虚拟环境