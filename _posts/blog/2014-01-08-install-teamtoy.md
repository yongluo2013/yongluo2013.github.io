---
layout: post
title: 企业内部安装teamtoy
description: 最近在《码农周刊》上看到一篇谈远程工作效率，偶尔看到团队协作平台，比较感兴趣，所以研究了一下
category: blog
---

声明：  
本博客欢迎转发，但请保留原作者信息!  
新浪微博：@孔令贤HW；  
博客地址：http://lingxiankong.github.io/  
内容系本人及本人团队学习、研究和总结，如有雷同，实属荣幸！  

## 关于teamtoy
最近在《码农周刊》上看到一篇谈[远程工作效率](http://beenhero.com/improve-remote-work-productivity/)的，其实我内心里特别希望能在家办公，时间自由，关键是能多陪家人。但我也知道，远程办公其实对个人的能力和素养要求甚高，并不适用于每个人。这篇文章中，我比较感兴趣的是作者提到的[pragmatic.ly](http://pragmatic.ly/)，一个团队协作工具。上官网看了下，发现它只有30天免费试用期，而且是个类SaaS的平台，界面倒是看着挺清爽，但明显不能满足我的需求。Google一把，发现类似的工具还真不是一般的多，还是老的套路，国外出现一个工具，国内就开始争相模仿。先后看了几个，发现基本都是在线平台，其实对于创业公司或小型团队来说，这些工具倒也够用，但毕竟是托管，我还是倾向于在内部搭建。最终，还真被我找到一款，就是[teamtoy][]了。关于[teamtoy][]我不多解释，有兴趣的自行google。

## 安装
因为是在内网，所以我是通过BMC安装的ubuntu-12.04.2-server-amd64。软件只装了ssh，ip地址为172.25.200.200。  
系统装完后，以root身份ssh远程登录。

1、先执行`apt-get update`  

2、安装mysql  
`apt-get install mysql-server mysql-client`  
中间会要求输入root密码，我这里是123456，安装完后，登录mysql验证安装成功。  
![](/images/2014-01-08-install-teamtoy/1.png)  

3、安装apache  
`apt-get install apache2`  
安装结束后打开浏览器登录http://172.25.200.200/，显示如下表示安装成功。  
![](/images/2014-01-08-install-teamtoy/2.png)  

4、安装php及相关组件。  
`apt-get install php5 libapache2-mod-php5`  
`apt-get install php5-mysql php5-curl php5-gd php5-json`  

5、配置php.ini  
确定php.ini中的`short_open_tag = On`  

6、配置mysql  
可以通过`SELECT @@GLOBAL.sql_mode;`或`select @@sql_mode;`进行查询，通过编辑mysql配置文件确保sql_mode是空或`NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION`，我这里的结果如下：  
![](/images/2014-01-08-install-teamtoy/3.png)  

为[teamtoy][]创建默认的数据库：  

	mysql> create database lpdb;
	Query OK, 1 row affected (0.00 sec)

7、下载[teamtoy][]，链接[点此](http://tt2net.sinaapp.com/?a=download&vid=2097)，可以在本机解压后，将[teamtoy][]目录中的内容拷贝到服务器上的apache根目录下（默认是/var/www/），并修改该目录权限为777，`chmod 777 -R /var/www`  

8、修改[teamtoy][]的配置文件，主要是mysql的密码，其他不用变。

	root@todoserver:~# cp -a /var/www/config/db.config.sample.php /var/www/config/db.config.php
	root@todoserver:~# cp -a /var/www/config/app.config.sample.php /var/www/config/app.config.php
	root@todoserver:~# vi /var/www/config/db.config.php

![](/images/2014-01-08-install-teamtoy/4.png)   

9、重启一下apache  
`/etc/init.d/apache2 restart`  

10、配置[teamtoy][]  
OK，软件一切就绪，我们打开chrome（目前[teamtoy][]暂不支持IE），访问http://172.25.200.200/index.php?c=install，出现如下界面：  
![](/images/2014-01-08-install-teamtoy/5.png)  
可以看到，一切OK，点击继续安装。提示如下，初始化管理员成功。  
![](/images/2014-01-08-install-teamtoy/6.png)  
默认的管理员邮箱是：member@teamtoy.net，登录后显示如下：  
![](/images/2014-01-08-install-teamtoy/7.png)  

11、Enjoy！

[teamtoy]:  http://tt2net.sinaapp.com/  "teamtoy"