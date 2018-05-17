---
title: dvwa_Command Injection 
categories:
- dvwa
tags:
- Command Injection 
updated: 2018-05-16
---

Command   Injection，即命令注入，是指通过提交恶意构造的参数破坏命令语句结构，从而达到执行恶意命令的目的。PHP命令注入攻击漏洞是PHP应用程序中常见的脚本漏洞之一，国内著名的Web应用程序Discuz!、DedeCMS等都曾经存在过该类型漏洞。

 命令注入攻击的常见模式为：仅仅需要输入数据的场合，却伴随着数据同时输入了恶意代码，而装载数据的系统对此并未设计良好的过滤过程，导致恶意代码也一并执行，最终导致信息泄露或者正常数据的破坏。

### low：

<img src="{{ site.url }}/assets/blog_images/dvwa_Command Injection_01.png" />

代码： 

<img src="{{ site.url }}/assets/blog_images/dvwa_Command Injection_02.png" />

相关函数介绍 

*stristr(string,search,before_search)*

stristr函数搜索字符串在另一字符串中的第一次出现，返回字符串的剩余部分（从匹配点），如果未找到所搜索的字符串，则返回  FALSE。参数string规定被搜索的字符串，参数search规定要搜索的字符串（如果该参数是数字，则搜索匹配该数字对应的 ASCII  值的字符），可选参数before_true为布尔型，默认为“false” ，如果设置为 “true”，函数将返回 search  参数第一次出现之前的字符串部分。

*php_uname(mode)*

这个函数会返回运行php的操作系统的相关描述，参数mode可取值”a”    （此为默认，包含序列”s n r v m”里的所有模式），”s        ”（返回操作系统名称），”n”（返回主机名），”            r”（返回版本名称），”v”（返回版本信息），                ”m”（返回机器类型）。

可以看到，服务器通过判断操作系统执行不同ping命令，但是对ip参数并未做任何的过滤，导致了严重的命令注入漏洞。

window和linux系统都可以用&&来执行多条命令

192.168.1.1&&net user（windows和linux都可以使用）

<img src="{{ site.url }}/assets/blog_images/dvwa_Command Injection_03.png" />

**命令连接符（可以使用cmd进行验证）**

1.        command1 && command2   先执行command1后执行command2

2.          command1 | command2     只执行command2

指令：127.0.0.1 | net user

<img src="{{ site.url }}/assets/blog_images/dvwa_Command Injection_04.png" />

3. command1 & command2    先执行command2后执行command1 

  ​    

   命令：192.168.1.1 & net user 

   

   

<img src="{{ site.url }}/assets/blog_images/dvwa_Command Injection_05.png" />





 以上三种连接符在windows和linux环境下都支持

如果程序没有进行过滤，那么我们就可以通过连接符执行多条系统命令。

而且在Linux下输入192.168.1.1&& cat /etc/shadow甚至可以读取shadow文件（shadow文件是passwd的影子文件）



**Medium**

<img src="{{ site.url }}/assets/blog_images/dvwa_Command Injection_06.png" />

 相比Low级别的代码，服务器端对ip参数做了一定过滤，即把”&&” ,”;”删除，本质上采用的是黑名单机制，因此依旧存在安全问题。因为被过滤的只有”&&”与”;”，所以”&”不会受影响。

由于使用的是str_replace把”&&”,”;”替换为空字符，因此可以采用以下方式绕过：（就像是xss中的双写一样`<scr<script>ipt>alert(/xss/)</script>`绕过一样）

:

​      192.168.1.1 &;& ipconfig

这是因为”192.168.1.1&;& ipconfig”中的” ;”会被替换为空字符，这样一来就变成了”192.168.1.1&& ipconfig” ,会成功执行。

<img src="{{ site.url }}/assets/blog_images/dvwa_Command Injection_07.png" />



同理： 192.168.1.1 &; net user也可以 

<img src="{{ site.url }}/assets/blog_images/dvwa_Command Injection_08.png" />

就算是顺序反过来也是不影响的（;和&符号） 

<img src="{{ site.url }}/assets/blog_images/dvwa_Command Injection_09.png" />

或者是 192.168.1.1 ;| net user  

<img src="{{ site.url }}/assets/blog_images/dvwa_Command Injection_10.png" />

**High** 

<img src="{{ site.url }}/assets/blog_images/dvwa_Command Injection_11.png" />

<img src="{{ site.url }}/assets/blog_images/dvwa_Command Injection_12.png" />

相比Medium级别的代码，High级别的代码进一步完善了黑名单，但由于黑名单机制的局限性，我们依然可以绕过。

漏洞利用

黑名单看似过滤了所有的非法字符，但仔细观察到是把”| ”（注意这里|后有一个空格）替换为空字符，于是    ”|” 就有用了。(感觉这里有点扯。。。。）

<img src="{{ site.url }}/assets/blog_images/dvwa_Command Injection_13.png" />

直接使用：

```
 192.168.1.1 |net user（|net 之间不要有空格，不然就被转义了。。。）
```

 

<img src="{{ site.url }}/assets/blog_images/dvwa_Command Injection_14.png" />





部分数据来自于互联网，本文只是自己的学习笔记，斟酌使用 



 





​
