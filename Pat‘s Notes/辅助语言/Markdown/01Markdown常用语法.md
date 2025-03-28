# 反向连接
```Markdown
[[这个文件的本名，必须是本库中，这个一般会有自动化的选项给你指引|链接显示名称]]
```
[[00注意点|示范]]

# 标题
```Markdown
# 这是一级标题
## 这是二级标题
### 这是三级标题
```
注意
标题最多支持六级
###### 六
####### 七

# 超链接
```Markdown
[超链接显示名称]（URL）
```
[示范](http://pattianfang.github.io/)

# 图片
```Markdown
![图片alt文字](图片链接"figure title")
```
![RUNOOB 图标](http://static.runoob.com/images/runoob-logo.png)

![RUNOOB 图标](http://static.runoob.com/images/runoob-logo.png "RUNOOB")
# 字体样式
```Markdown
*这是倾斜的文字*
**这是加粗的文字**
***这是倾斜加粗的文字***
==这是加高光的文字==
~~这是加删除线的文字~~
<u>这是加下划线的文字</u>
```
示范：
	*这是倾斜的文字*
	**这是加粗的文字**
	***这是倾斜加粗的文字***
	==这是加高光的文字==
	~~这是加删除线的文字~~
	 <u>这是加下划线的文字</u>

# 分割线
```Markdown
***(超过两个即可)
```
示范
*
**
***

# 列表
## 无序列表
```Markdown
- 列表内容
```
示范
- 1
- 2
- 3
- 4

无序列表使用星号(*)、加号(+)或是减号(-)作为列表标记，这些标记后面要添加一个空格，然后再填写内容：
```Markdown
* 第一项
* 第二项
* 第三项

+ 第一项
+ 第二项
+ 第三项


- 第一项
- 第二项
- 第三项
```
* 第一项
* 第二项
* 第三项

+ 第一项
+ 第二项
+ 第三项

## 有序列表
```Markdown
1. 列表内容
```
示范
1. Markdown
2. 我要
3. 学会你

注意
中间的空格别忘了
不然就这样了：
1.错误示范

## 列表嵌套
```Markdown
1. 第一项
    1. 用四个空格：列表嵌套 或者 Tab键 再输入这些
```
示范
1. 哈哈
2. 哇哇
	1. 呱呱
		1. 叽叽
	2. 呃呃

# 表格
```Markdown
表头|表头|表头
(:，因为有无都是左对齐)---（且这个-数得大于等于三）|:---:|---:
这玩意据说<br/>可以换行|内容|内容
内容|内容|内容
```
我们可以设置表格的对齐方式：

-: 设置内容和标题栏居右对齐。
:- 设置内容和标题栏居左对齐。
:-: 设置内容和标题栏居中对齐。

| 左对齐 | 右对齐 | 居中对齐 |
| :-----| ----: | :----: |
| 单元格 | 单元格 | 单元格 |
| 单元格 | 单元格 | 单元格 |
# 备注
```Markdown
`(备注)`
```
示范
`备注`
（和单行代码一样）

# 代码
## 单行代码
```Markdown
`(单行代码)`
```
示范
`print("Hello World!")`

## 代码块
```Markdown
(```)(语言名)
（代码块）
(```)
```

```Markdown
代码区块前使用 4 个空格或者一个制表符（Tab 键）。
```
示范

	$(document).ready(function () {
	    alert('RUNOOB');
	});

# 代办事项
```Markdown
- [x] 支持 #标签 ，[链接]()，**样式**
- [x] 要求包含列表标志（有序表无序表均可，比如 `1.[x]` 同样可以）
- [x] 这是一个已经完成的项目
- [?] 这也是一个已完成的项目（实际上你可以在其中使用任何字符）
- [ ] 这是一个未完成的项目
- [ ] 在预览模式下单击选框可以切换项目完成状态
```
示范
- [x] 支持 #标签 ，[链接]()，**样式**
- [x] 要求包含列表标志（有序表无序表均可，比如 `1.[x]` 同样可以）
- [x] 这是一个已经完成的项目
- [?] 这也是一个已完成的项目（实际上你可以在其中使用任何字符）
- [ ] 这是一个未完成的项目
- [ ] 在预览模式下单击选框可以切换项目完成状态

# 引用
```Markdown
>引用的内容
>>嵌套引用的内容
```
示范
	对比（没加>）
>引用的内容
>>嵌套引用的内容



# 参考资料
[^1.https://blog.csdn.net/Malulim/article/details/123124534]



#Markdown
#常用语法