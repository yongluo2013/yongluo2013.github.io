---
layout:     post
title:      Mirantis OpenStack
category: blog
description: 简单分析mirantis的openstack社区版都做了什么
tags: openstack, mirantis
---

声明：  
本博客欢迎转发，但请保留原作者信息!  
博客地址：<http://lingxiankong.github.io/>  
内容系本人及本人团队学习、研究和总结，如有雷同，实属荣幸！  
Author：华为云计算工程师 [孔令贤](http://weibo.com/lingxiankong) 

# Mirantis OpenStack #

## About mirantis
mirantis，一家很牛逼的openstack服务集成商，他是社区贡献排名前5名中唯一一个靠软件和服务吃饭的公司（其他分别是Red Hat, HP, IBM, Rackspace）。相对于其他几个社区发行版，mirantis openstack的版本节奏很快，平均每两个月就能提供一个相对稳定的社区版。

## What is Mirantis OpenStack?
mirantis openstack是mirantis的openstack社区发行版，除了OpenStack社区源码外，主要包含：  

- 安装工具Fuel。支持生产环境的安装（HA），同时包含了安装之后的管理功能，例如扩容，减容以及回退安装等，同时提供了用户友好的配置界面进行复杂的网络和存储的配置。  
- 增强代码。这部分主要包括：支持HA的代码；未合入社区的bug fix；由mirantis发起的孵化项目（Savanna and Murano）；由mirantis认证的第三方的插件的集成。  
- 技术支持。例如，根据SLA的不同，根据严重性的不同等级，提供不同的响应时间。

mirantis openstack 3.2基于Grizzly版本，而最新发布的4.0版本是基于Havana版本的技术预览版，不可用作生产环境使用，同时，4.0版本仍然不包含Heat和Ceilometer组件。

## 安装
Fuel在mirantis openstack中扮演着重要的角色。基本的安装步骤：先安装Fuel server，再安装node servers。既支持虚拟部署，又可以物理部署。Fuel Server其实就是Cobbler Server和Puppet Master，作为种子节点。Puppet组件是通过Cobbler送到各节点，而Red Hat的packstack则是通过ssh的scp命令直接拷过去的。
  
mirantis openstack同时支持CentOS & Ubuntu操作系统，也支持在RHEL上安装Red Hat OpenStack。而Ubuntu 12.04操作系统是直接集成在Fuel的ISO安装包中的。

如果是虚拟部署，mirantis推荐在Mac OS 10.7.x/10.8.x, CentOS 6.4, or Ubuntu 12.04等操作系统上安装VirtualBox，使用mirantis提供的VirtualBox脚本安装。该脚本会先使用fuel iso镜像创建fuel node虚拟机，然后创建node servers虚拟机并从fuel node PXE启动，如果是使用物理服务器作为node servers，要确保服务器与fuel node虚拟机在同一个2层网络，并且手动对他们进行PXE启动，节点会被Fuel自动发现。安装完fuel node，会返回一个链接，登陆后进行配置并部署整个环境。

安装fuel server前的checklist:  
![](/images/2013-11-19-mirantis-openstack/1.png)

安装fuel后，部署openstack需要的配置步骤：  
![](/images/2013-11-19-mirantis-openstack/2.png)
![](/images/2013-11-19-mirantis-openstack/3.png)

安装fuel很简单，mirantis提供了ISO/IMG镜像文件，可以安装在虚拟机或物理机上。安装完后，对外提供web服务，可以登录http://<*ip address*>:8000，按照上述步骤对环境进行配置部署，安装过程中可以查看详细的操作日志。

整个安装部署架构图如下：  
![](/images/2013-11-19-mirantis-openstack/4.png)

Fuel中集成的开源组件：  
![](/images/2013-11-19-mirantis-openstack/6.png)

安装完后，mirantis openstack提供了健康检查工具对环境进行检查，如果失败，会有详细的失败操作描述。检查过程如下图(检查过程有些类似于tempest测试用例)：  
![](/images/2013-11-19-mirantis-openstack/5.png)

## HA

- Mysql使用Galera做Active/Active集群，同时使用Pacemaker，因为Galera mysql用到了领导机选举机制quorum，所以控制节点至少三个  
- RabbitMQ使用mirrored queues，运行在Active/Active模式  
- 有状态服务如neutron agents使用Pacemaker做Active/Passive部署  
- 无状态服务前端加HAProxy，所以无状态服务并没有部署在计算节点上  

![](/images/2013-11-19-mirantis-openstack/7.png)

Controller Node（至少3节点）的HA部署图如下，每个controller node都运行有HAProxy，为所有的controller node管理一个VIP，提供HTTP和TCP协议的负载均衡。  
![](/images/2013-11-19-mirantis-openstack/8.png)

## 总结
总结一下，mirantis openstack几个优点：  
1、节点的自动发现和预校验  
2、配置简单、快速  
3、支持多种操作系统和发行版，支持HA部署  
4、对外提供API对环境进行管理和配置，例如动态添加计算/存储节点  
5、自带健康检查工具  
6、支持Neutron，例如GRE和namespace都做进来了，子网能配置具体使用哪个物理网卡

几个缺点：  
1、Grizzly中的有一些特性目前mirantis openstack还不支持，例如：对于Nova不支持cells, avaliability zones, host aggregates；对于Neutron不支持LBaaS和multi-host; 对于Keystone不支持multi-factore授权和PKI授权；对于Cinder不支持FCoE和使用LIO作为iSCSI后端；也不支持Ceilometer与Heat。  
2、CLI不支持部署Swift和Neutron  
3、配置很灵活，但灵活的另一面就是繁琐

----------
*参考链接*

- [mirantis](http://software.mirantis.com/)  
- [Mirantis Fuel调研](http://www.openstack.cn/p383.html)

