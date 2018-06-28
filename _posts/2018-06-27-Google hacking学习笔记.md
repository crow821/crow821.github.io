---
title: Google hacking学习笔记
categories:
- 渗透
tags:
- Google hacking
updated: 2018-06-27
---



GoogleHack,旨在使用Google搜索引擎或者其他Google应用程序通过特定语法来查找网站配置或代码中的安全漏洞。 



基础知识：

1. google查询是不区分大小写的。（除布尔操作符OR）

2.      google通配符；（*仅代表搜索词组中的一个词。在一个词的开始或结尾使用星号和直接使用这个单词的效果相同。

3. Google 会智能地保留一些内容，比如一些过时的词，一些不适合呈现的内容（比如违法信息
4. 32个单词的限制；（如一串英文单词，如果用*部分替换单词，可以扩展搜索单词的数量）
5. 最常用的："关键字" ，双引号会使Google强制搜索包含关键字的内容



基本教程：

intext:关键字(搜索全世界网页正文中有这个关键字的网页)

<img src="{{ site.url }}/assets//blog_images/google_hacking学习笔记_01.png" />

intitle:关键字(标题中含有该关键字的网页)

<img src="{{ site.url }}/assets//blog_images/google_hacking学习笔记_02.png" />

cache:关键字(缓存中的内容)

<img src="{{ site.url }}/assets//blog_images/google_hacking学习笔记_03.png" />

define:关键字(关键字的定义)

<img src="{{ site.url }}/assets//blog_images/google_hacking学习笔记_04.png" />

filetype:文件名.后名(特定的文件类型)  eg: filetype：password.doc

<img src="{{ site.url }}/assets//blog_images/google_hacking学习笔记_05.png" />

filetype中一般会关注的特定的文件：.pwl口令文件        .tmp临时文件       .cfg配置文件        .ini系统文件   

​                                                               .hlp 帮助文件      .dat数据文件      .log 日志文件    

 

info:关键字(搜索指定站点的一些基本信息)

<img src="{{ site.url }}/assets//blog_images/google_hacking学习笔记_06.png" />

inurl:关键字(搜索含有关键字的 URL 地址)  还可以使用 allinurl 来更加精确的定位 URL 地址。eg: 输入 

“inurl:movie”，那么找出来的大部分是电影网站.

<img src="{{ site.url }}/assets//blog_images/google_hacking学习笔记_07.png" />

link:关键字(查找与关键字做了链接的 URL 地址) 利用它我们可能搜索到一些敏感信息。

<img src="{{ site.url }}/assets//blog_images/google_hacking学习笔记_08.png" />

site:域名(返回域名中所有的 URL 地址) 它可以探测网站的拓扑结构，eg: 输入“学院 site:xznu.edu.cn” 就可以搜索到它所有的学院

<img src="{{ site.url }}/assets//blog_images/google_hacking学习笔记_09.png" />

stocks:搜索有关一家公司的股票市场信息 

insubject:搜索 Google 组的标题行 

msgid:搜索识别新闻组帖子的 Google 组信息标识符和字符串 

group:搜索 Google 组搜索词汇帖子的题目 

author:搜索新闻组帖子的作者 

bphonebook:仅搜索商业电话号码簿 

rphonebook:仅搜索住宅电话号码簿 

phonebook:搜索商业或者住宅电话号码簿 

<img src="{{ site.url }}/assets//blog_images/google_hacking学习笔记_10.png" />

daterange:搜索某个日期范围内 Google 做索引的网页 

inanchor:搜索一个 HTML 标记中的一个链接的文本表现形式 



 入侵：

1.查找别人留下的Webshell

很多人在入侵网站得到 Webshell 后，并没有把网页木马的一些关键字去掉，而是保留了原样，这样我们就可以利用Google强大的搜索能力。利用木马的关键字找出那些Webshell 来。比如很多木马都有“绝对的路径、输入保存的路径、输入文件的内容”等关键字。有这个关键字的木马的文件名默认是 diy.asp。那么我们就有了搜索条件，搜索内容为：绝对的路径  输入保存的路径  输入文件的内容  inurl:diy.asp。（亲测无效）

2.搜索存在注入漏洞的站点

我们完全可以结合 Google Hack 技术来达到批量注入的效果， 这里注入我们还需要一个注入工具，对于批量搜索注入方面我觉得还是   啊D   这个工具配合的 比较好，比如我们打开 google，在里面搜索 URL 地址中含有 asp?id=关键字的 URL，然后打开    啊D    把搜索出来的url填入到扫描注入点中的检测网址，然后点击打开网页。后就是点击 google 页面中的下一页按钮，那么这样就可以找到存在注入漏洞的网站了。(哪有那么好找。。。)

<img src="{{ site.url }}/assets//blog_images/google_hacking学习笔记_11.png" />

3.查找特定网站的注入漏洞

我们也可以利用 Google Hack 技术来查 找特定网站的漏洞。比如我们要渗透搜狐网站，找到搜狐网站的注入漏洞。上面我们找的 asp 网站的注入漏洞，下面我们就来找找 PHP 网站的漏洞，我们在 google 中输入 site:sohu.com  inurl:php?id=后就可以看到 sohu 网站所有存在 php?id=的网站了， 查到了这些 php 页面后我们就要自己去一个个判断是否存在注入漏洞了，判断注入漏洞 的方法很简单，就是在网址的后面加上两段代码，一为 and 1=1 和 and 1=2。如果加上了 上面的代码后返回的 and 1=1 和 and 1=2 页面不同，那么就说明存在注入漏洞了。这里还需要注意的一点是 and 和 url 的后面应该有一个空格，比如 http://www.sohu.com/attric.php?id=12 and 1=1，这里大家要特别注意：id=12 与 and 之间存在一个空格，而空格在输入我们的 URL 地 址栏内后就会被编码成%20，也就说%20 在 URL 地址栏内就代表一个空格。（哪有那么简单。。。）







**小结：** 

其实利用 google hacking 来踩点就是利用一些关键字来查询,而且关键字要是越独一无二的那么就收集的信息就越全面。利用 google hacking 来入侵同样也是利用关键字。它的原理是很简单，语法也不多给了我们自己大的自由发挥空间。利用它来入侵网站看上去是很合法的，所以它的隐藏性也是相当强的。只有灵活的运用它才能够达到大的效果。下面给出一些常见的语句

allinurl:bbs data                      查找所有 bbs 中的含有 data 的 URL 

filetype:mdb inurl:database   查找含有 database 的 URL，且查找后名为 mdb 的文件

inurl:data filetype:mdb            查找含有 data 的 URL，且查找后名为 mdb 的文件

intitle:"index of" data            查找网页标题中含有"index of" data 的网页

intitle:"Index of" .bash_history   查找网页标题中含有"Index of" .bash_history的网页

intitle:"index of" passwd          查找网页标题中含有"index of" passwd 的网页

"# -FrontPage-" inurl:service.pwd  查找含有 service.pwd 的 URL 且网页中含有"# -FrontPage-" 

应用：

1.直接搜索获取后台等的信息：

site:www.****.com intext:管理|后台|登陆|用户名|密码|验证码|系统|帐号|manage|admin|login|system

site:www.*****.com inurl:login|admin|manage|manager|admin_login|login_admin|system

site:www.***.com intitle:管理|后台|登陆|

site:www.****.com intext:验证码

2.配合domian等工具实现漏洞扫描

inurl:.asp?id= intext:公司

inurl:115.com  intext:手工注入教程 返回带有115网盘中关于该教程的网页同时，包含校验码。

常用语句：inurl:asp?id=  site:xxx.com

​         inurl:(asp=167?)








