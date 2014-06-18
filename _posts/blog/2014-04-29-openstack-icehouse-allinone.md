---
layout: post
title: OpenStack Icehouse 版本 All-in-one 离线安装指导
description: OpenStack Icehouse 版本 All-in-one 离线安装指导
category: blog
---

声明：  
本博客欢迎转发，但请保留原作者信息!  
新浪微博：[@孔令贤HW](http://weibo.com/lingxiankong)；   
博客地址：<http://lingxiankong.github.io/>  
内容系本人及本人团队学习、研究和总结，如有雷同，实属荣幸！

ISO第一作者：钱林  
支撑团队：华为OpenStack社区团队（西安）  
**更新日期：2014.6.4**

----------

## 优点
* 基于4.17号发布的Icehouse版本
* 主机操作系统基于ubuntu 12.04 server版，与openstack兼容性高
* 离线安装，特别适用于有网络限制的场景
* 对ubuntu安装过程进行了优化，傻瓜式安装配置，简单，高效
* 集成了简单的健康检查
* 同时支持虚拟部署和物理部署
* discovered by you……

## 缺点
* 对物理/虚拟服务器的网卡和磁盘有一定要求(网卡数目>=3; disk size>=30G)
* 如果您在使用过程中遇到其他问题，请直接留言，我会尽快回复

## 使用前提
* 获取ISO，地址：<http://dl.vmall.com/c04azzmdnu> (2014.6.5号更新)
* 获取网络信息规划
* 物理/虚拟服务器需要至少3块网卡和1块至少30G空间的磁盘
* 如果是物理安装，请获取预安装服务器的BMC IP地址；

> 当然，你也可以配置PXE服务器

## 物理安装指导
一、配置服务器光盘启动  

二、使用BMC IP登录服务器，加载ISO，重启服务器  
![](/images/2014-04-29-openstack-icehouse-allinone/1.png)

三、服务器重启后会自动进入安装程序  
![](/images/2014-04-29-openstack-icehouse-allinone/2-1.png)

四、如果您的网络中有DHCP服务器，则跳过此步骤；否则，根据您的网络规划配置eth0的网络  
![](/images/2014-04-29-openstack-icehouse-allinone/4-1.png)  
![](/images/2014-04-29-openstack-icehouse-allinone/4-2.png)  

五、如果您的服务器只有1块磁盘，则跳过此步骤；否则，请配置磁盘和分区  
![](/images/2014-04-29-openstack-icehouse-allinone/5-1.png)   

六、自动安装主机操作系统和OpenStack (主机操作系统用户名和密码：root/root)，系统默认只安装OpenSSH   
![](/images/2014-04-29-openstack-icehouse-allinone/6-1.png)  
![](/images/2014-04-29-openstack-icehouse-allinone/6-2.png)

七、请喝杯茶，耐心等待...  

八、验证安装。登录主机，查看健康检查的日志；登录Horizon页面  
![](/images/2014-04-29-openstack-icehouse-allinone/7-1.png)   
![](/images/2014-04-29-openstack-icehouse-allinone/7-2.png)

九、更详细的使用指导请参考[这里](http://lingxiankong.github.io/blog/2014/05/12/huawei-allinone-operation-guide/)

## 虚拟安装指导
在centos 6.5 KVM 虚拟机、ESX 5.1 虚拟机、FusionSphere 虚拟机中验证通过。

### centos 6.5
在centos6.5下使用virtual Machine Manager创建**3网卡**虚拟机，挂载ISO，启动系统，安装过程同上。    
![](/images/2014-04-29-openstack-icehouse-allinone/image019.png)  

![](/images/2014-04-29-openstack-icehouse-allinone/image021.png)  

![](/images/2014-04-29-openstack-icehouse-allinone/image023.png)  

![](/images/2014-04-29-openstack-icehouse-allinone/image025.png)  

![](/images/2014-04-29-openstack-icehouse-allinone/image027.png)  

![](/images/2014-04-29-openstack-icehouse-allinone/image029.png)  

### ESX 5.1
在ESX 5.1下创建vmware虚拟机，配置如下，安装过程同上。    
![](/images/2014-04-29-openstack-icehouse-allinone/image031.png)