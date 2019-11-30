---
title: 2018.04.21-安恒杯线上赛 RSA writeup
categories:
- ctf
tags:
- RSA
updated: 2018-04-21
---

这题是一个2018年4月安恒的一个线上RSA的writeup

<img src="{{ site.url }}/assets//blog_images/2018/4月/2018.04.21-安恒线上RSA-01.png"/>

 												



### 1.openssl分析私钥

pub.key 打开之后就是这样的

-----BEGIN PUBLIC KEY-----
MDwwDQYJKoZIhvcNAQEBBQADKwAwKAIhAMAzLFxkrkcYL2wch21CM2kQVFpY9+7+
/AvKr1rzQczdAgMBAAE=
-----END PUBLIC KEY-----

首先将两个文件复制到Openssl.exe所在文件目录下，然后打开软件

<img src="{{ site.url }}/assets//blog_images/2018/4月/2018.04.21-安恒线上RSA-02.png"/>



openssl分析私钥，执行**rsa -pubin -text -modulus -in pub.key** 命令exponent就是e值，modulus是n模数的值。

<img src="{{ site.url }}/assets//blog_images/2018/4月/2018.04.21-安恒线上RSA-03.png"/>



Exponent: 65537 (0x10001)
Modulus=C0332C5C64AE47182F6C1C876D42336910545A58F7EEFEFC0BCAAF5AF341CCDD
writing RSA key
-----BEGIN PUBLIC KEY-----
MDwwDQYJKoZIhvcNAQEBBQADKwAwKAIhAMAzLFxkrkcYL2wch21CM2kQVFpY9+7+
/AvKr1rzQczdAgMBAAE=

-----END PUBLIC KEY-----

### 2.然后使用msieve进行分解n值

msieve下载地址：<https://sourceforge.net/projects/msieve/>

\>msieve153.exe 0xC0332C5C64AE47182F6C1C876D42336910545A58F7EEFEFC0BCAAF5AF341CCDD -v

记得在前面加上 0x  后面是-v

分解之后的结果如下：

<img src="{{ site.url }}/assets//blog_images/2018/4月/2018.04.21-安恒线上RSA-04.png"/>



p39 factor: 285960468890451637935629440372639283459
p39 factor: 304008741604601924494328155975272418463

此时既知：p和q的值

p39 factor: 285960468890451637935629440372639283459   //p

p39 factor: 304008741604601924494328155975272418463   //q

### 3.使用脚本

```python
    #!/usr/bin/python 
​    # coding=utf-8   
​    #代码转自实验吧`
​    #通过脚本，根据p，q，e值，生成私钥，貌似该脚本只能在Linux或者cygwin的python下运行。  
​    #我就在windows试试不行，装不了能力有限，试过pip install pycrypto  
​    #果断用Linux吧  
​    import math  
​    import sys  
​    from Crypto.PublicKey import RSA  
​    keypair=RSA.generate(1024)  
​    keypair.p=285960468890451637935629440372639283459`
​    `keypair.q=304008741604601924494328155975272418463  
​    keypair.e=65537  //这个值不要忘记了，不一样的`
​    `keypair.n=keypair.p*keypair.q  
​    Qn=long((keypair.p-1)*(keypair.q-1))  
​    i=1  
​    while(True):  
​        x=(Qn*i)+1  
​        if(x%keypair.e==0):  
​            keypair.d=x/keypair.e  
​            break  
​        i+=1  
​    private=open('private.pem','w')  
​    private.write(keypair.exportKey())  
​    private.close()`  
```

结束之后就可以得到一个private.pem的文件

<img src="{{ site.url }}/assets//blog_images/2018/4月/2018.04.21-安恒线上RSA-05.png"/>



### 4.使用密钥进行解密

**还是在openssl中进行操作：**

命令： rsautl -decrypt -in flag.enc -inkey private.pem

<img src="{{ site.url }}/assets//blog_images/2018/4月/2018.04.21-安恒线上RSA-06.png"/>



flag{decrypt_256}


