---
layout: post
title: OpenStack社区动态第十四期(0508-0528)
description: OpenStack社区动态第十四期(0508-0528)
category: blog
---

声明：  
本动态跟踪系列由华为OpenStack团队出品，由孔令贤整理，欢迎转发，但请保留原作者信息!  
新浪微博：[@孔令贤HW](http://weibo.com/lingxiankong)；  
博客地址：<http://lingxiankong.github.io/> 

## 业界动态
OpenStack私有云服务商Metacloud近日完成B轮1500万美元融资，推动企业自建和Metacloud托管私有云服务开发和营销推广。目前Metacloud已经累计融资2700万美元。

[EasyStack](http://www.easystack.cn/en/)将会在本次的Atlanta峰会上有一个demo theater，EasyStack是一家OpenStack云解决方案和服务提供商，基于OpenStack为企业用户提供开放、稳定、可靠、可扩展的弹性云计算平台。

北京时间5.12号，OpenStack一年两次的峰会在Atlanta召开，主会议时间将从在亚特兰大时间五月12日早9点（北京时间五月12日晚9点）开始，持续到美国时间周四下午5点。同时，设计峰会将从周二持续到周五

5.6号，HP注资10亿美元到Helion，基于OpenStack提供混合云和主机租赁服务。同时，HP发布了HP Helion OpenStack Community edition

在Atlanta峰会上，Canonical发布了Orange Box，我的理解是不是基于OpenStack的一体机？当然，官方宣称目前仅用作测试或POC用；同时，发布了Your Cloud云服务；同时，与IBM和CloudBase在Juju的推广和使用方面进行合作。

Atlanta峰会上，IBM发布了自己的基于Icehouse版本的OpenStack发行版，支持自己的硬件设备。

5.12，SUSE宣称在其基于 OpenStack Havana 版本的[SUSE Cloud3](https://www.suse.com/zh-cn/promo/susecloud3.html)产品中增强了系统的[可用性](https://www.suse.com/zh-cn/products/highavailability/)，成为首个实现高可用性配置和部署自动化的企业级 OpenStack 发行套件，旨在确保无缝的安装体验和关键云服务的不间断交付。

OpenStack Marketplace上线了，上去看一眼就知道它是干什么的，说白了，就是信息传递，业界都有哪些发行版，哪些基于OpenStack的公有云服务，哪些厂商接入了OpenStack等等。  
<http://www.openstack.org/marketplace>

DreamHost发布了基于OpenStack的公有云产品[DreamCompute]()，采用自家三层网络产品(Akanda)，提供IPv6的支持和完全的租户隔离，而二层与VMware合作，使用NVP提供服务。存储使用了Ceph。

2014.05.28，Mirantis发布了Mirantis OpenStack 5.0，包含如下特性：  
集成 OpenStack Icehouse 2014.1 版本；  
Fuel Master Node 支持升级；  
与 VMWare vCenter 集成, 支持 ESXi servers 作为独立的计算节点；  
使用 MongoDB 作为 Ceilometer 的默认后端数据库；  
将 Murano 和 Sahara 升级到最新版本；  
Fuel 功能的增强；  

## 社区跟踪
### Common
Oz制作CentOS镜像，来自陈沙克  
<http://www.chenshake.com/oz-making-centos-mirror/>

为OpenStack搭建高可用RabbitMQ集群  
<http://eccp.csdb.cn/blog/?p=400> 

来自TryStack的Rally的简介：  
<http://prajnagarden.com/openstack/2014/05/06/rally-guide-01/?bsh_bid=401510023>

来自官方的OpenStack 2014 用户调查解析  
<http://www.openstack.cn/wp-content/uploads/2014/05/OpenStack2014UserSurveyFromOpenStackCN.pdf>

Atlanta峰会的官方视频已经放出：<https://www.youtube.com/watch?list=UUQ74G2gKXdpwZkXEsclzcrA&awesm=awe.sm_iKVqq&v=mNg2-tOFsGQ>

峰会期间，董事会的非正式会议纪要：  
<http://blogs.gnome.org/markmc/2014/05/17/may-11-openstack-foundation-board-meeting>

在Java领域如何与OpenStack互操作？可以考虑使用[Apache jclouds](http://jclouds.apache.org/)。  
<http://filippogaudenzi.blogspot.com/2014/05/jclouds-apis-how-to-deploy-vm-on.html>

### Nova
IBM将 IBM Power virtualization management接入OpenStack的插件托管到了Stackforge上。  
PowerVC driver repository stackforge:  
<https://github.com/stackforge/powervc-driver>  
PowerVC driver project on launchpad:  
<https://launchpad.net/powervc-driver>

### Neutron
OpenStack OVS GRE/VXLAN网络详解  
<http://blog.sina.com.cn/s/blog_6de3aa8a0101pfgz.html>

### Ceilometer
Openstack ceilometer 宿主机监控模块扩展，来自[ECCP企业级云计算平台](http://eccp.zedata.cn)   
<http://eccp.csdb.cn/blog/?p=415>

Openstack ceilometer监控项扩展，同样来自[ECCP企业级云计算平台](http://eccp.zedata.cn)：  
<http://eccp.csdb.cn/blog/?p=352>

### Keystone
PKI类型的token比UUID的token减少了Keystone的负担，但同时也带来了一些不便，比如随着catalog的增加，token变得越来越长。也许，压缩是一种比较好的方法：  
<http://adam.younglogic.com/2014/02/compressed-tokens/>  
<https://blueprints.launchpad.net/keystone/+spec/compress-tokens>