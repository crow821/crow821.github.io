---
title: Unbantu16.04开启ssh
categories:
- Unbantu
tags:
- ssh
updated: 2018-05-16
---

**一、注意点：**

 1. 记得在首先使用的时候对于unbantu的源进行更换到国内的源

 2. 而且命令要使用sudo 

    

    


 **二、步骤：**


 - 1.unbantu16.04开启ssh的方法和kali的有些不同，需要安装ssh 服务 

   

- 

```Java
sudo apt-get install openssh-server
```

- <img src="{{ site.url }}/assets//blog_images/unbantu开启shh01.png" />
```Java
然后开启ssh服务： service ssh start
```

<img src="{{ site.url }}/assets//blog_images/unbantu开启shh02.png" />

---

 - 可以查看ssh服务是否启动：

   

   

 ```Java
 sudo ps -e |grep ssh
 ```
 - 然后使用ifconfig命令查看本机的ip：
 - 

```Java
ifconfig
```
 - <img src="{{ site.url }}/assets//blog_images/unbantu开启shh03.png" />

 - 然后使用ssh工具xshell进行连接即可：

   

```Java
命令：ssh 192.168.127.153
```
<img src="{{ site.url }}/assets//blog_images/unbantu开启shh04.png" />

[]: 



​
