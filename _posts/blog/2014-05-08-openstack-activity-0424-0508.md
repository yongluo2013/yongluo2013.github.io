---
layout: post
title: OpenStack社区动态第十三期(0424-0508)
description: OpenStack社区动态第十三期(0424-0508)
category: blog
---

声明：  
本动态跟踪系列由华为OpenStack团队出品，由孔令贤整理，欢迎转发，但请保留原作者信息!  
新浪微博：[@孔令贤HW](http://weibo.com/lingxiankong)；  
博客地址：<http://lingxiankong.github.io/> 

## 业界动态
Symantec加入OpenStack企业赞助   
<http://www.symantec.com/connect/blogs/symantec-joins-openstack-foundation>

4.29号，Oracle发布了集成OpenStack的Solaris 11.2

4.30，Redhat以1.75亿美元收购inktank公司，inktank的主要产品是软件定义存储ceph解决方案。ceph本身是开源，基于一个统一的对象存储系统，同时对外提供块、文件和对象接口，其中对象接口兼容Swift和Amazon S3。对象接口和文件接口的数据是拉通访问的，即，用对象接口存储的数据，可以直接用共享文件系统访问。这是因为ceph的文件接口本身是建立在对象存储之上的。完成收购之后，Ceph之前为企业用户提供的一些诊断和管理工具将会很快被开源，Ceph将会变得更加开放。

4.30，RightScale发布了针对vSphere的云管理工具，利用该工具，可同时兼容vSphere, AWS, Google, Azure, OpenStack, and other clouds

[Talligent](http://talligent.com/) 公司将在OpenStack Atlanta峰会上发布 OpenBook 2.0 ，OpenStack的计费和消费生命周期管理的解决方案

又一款兼容多种云基础设施的云管理和监控工具：[traverse](http://www.kaseya.com/solutions/traverse)

## 社区跟踪
### Common
看看Mirantis对于过去Icehouse版本的总结和未来的展望，不得不说，这家创新、务实的公司在OpenStack领域应该受到越来越多的重视。  
<http://www.mirantis.com/blog/icehouse-community-investment-report>

Openstack配置文件管理的变迁之路  
<http://www.cnblogs.com/yuxc/p/3650660.html>

OpenStack要满足企业级交付，还有多远的路要走？请看分析：  
<http://cloudscaling.com/blog/openstack/the-6-requirements-of-enterprise-grade-openstack-part-1>  
<http://www.cloudave.com/34681/6-requirements-enterprise-grade-openstack-part-2>

HP和Intel在社区推出[Graffiti](https://wiki.openstack.org/wiki/Graffiti)项目，为不同项目的资源打标签并且做到快速搜索的目的，这里有一个POC视频：<http://youtu.be/f0SZtPgcxk4>

OpenStack Operation Guide出新版本了，<http://docs.openstack.org/ops/>

Gerrit升级后，如何更有效的进行查询？  
<https://dague.net/2014/04/30/helpful-gerrit-queries-gerrit-2-8-edition/>

### Nova
Nova对EC2的支持不太完善，而且越来越得不到大家的关注，关于EC2的问题处理的也不及时，社区就此展开讨论，Tempest和Nova的Core: Sean Dague在Maillist发起求助，获得了Reliance公司的Rushi Agrawal响应，他表示他们已经做了很多完善的工作，也在社区提了相关的bp，但因为不受关注被推迟到Juno。但可以预见，如果社区确实对EC2有决心维护和完善，在Juno版本，关于EC2的工作应该比较容易接纳。  
<http://openstack.markmail.org/thread/6u23rqvyccxpnl32#query:+page:1+mid:vw5pzf566v4hxfns+state:results>

如何删除主机信息？以往，Nova中某台计算节点出问题时，Nova会通过状态显示。但无法删除该节点记录。  
<http://blog.zhaw.ch/icclab/managing-hosts-in-a-running-openstack-environment/>

### Neutron
一篇讲Neutron中的数据流的电子书:  
<http://www.hastexo.com/system/files/neutron_packet_flows-notes-handout.pdf>

RedHat内部的PPT，Introduction to Neutron:  
<http://assafmuller.files.wordpress.com/2014/05/neutron.pdf>

### Ceilometer
Alexandre Viau提出一个[bp](https://blueprints.launchpad.net/ceilometer/+spec/monitoring-as-a-service)，增加一个监控服务项目，社区部分人认为监控应该放到Ceilometer的范畴，但一部分人否定。HP内部也实现了监控服务Jahmon，并打算在Atlanta峰会上介绍。目前的讨论是大家到峰会上面对面讨论。

Nagios和Ceilometer的集成：  
<http://blog.zhaw.ch/icclab/nagios-ceilometer-integration-new-plugin-available/>

### Heat
在Icehouse版本，Heat到底有哪些新的特性？前Heat的PTL通过一个视频告诉你：  
<https://plus.google.com/u/0/events/ckhqrki6iepg12vkqk5vnt7ijd0>

IBM的 Thomas Spatzier 准备在Atalanta峰会上主持一个session，讨论[heat-translator](https://github.com/stackforge/heat-translator)项目，适配Heat对TOSCA的支持。

一个简单的指定虚拟机创建顺序的使用教程，基于Havana版本，包含一个问题的解决：  
<http://blog.zhaw.ch/icclab/manage-instance-startup-order-in-openstack-heat-templates/>