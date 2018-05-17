---
title: dvwa_brute force(low、medium)
categories:
- dvwa	
tags:
- brute force
updated: 2018-05-16
---

暴力破解一般指穷举法，穷举法的基本思想是根据题目的部分条件确定答案的大致范围，并在此范围内对所有可能的情况逐一验证，直到全部情况验证完毕。若某个情况验证符合题目的全部条件，则为本问题的一个解；若全部情况验证后都不符合题目的全部条件，则本题无解。穷举法也称为枚举法。(转自internet)



 **low：**

代码就不贴了，自己看，然后直接使用工具进行爆破

设置随意的密码和账号进行抓包：

<img src="{{ site.url }}/assets/blog_images/brute force_low_medium_01.png" />



对于账号666 密码 333加$符号：

<img src="{{ site.url }}/assets/blog_images/brute force_low_medium_02.png" />

然后进行爆破：(中间的过程自行百度，为了节省时间，我中间的有些东西已经省略了，可以参考下面的标准）

<img src="{{ site.url }}/assets/blog_images/brute force_low_medium_03.png" />

第一种：
Sniper标签   这个是我们最常用的，Sniper是狙击手的意思。这个模式会使用单一的payload【就是导入字典的payload】组。它会针对每个position中$$位置设置payload。这种攻击类型适合对常见漏洞中的请求参数单独地进行测试。攻击中的请求总数应该是position数量和payload数量的乘积。
第二种：
Battering  ram –  这一模式是使用单一的payload组。它会重复payload并且一次把所有相同的payload放入指定的位置中。这种攻击适合那种需要在请求中把相同的输入放到多个位置的情况。请求的总数是payload组中payload的总数。简单说就是一个playload字典同时应用到多个position中
第三种：
Pitchfork  –  这一模式是使用多个payload组。对于定义的位置可以使用不同的payload组。攻击会同步迭代所有的payload组，把payload放入每个定义的位置中。比如：position中A处有a字典，B处有b字典，则a【1】将会对应b【1】进行attack处理，这种攻击类型非常适合那种不同位置中需要插入不同但相关的输入的情况。请求的数量应该是最小的payload组中的payload数量
第四种：
Cluster  bomb –  这种模式会使用多个payload组。每个定义的位置中有不同的payload组。攻击会迭代每个payload组，每种payload组合都会被测试一遍。比如：position中A处有a字典，B处有b字典，则两个字典将会循环搭配组合进行attack处理这种攻击适用于那种位置中需要不同且不相关或者未知的输入的攻击。攻击请求的总数是各payload组中payload数量的乘积。

详见：[https://blog.csdn.net/u011781521/article/details/54964926](http://https//blog.csdn.net/u011781521/article/details/54964926)

### **Medium**

相比Low级别的代码，Medium级别的代码主要增加了mysql_real_escape_string函数，这个函数会对字符串中的特殊符号（x00，n，r，，’，”，x1a）进行转义，基本上能够抵御sql注入攻击，说基本上是因为查到说  MySQL5.5.37以下版本如果设置编码为GBK，能够构造编码绕过mysql_real_escape_string  对单引号的转义（因实验环境的MySQL版本较新，所以并未做相应验证）；同时，$pass做了MD5校验，杜绝了通过参数password进行sql注入的可能性。但是，依然没有加入有效的防爆破机制（sleep(2)实在算不上）。（转自http://www.freebuf.com/articles/web/116437.html）

具体的mysql_real_escape_string函数绕过问题详见

<http://blog.csdn.net/hornedreaper1988/article/details/43520257>

<http://www.cnblogs.com/Safe3/archive/2008/08/22/1274095.html>

使用方法：方法和low中的没有什么太大的区别（为了速度，故意少添加了字段）

<img src="{{ site.url }}/assets/blog_images/brute force_low_medium_04.png" />



## High级：

High级别的代码加入了Token，可以抵御CSRF攻击，同时也增加了爆破的难度，通过抓包，可以看到，登录验证时提交了四个参数：username、password、Login以及user_token。这个时候就无法进行burpsuit爆破了。。。。

<img src="{{ site.url }}/assets/blog_images/brute force_low_medium_05.png" />



每次服务器返回的登陆页面中都会包含一个随机的user_token的值，用户每次登录时都要将user_token一起提交。服务器收到请求后，会优先做token的检查，再进行sql查询。

同时，High级别的代码中，使用了stripslashes（去除字符串中的反斜线字符,如果有两个连续的反斜线,则只去掉一个）、 mysql_real_escape_string对参数username、password进行过滤、转义，进一步抵御sql注入。

（high会另外写，需要使用脚本）



​
