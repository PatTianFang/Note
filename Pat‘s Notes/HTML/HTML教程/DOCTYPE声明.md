常见的 DOCTYPE 声明
HTML 5
<!DOCTYPE html>
HTML 4.01 Strict
这个 DTD 包含所有 HTML 元素和属性，但不包括表象或过时的元素（如 font ）。框架集是不允许的。
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">
HTML 4.01 Transitional
这个 DTD 包含所有 HTML 元素和属性，包括表象或过时的元素（如 font ）。框架集是不允许的。
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
HTML 4.01 Frameset
这个 DTD 与 HTML 4.01 Transitional 相同，但是允许使用框架集内容。
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Frameset//EN" "http://www.w3.org/TR/html4/frameset.dtd">
XHTML 1.0 Strict
这个 DTD 包含所有 HTML 元素和属性，但不包括表象或过时的元素（如 font ）。框架集是不允许的。结构必须按标准格式的 XML 进行书写。 ^5a394a
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
XHTML 1.0 Transitional
这个 DTD 包含所有 HTML 元素和属性，包括表象或过时的元素（如 font ）。框架集是不允许的。结构必须按标准格式的 XML 进行书写。
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
XHTML 1.0 Frameset
这个 DTD 与 XHTML 1.0 Transitional 相同，但是允许使用框架集内容。
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Frameset//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-frameset.dtd">
XHTML 1.1
这个 DTD 与 XHTML 1.0 Strict 相同，但是允许您添加模块（例如为东亚语言提供 ruby 支持）。
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.1//EN" "http://www.w3.org/TR/xhtml11/DTD/xhtml11.dtd">

来自 <https://www.runoob.com/tags/tag-doctype.html> 



<!DOCTYPE html> 是 HTML5 中唯一的 doctype，也被视作将网页 "升级" 到 HTML5 的第一步。
很多国外网站的 <!DOCTYPE html> 和 <HEAD> 之间都会有一段注释,如:
<!—[if IE 6 ]><html class="ie ielt9 ielt8 ielt7 ie6" lang="en-US"><![endif]—>
<!—[if IE 7 ]><html class="ie ielt9 ielt8 ie7" lang="en-US"><![endif]—>
<!—[if IE 8 ]><html class="ie ielt9 ie8" lang="en-US"><![endif]—>
<!—[if IE 9 ]><html class="ie ie9" lang="en-US"><![endif]—>
<!—[if (gt IE 9)|!(IE)]><!—><html lang="en-US"><!—<![endif]—>
改代码作用于 css，来写一些针对 IE 各版本的样式差异。
先判断用户用的哪个 IE 版本，然后在标签上加上该版本的 class，这样可以方便 hack。
css 文件是这样写的：
.ie6 xxx {};
.ie7 xxx {};
这是目前最好的 hack 方式之一。

来自 <https://www.runoob.com/tags/tag-doctype.html> 


#HTML 
