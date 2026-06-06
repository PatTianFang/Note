1.SSI定义

“服务器端包含”或“服务器端嵌入”技术。

SSI在HTML文件中，可以通过注释行调用的命令或指针，是一种基于服务器端的网页制作技术。shtml文件就是应用了SSI技术的html文件SSI工作原理因为是基于服务器端的网页制作技术，所以在.shtml页面返回到客户端前，页面中的SSI指令将被服务器解析，在给客户端返回的页面中不会包含SSI指令。如果SSI指令不能被解析，则浏览器会将其做为普通的HTML注释处理。

SSI的速度介于类似于[.asp](onenote:#ASP介绍&section-id={8D9B1C8C-7763-4BA8-95F3-092819A76B6D}&page-id={CEB85FBF-89B5-4478-9D49-439FB6FDA6AA}&end&base-path=https://d.docs.live.net/91966832e6ca5a0c/文档/HTML/细节概念.one)与.html之间。比.asp快，但比.html慢。SSI能使页面在维护的时候更简单，维护的效率也更高。

2.SSI指令基本格式

基本格式：<!--指令名称="指令参数">

eg：<!--#include file="info.htm"-->

说明：

1.<!-- -->是HTML语法中表示注释,当WEB服务器不支持SSI时,会忽略这些信息。

2.[#include 为SSI指令之一](onenote:#SSI网络技术Server%20Side%20Includes&section-id={8D9B1C8C-7763-4BA8-95F3-092819A76B6D}&page-id={E452F0E1-0959-4357-B52F-F862E30C424D}&object-id={6226CAB6-731B-4089-BB71-5A0C90DF6BCC}&D7&base-path=https://d.docs.live.net/91966832e6ca5a0c/文档/HTML/细节概念.one)。

3.file为include的参数,info.htm为参数值，在本指令中指将要包含的文档名。

注意：

1.<!--与#号间无空格,只有SSI指令与参数间存在空格。

2.上面的标点="",一个也不能少。

3.SSI指令是大小写敏感的,因此参数必须是小写才会起作用(经过测试大写也可以)。

3.SSI语法

1、config指令：用于修改SSI的默认设置

参数：errmsg, timefmt, sizefmt

errmsg:设置默认的错误信息，该指令必须入在其它指令的前面

eg：<!--#config errmsg="error!please email mamager!"-->

Timefmt:设置日期与时间的显示格式，需放在echo指令前

eg：<!--#config timefmt="%A, %B %d, %Y"-->

<!--#echo var="last_modified"-->

Sizefmt:设置表示文件大小的单位。如bytes。该指令需要放在fsize指令前使用。

eg：<!--#config sizefmt="bytes"-->

<!--#fsize file="head.html"-->

2、Include指令：用于将其它文档或元素包含在当前文档中

参数：virtual ，file

virtual:给出到服务器端某个文档的虚拟路径

eg：<!--include virtual="/includes/header.html"-->

[file:给出到当前目录的相对路径，其中不能使用"../"，也不能使用绝对路径](file://给出到当前目录的相对路径，其中不能使用%22../%22，也不能使用绝对路径)

eg：<!--include file="header.html"-->

这就要求每一个目录中都包含一个header.html文件。

3、exec指令：将某一外部程序的输出插入到页面中，执行 CGI 脚本或者 shell 命令

参数：cmd，cgi

cmd 常规应用程序：<!--#exec cmd="文件名称"-->

cgi CGI脚本程序：<!--#exec cgi="文件名称"-->

注意：这个指令相当方便,但是也存在安全问题。

4、Echo：用于显示各种环境变量

参数：var

eg：<!--#config timefmt="%A,the %d of %B,in the year %Y"-->

<!--#echo var="DATE_LOCAL"-->

输出结果：Saturday, the 15 of April, in the year 2000

以下是常见的服务器变量：

DOCUMENT_NAME：显示当前文档的名称

DOCUMENT_URI：显示当前文档的虚拟路径

QUERY_STRING_UNESCAPED：显示未经转义处理的由客户端发送的查询字串，其中所有的特殊字符前面都有转义符"\

DATE_LOCAL：显示服务器设定时区的日期和时间。用户可以结合config命令的timefmt参数，定制输出信息

DATE_GMT：功能与DATE_LOCAL一样，只不过返回的是以格林尼治标准时间为基准的日期

LAST_MODIFIED：显示当前文档的最后更新时间

除了SSI环境变量之外，echo命令还可以显示以下CGI环境变量:

SERVER_SOFTWARE：显示服务器软件的名称和版本

SERVER_NAME：显示服务器的主机名称，DNS别名或IP地址

SERVER_PROTOCOL：显示客户端请求所使用的协议名称和版本，如HTTP/1.0

SERVER_PORT：显示服务器的响应端口

REQUEST_METHOD：显示客户端的文档请求方法，包括GET, HEAD, 和POST

REMOTE_HOST：显示发出请求信息的客户端主机名称

REMOTE_ADDR：显示发出请求信息的客户端IP地址

AUTH_TYPE：显示用户身份的验证方法

REMOTE_USER:显示访问受保护页面的用户所使用的帐号名称

5、Fsize：显示指定文件的大小，可以结合config命令的sizefmt参数定制输出格式

参数：file

eg：<!--#fsize file="index_working.html"-->

Sizefmt：决定文件大小是以字节、千字节还是兆字节为单位表示。如果以字节为单位，参数值为 "bytes"；对于千字节和兆字节可以使用缩写形式。同样，sizefmt 参数必须放在 fsize 命令的前面才能使用。

eg：<!--#config sizefmt="bytes"-->

<!--#fsize file="index.html"-->

6、Flastmod：显示指定文件的最后修改日期，可以结合config 命令的timefmt参数控制输出格式

参数：file

eg：<!--#config timefmt="%A, the %d of %B, in the year %Y"-->

<!--#flastmod file="file.html"-->

XSSI(Extended SSI)是一组高级SSI指令,内置于Apache1.2或更高版本的mod-include模块之中。

#printenv：显示当前存在于WEB服务器环境中的所有环境变量。

语法：<!--#printenv-->

#set：可给变量赋值,以用于后面的if语句。

语法：<!--#set var="变量名" value="变量值"-->

eg：

<!--#set var="color" value="red"-->

<!--#echo var="color"-->

#if：创建可以改变数据的页面,这些数据根据使用if语句时计算的要求予以显示。

语法：

<!--#if expr="$变量名='变量值A'"-->

  显示内容

<!--#elif expr="$变量名='变量值B'"-->

  显示内容

<!--#else-->

  显示内容

<!--#endif-->

eg：

<!--#if expr="$color='red'"-->

    红色

<!--#elif expr="$color='blue'"-->

    蓝色

<!--#else-->

    黑色

<!--#endif-->

SSI内置的变量：

AUTH_TYPE ——针对用户的认证授权方式： BASIC ， FORM ， etc. 和 Tomcat 内的认证方式同步

CONTENT_LENGTH ——从服务器表单传过来的数据长度，字符数目或者数据的字节数

CONTENT_TYPE ——服务器访问呢数据的 MIME 类型，比如“ text/html ”

DATE_GMT ——目前的时间格式方式使用 GMT

DATE_LOCAL ——目前的时间格式方式设置成为本地时间格式

DOCUMENT_NAME ——当前上下文环境的文件地址

DOCUMENT_URI ——虚拟路径定义的文件地址

GATEWAY_INTERFACE —— CGI 的版本定义：“ CGI/1.1 ”

HTTP_ACCEPT ——一个客户端可以接受的 MIME 类型列表

HTTP_ACCEPT_ENCODING ——客户端可以接受的压缩文件类型的列表

HTTP_ACCEPT_LANGUAGE ——客户端可以支持的语言列表

HTTP_CONNECTION ——管理客户端的连接：是“ Close ”还是“ Keep-Alive ”

HTTP_HOST ——客户端请求的站点地址

HTTP_REFERER ——客户端请求之前所在的 URL 地址

HTTP_USER_AGENT ——客户使用的浏览器端的请求结果

LAST_MODIFIED ——当前页面上一次访问和修改的时间

PATH_INFO ——访问此 Servlet 的路径信息

PATH_TRANSLATED —— PATH_INFO 提供的 translated 版本

QUERY_STRING ——在 URL 地址 ? 之后的请求参数列表

QUERY_STRING_D ——没有经过编码过的请求参数

REMOTE_ADDR ——用户请求客户端 IP 地址

REMOTE_HOST ——用户发送请求的主机名

REMOTE_PORT ——用户发送请求的端口号

REMOTE_USER ——认证授权需要的发送请求的用户名

REQUEST_METHOD ——请求使用方法： GET 或者 POST

REQUEST_URI ——客户端原来访问请求的 Web 页面的 URI 地址

SCRIPT_FILENAME ——在服务器上当前页面的地址

SCRIPT_NAME ——当前页面的名称

SERVER_ADDR ——服务器所在的 IP 地址

SERVER_NAME ——服务器的主机名或者 IP 地址

SERVER_PORT ——服务器接受请求的端口号

SERVER_PROTOCOL ——服务器处理请求的协议：“ HTTP/1.1 ”

SERVER_SOFTWARE ——服务器响应客户端请求的名称和版本号

SSI中日期格式的定义：

     %a 一周七天的缩写形式 Thu

    %A 一周七天 Thursday

    %b 月的缩写形式 Apr

    %B 月 April

    %d 一个月内的第几天 13

    %D mm/dd/yy日期格式 04/13/00

    %H 小时（24小时制，从00到23） 01

    %I 小时（12小时制，从00到11） 01

    %j 一年内的第几天，从01到365 104

    %m 一年内的第几个月，从01到12 04

    %M 一小时内的第几分钟，从00到59 10

    %p AM或PM AM

    %r 12小时制的当地时间，格式为 01:10:18 AM

    %S 一分钟内的第几秒，从00到59 18

    %T 24小时制的%H:%M:%S时间格式 01:10:18

    %U 一年内的第几个星期，从00到52，以星期天作为每个星期的第一天

    %w 一星期内的第一天，从0到6 4

    %W 一年内的第几个星期，从00到 53，以星期一作为每个星期的第一天

    %y 年的缩写形式，从00到99 00

    %Y 用四位数字表示一年 2000

    %Z 时区名称 MDT

转载于:https://www.cnblogs.com/jspa/p/10823549.html

来自 <[https://blog.csdn.net/weixin_30693183/article/details/97769413](https://blog.csdn.net/weixin_30693183/article/details/97769413)>

#SSI（Server_Side_Includes）
