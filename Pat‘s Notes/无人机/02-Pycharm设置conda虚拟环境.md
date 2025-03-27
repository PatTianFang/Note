# 1-查看或创建虚拟环境

参考[[01-Anaconda建立Python虚拟环境]]

# 2-在pycharm中设置conda创建好的虚拟环境

点击pycharm左上角的`file/settings`
![[Pasted image 20250315193108.png]]
选择`Project/Python Interpreter`，然后选择`Show ALL`
![[Pasted image 20250315193211.png]]
在弹出的界面选择左上角的`+`号
选择Conda环境，然后在右侧选择anconda安装目录下的`scripts/conda.exe`文件。  
【注意：这里选择的是conda的可执行文件，而不是python的可执行文件】
![[Pasted image 20250315193347.png]]
点击右侧的加载环境按钮，下方就会出现已经创建好的虚拟环境选项
![[Pasted image 20250315193511.png]]

# 3-安装库

在控制台中使用`pip`
输入

```python
pip install python-opencv
```

![[Pasted image 20250317192531.png]]










#PyCharm #Conda 