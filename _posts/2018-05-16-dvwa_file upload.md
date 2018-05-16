---
title: dvwa_file upload
categories:
- dvwa
tags:
- file upload
updated: 2018-05-16
---



**low等级:**

可以使用一句话木马和中国菜刀

<img src="{{ site.url }}/assets//blog_images/dvwa_file upload_01.png" />



低等级的没有对于上传做任何的过滤或者是防护，所以直接用一句话木马进行上传即可（相关原理可以参照其他大佬的）

首先直接用一个txt里面构造一句话木马即可：（php）：`<?php @eval($_POST['pass']);?>`

<img src="{{ site.url }}/assets//blog_images/dvwa_file upload_02.png" />



直接命名为pass.php即可（只是针对low等级），然后上传

<img src="{{ site.url }}/assets//blog_images/dvwa_file upload_03.png" />



上传成功，路径如下：`../../hackable/uploads/pass.php succesfully uploaded!`

完整路径为：http://127.0.0.1/dvwa/hackable/uploads/pass.php   密码为：pass（就是一句话中间的密码）

使用中国菜刀进行连接

<img src="{{ site.url }}/assets//blog_images/dvwa_file upload_04.png" />



直接连接即可

<img src="{{ site.url }}/assets//blog_images/dvwa_file upload_05.png" />







# Medium级：

<img src="{{ site.url }}/assets//blog_images/dvwa_file upload_06.png" />



由此可以看到，medium中，限定了文件的上传类型，只允许JPEG或者是png格式的文件，直接上传php文件会显示失败，所以这个时候需要使用burpsuit进行修改包。（法1）

<img src="{{ site.url }}/assets//blog_images/dvwa_file upload_07.png" />

将文件修改为pass.png，直接点击上传，然后修改burpsuit中数据

<img src="{{ site.url }}/assets//blog_images/dvwa_file upload_08.png" />



!<img src="{{ site.url }}/assets//blog_images/dvwa_file upload_09.png" />

上传成功。

然后显示上传成功，直接使用中国菜刀进行连接即可。





<img src="{{ site.url }}/assets//blog_images/dvwa_file upload_10.png" />

​    法二

在php版本小于5.3.4的服务器中，当Magic_quote_gpc选项为off时（其实不止是这个函数，还有很多），可以在文件名中使用%00截断，所以可以把上传文件命名为hack.php%00.png。如果Magic_quote_gpc函数打开的时候，所有的' " \ NULL 等都会被加上一个反斜线进行转义

可以看到，包中的文件类型为image/png，可以通过文件类型检查。

这个时候直接上传可有：

<img src="{{ site.url }}/assets//blog_images/dvwa_file upload_11.png" />

上传成功。



   <img src="{{ site.url }}/assets//blog_images/dvwa_file upload_12.png" />

直接进行连接即可！

法3：使用burpsuit    00截断：

命名文件为：

<img src="{{ site.url }}/assets//blog_images/dvwa_file upload_13.png" />



然后使用burpsuit进行拦截，改包

<img src="{{ site.url }}/assets//blog_images/dvwa_file upload_14.png" />



在hex中将2e修改为00，然后查看下：

<img src="{{ site.url }}/assets//blog_images/dvwa_file upload_15.png" />



这个时候直接进行上传即可。

<img src="{{ site.url }}/assets//blog_images/dvwa_file upload_16.png" />



显示上传成功。

**high:**



此时如果继续使用

<img src="{{ site.url }}/assets//blog_images/dvwa_file upload_17.png" />

这种格式进行上传的话，会报错。

<img src="{{ site.url }}/assets//blog_images/dvwa_file upload_18.png" />



（法一）

构造图片一句话木马，用getimagesize()函数识别测试（注意图片大小尽可能的小，不然在上传的时候会报错）

使用的是notepad++进行在尾部添加一句话木马

<img src="{{ site.url }}/assets//blog_images/dvwa_file upload_19.png" />



文件名称是pass.png。然后直接上传，进行连接：

<img src="{{ site.url }}/assets//blog_images/dvwa_file upload_20.png" />



法二：将文件头部构造为图片的格式，然后进行%00上传即可：

采用%00截断的方法可以轻松绕过文件名的检查，但是需要将上传文件的文件头伪装成图片

<img src="{{ site.url }}/assets//blog_images/dvwa_file upload_21.png" />



然后上传

<img src="{{ site.url }}/assets//blog_images/dvwa_file upload_22.png" />



再进行连接：

<img src="{{ site.url }}/assets//blog_images/dvwa_file upload_23.png" />

即可！



部分学习于互联网，中间有不详细的请自行百度，斟酌使用！

​
