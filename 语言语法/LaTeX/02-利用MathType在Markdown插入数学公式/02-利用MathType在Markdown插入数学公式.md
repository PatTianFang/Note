---
title: 02-利用MathType在Markdown插入数学公式
date: 2025-05-02 15:55:44
categories: LaTex
tags:
  - MathType
  - Markdown
  - LaTex
---

主要思路就是，在MythType中进行相对应的设置后，直接复制对应公式可以复制出对应的LaTex形式的公式，复制进Markdown即可解析。

打开MythType

![](Pasted%20image%2020250502155956.png)

依次选择**预置**>**剪切和复制预置**

![](Pasted%20image%2020250502160026.png)

![](Pasted%20image%2020250502160204.png)

勾选第二项，下拉框选择**MathML或TeX**，**不要勾选在转换中包括MathType数据**点击确定。  

![](Pasted%20image%2020250502160125.png)

![](Pasted%20image%2020250502161048.png)

输入想要的公式

![](Pasted%20image%2020250502160416.png)

复制

![](Pasted%20image%2020250502160446.png)

直接粘贴即可得到LaTex公式

```LaTex
\[f = \frac{{\sum\limits_i^n {\sqrt {{b^2} - 4ac} } }}{{\frac{{n!}}{{r!\left( {n - r} \right)!}}}}\]
```

将开头和结尾的`\[`修改为`$$`即可

```LaTex
$$f = \frac{{\sum\limits_i^n {\sqrt {{b^2} - 4ac} } }}{{\frac{{n!}}{{r!\left( {n - r} \right)!}}}}$$
```

{% raw %}
$$f = \frac{{\sum\limits_i^n {\sqrt {{b^2} - 4ac} } }}{{\frac{{n!}}{{r!\left( {n - r} \right)!}}}}$$

{% endraw %}

如果不想修改直接粘贴为LaTex公式可以使用下面的方法

方法主要参考CSDN隆里卡那唔的文章《插入mathtype公式如何使公式为`$$`形式，而不是`/[/]`》^[https://blog.csdn.net/weixin_44090680/article/details/130547953]

选择**格式**>**内联公式**

![](Pasted%20image%2020250502162118.png)

复制粘贴后为

```LaTex
$\sqrt {{b^2} - 4ac} $
```

在两边加上`$`即可

```LaTex
$$\sqrt {{b^2} - 4ac} $$
```

{% raw %} $$\sqrt {{b^2} - 4ac} $${% endraw %}




#MathType #LaTex