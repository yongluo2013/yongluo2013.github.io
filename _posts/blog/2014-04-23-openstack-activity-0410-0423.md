---
layout: post
title: OpenStack社区动态第十二期(0410-0423)
description: OpenStack社区动态第十二期(0410-0423)
category: blog
---

声明：  
本动态跟踪系列由华为OpenStack团队出品，由孔令贤整理，欢迎转发，但请保留原作者信息!  
新浪微博：[@孔令贤HW](http://weibo.com/lingxiankong)；  
博客地址：<http://lingxiankong.github.io/> 

## 业界动态
如何运营一个像OpenStack这样的开源项目？  
<http://opensource.com/business/14/4/governance-openstack>

Mirantis最近可以说是好事成双，在与爱立信签订三千万美元大单后，马上与Parallels公司完成其OpenStack[第二大单](http://www.businesscloudnews.com/2014/04/09/mirantis-scores-iaas-deal-with-parallels-in-second-openstack-win/)，云计算服务提供商Parallels Automation将与Mirantis OpenStack进行整合以提供OpenStack IaaS服务给他们的客户，新的合作将使Parallels公司以更快的速度部署新的云服务。

清野先生读书期间也不忘发布开源云平台的对比报告啊，报告中第一次对huawei引用了多次  
<http://www.qyjohn.net/?p=3528>

4.17号，Canonical发布了Ubuntu 14.04 LTS，集成了OpenStack最新版本Icehouse

4.15号，Oracle发布两款云存储产品，[ Oracle Database Backup](https://cloud.oracle.com/database_backup)和Oracle Storage Cloud(提供与Swift兼容的对象存储服务)

## 社区跟踪
### Common
4.12号，因为出现了一些bug和安全相关的问题，Glance, Swift又相继发布了RC2版本。

目前OpenStack尚未对安全问题有一个权威的指导或说明，官方只提供了一个security guide，但对于对安全要求严格的主业来说参考的价值不大，因此，来自redhat的Nathan Kinder通过一篇[博客](http://blog-nkinder.rhcloud.com/?p=51)介绍了OpenStack中的安全审计需要做的事情，同时在ML中也开展了一些[讨论](http://openstack.markmail.org/thread/l6nbypuxpazlrdfh#query:+page:1+mid:5bf5s2lxa24ylzo4+state:results)，但从目前来看，响应者并不多。

4.15号，[Cinder](https://launchpad.net/cinder/icehouse/icehouse-rc3), [Ceilometer](https://launchpad.net/ceilometer/icehouse/icehouse-rc3)相继发布了RC3版本；

4.17号，OpenStack 2014.1 ("Icehouse")发布，该版本包含了一个新的组件Trove，共实现了350个特性，修复3000多个bug。Release Notes:  
<https://wiki.openstack.org/wiki/ReleaseNotes/Icehouse>

4.11号，社区各个项目的PTL竞选结果出炉：  
* Compute (Nova): Michael Still  
* Object Storage (Swift): John Dickinson  
* Image Service (Glance): Mark Washenberger    
* Identity (Keystone): Dolph Mathews  
* Dashboard (Horizon): David Lyle  
* Networking (Neutron): Kyle Mestery  
* Block Storage (Cinder): John Griffith  
* Metering/Monitoring (Ceilometer): Eoghan Glynn  
* Orchestration (Heat): Zane Bitter  
* Database Service (Trove): Nikhil Manchanda  
* Bare metal (Ironic): Devananda van der Veen  
* Common Libraries (Oslo): Doug Hellmann  
* Infrastructure: Jim Blair  
* Documentation: Anne Gentle  
* Quality Assurance (QA): Matt Treinish  
* Deployment (TripleO): Robert Collins  
* Devstack (DevStack): Dean Troyer  
* Release cycle management: Thierry Carrez  
* Queue Service (Marconi): Kurt Griffiths  
* Data Processing Service (Sahara): Sergey Lukjanov  
* Key Management Service (Barbican): Jarret Raim

再次讨论VMware和OpenStack的关系，[Going Hybrid With vSphere And OpenStack](http://www.rackspace.com/blog/going-hybrid-with-vsphere-and-openstack/)

再发两个收集OpenStack开发信息的网站：  
<http://activity.openstack.org/data/display/OPNSTK2/OpenStack+Community+Insights>  
<http://activity.openstack.org/dash/browser/>

跟OpenStack无关，但有参考价值：  
[如何参与一个GitHub开源项目](http://mp.weixin.qq.com/s?__biz=MjM5MzA0ODkyMA==&mid=200909764&idx=1&sn=5184c6637977a94916508379b194f3e0)

### Tempest
之前提到过Tempest要取消分支，设计初稿已经出炉：  
<http://docs-draft.openstack.org/77/86577/2/check/gate-qa-specs-docs/3f84796/doc/build/html/specs/branchless-tempest.html>  

### Nova
Michael Still被推举为Nova项目的PTL，4.14号他通过maillist发布了自己对于未来Juno版本的想法：  
1、在开发周期过半时开个例会  
2、加强对feature specs的review  
3、导师制度如何运作  
4、本次峰会议题的审核  
5、鼓励core reviewer积极对bug fix进行检视，提高J版本的代码质量

### Heat
Heat在设计实现过程中遇到过很多问题，代理用户操作便是其中一个，那么Heat是如何解决的？中间经历了哪些讨论？  
<http://hardysteven.blogspot.co.uk/2014/04/heat-auth-model-updates-part-1-trusts.html>  
<http://hardysteven.blogspot.com/2014/04/heat-auth-model-updates-part-2-stack.html>

Heat中的Plugin-in机制：  
<http://docs.openstack.org/developer/heat/pluginguide.html>

还记得Docker么？在Havana，Docker作为Nova的driver实现，但从OpenStack Icehouse开始，Docker将与Heat集成  
<http://www.openstack.cn/p1423.html>

### Ceilometer
如何在Ceilometer进行监控项扩展  
<http://eccp.csdb.cn/blog/?p=352>

### Neutron
增加公网IP段的非常规方法：  
<http://blog.zhaw.ch/icclab/floating-ips-management-in-openstack/>

OpenStack Neutron网络状态详解:  
<http://www.openstack.cn/p1321.html>

继Nova采用spec review的方式审核bp，Neutron团队也决定使用该方式  
<https://wiki.openstack.org/wiki/Blueprints#Neutron>

### Sahara
Sahara是OpenStack社区推出的大数据云化项目，回答了OpenStack与Hadoop这两个炙手可热的开源项目如何融合的问题。章宇兄（ 新浪微博ID：@一棹凌烟 ）刚刚撰写的“Sahara浅析”系列文章对Sahara进行了较为全面的介绍，是目前为止了解Sahara最好的参考，没有之一。特此推荐: <http://yizhaolingyan.net/?p=134>



