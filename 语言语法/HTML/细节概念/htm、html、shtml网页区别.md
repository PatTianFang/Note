很多人会认为网页扩展名html和htm是等同的，但事实上他们还是有区别的。

包含HTML内容的文件最常用的扩展名是.html，但是像DOS这样的旧操作系统限制扩展名为最多3个字符，所以.htm扩展名也被使用。虽然现在使用的比较少一些了，但是.htm扩展名仍旧普遍被支持。

两种都是静态网页文件的扩展名，扩展名可以互相更换而不会引起错误(这是指打开而言，但是对于一个链接来说，如果它指向的是一个htm文件，而那个htm文件被更改为html文件，那么是找不到这个连接的)

1. html 比 htm 的载入速度快

2. htm 为DOS三字符文件扩展名时代而来

3. html 为Windows时代支持多字符扩展名

来自 <[https://www.cnblogs.com/Renyi-Fan/p/8733001.html](https://www.cnblogs.com/Renyi-Fan/p/8733001.html)>

.html与.htm均是静态网页后缀名，网页文件没有区别与区分，html与htm后缀网页后缀可以互换，对网页完全没有影响同时也没有区别。

来自 <[https://www.cnblogs.com/Renyi-Fan/p/8733001.html](https://www.cnblogs.com/Renyi-Fan/p/8733001.html)>

shtml命名的网页文件里，使用了[[SSI网络技术Server Side Includes|SSI]]的一些指令，就像asp中的指令，你可以在SHTML文件中写入SSI指令，当客户端访问这些shtml文件时，服务器端会把这些SHTML文件进行读取和解释，把SHTML文件中包含的SSI指令解释出来。

而shtml与shtm后缀的网页文件没有区别，后缀名可以互换，区别在于和html与htm一样多与少“L”。

来自 <[https://www.cnblogs.com/Renyi-Fan/p/8733001.html](https://www.cnblogs.com/Renyi-Fan/p/8733001.html)>

shtml的SSI功能

SSI是为WEB服务器提供的一套命令，这些命令只要直接嵌入到HTML文档的注释内容之中即可。如：

<!--#include file="info.htm"-->

就是一条SSI指令，其作用是将"info.htm"的内容拷贝到当前的页面中，当访问者来浏览时，会看到其它HTML文档一样显示info.htm其中的内容。

假如我们A页面是shtml的静态网页，而A页面里我们使用了include包含嵌入B静态html页面，如果你的服务器空间支持Shtml SSI这个时候我们，浏览器打开A页面时候，就会在A页面显示A原本内容以及B页面内容，我们查看网页源代码，不会发现B页面引入痕迹，而是看到B页面内容完全在A页面里。

假如：

1、A shtml页面里内容是：

我包含页面B：<!--#include file="b.html"-->

2、B html网页内容：

我是B页面内容

3、这个时候浏览器查看A页面HTML源代码：

A shtml页面里内容是：我是B页面内容

这个就是shtml ssi 包含include魅力之处。

其它的SSI指令使用形式基本同刚才的举例差不多，可见SSI使用只是插入一点代码而已，使用形式非常简单。

当然，如果WEB服务器不支持SSI，它就会只不过将它当作注释信息，直接跳过其中的内容；浏览器也会忽略被包含信息，我们可以查看源代码看到include引入注解信息。

来自 <[https://www.cnblogs.com/Renyi-Fan/p/8733001.html](https://www.cnblogs.com/Renyi-Fan/p/8733001.html)>


#HTML