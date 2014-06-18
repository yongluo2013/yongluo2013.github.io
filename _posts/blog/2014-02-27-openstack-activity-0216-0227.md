---
layout: post
title: OpenStack社区动态第九期(02.15-02.27)
description: OpenStack社区动态第九期(02.15-02.27)
category: blog
---

声明：  
本周报由华为OpenStack团队出品，由孔令贤整理，欢迎转发，但请保留原作者信息!  
新浪微博：[@孔令贤HW](http://weibo.com/lingxiankong)；  
博客地址：<http://lingxiankong.github.io/>  

## 业界动态
2.10号，大数据公司Hortonworks和Red Hat建立战略合作伙伴关系，Red Hat会将Hortonworks Data Platform (HDP) 接入自己的存储产品，并且HDP也会与Red Hat Enterprise Linux OpenStack Platform整合以降低成本。  
<http://www.redhat.com/about/news/press-archive/2014/2/hortonworks-and-red-hat-deepen-strategic-alliance>

全球领先的智能互联系统嵌入式软件提供商风河®公司近日宣布，风河已经成为OpenStack基金会的企业赞助商。作为OpenStack社区的一份子，风河将帮助推动这项平台技术在电信级云计算基础设施中的应用普及，同时与基金会其它成员积极协作，在各个行业推广开源云计算基础设施的发展。

SUSE宣布正式发布[SUSE Cloud 3](https://www.suse.com/zh-cn/promo/susecloud3.html)，它是企业级OpenStack发行版的最新版本，用于构建基础设施即服务（IaaS）私有云。SUSE Cloud 3 集成了VMware vCenter Server，可以完全支持VMware vSphere，因而能让客户非常灵活地、以高性价比、在现有的虚拟数据中心部署私有云。其安装框架使用Crowbar

2.24号，东软集团已与阿里巴巴旗下阿里云结盟，将逐渐把传统IT服务迁移到阿里云的云计算平台上。这意味着阿里云一举击败IBM、甲骨文等国外厂商，成功拿下国内最大的IT解决方案与服务提供商。

阿朗（Alcatel-Lucent）在其CloudBand中选择了Red Hat Enterprise Linux OpenStack Platform作为云平台，实现NFV。此前，在NFV领域，RedHat已经与Dell合作，因为Dell是[CloudNFV](http://www.cloudnfv.com/)的领军人物。

VDI厂商[Virtual Bridges](http://vbridges.com/2014/02/18/virtual-bridges-announces-sponsorship-openstack-foundation)作为OpenStack企业赞助商，其声称已经可以提供基于OpenStack的桌面云服务。

我们都知道OpenStack是兼容AWS EC2 API的（虽然在兼容程度上不太理想），提供OpenStack私有云服务的各家厂商也都直接利用这个所谓的优势。记得在[第三期](http://lingxiankong.github.io/blog/2013/12/05/openstack-report-1128-1205/)曾提过Google推出了GCE，2.19号，CloudScaling宣布其私有云解决方案OCS支持GCE API，并将其贡献到[StackForge](http://github.com/stackforge/gce-api)。

## 社区跟踪
### Common
关于OpenNebula和OpenStack的对比，已经不是什么新话题了，OpenNebula的工程师也觉得有必要系统的澄清一下，"While OpenNebula is an open-source effort focused on user needs, OpenStack is a vendor-driven effort"  
<http://opennebula.org/opennebula-vs-openstack-user-needs-vs-vendor-driven>

对于刚加入OpenStack贡献的新手来讲，如果配置不慎，估计都会碰到git review -s的问题，其中之一便是“Permission denied (publickey).”，解决办法如下：  
<http://blog.phymata.com/2014/02/12/fixing-my-openstack-gerrit-problem/>

因为OpenStack相对灵活的安装部署，大部分的贡献者都会在自己的个人主机上使用虚拟机搭建OpenStack开发环境，网上的文档有很多，看一下Rackspace的工程师是怎么玩的？  
<http://www.rackspace.com/blog/inside-my-home-rackspace-private-cloud-openstack-lab-part-1-the-setup/>  
<http://www.rackspace.com/blog/inside-my-home-rackspace-private-cloud-openstack-lab-part-2-the-networking/>  
<http://www.rackspace.com/blog/inside-my-home-rackspace-private-cloud-openstack-lab-part-3-installing-a-high-availability-rackspace-private-cloud-with-chef-cookbooks/>  
<http://www.rackspace.com/blog/inside-my-home-rackspace-private-cloud-openstack-lab-part-4-neutron/>  
<http://www.rackspace.com/blog/inside-my-home-rackspace-private-cloud-openstack-lab-part-5-adding-extra-compute-nodes/>

Icehouse发布在即，一些新的项目已经为Juno做准备，比如Murano，正在申请进入Juno的孵化项目。但Murano的境遇比较尴尬，因为其实它同时覆盖了Glance和Heat的功能，而OpenStack是不允许出现项目范围重复的。    
<https://wiki.openstack.org/wiki/Murano/Incubation>

今天(2.24号)看maillist时发现了一个新项目["PythonOpenStackSDK"](https://wiki.openstack.org/wiki/PythonOpenStackSDK)，目的是提供一个与OpenStack服务统一交互接口

OpenStack社区专为女性的扩展项目开始了，为期3个月（3-5）。报名地址（我相信国内应该没有女性关心吧）：<https://wiki.openstack.org/wiki/OutreachProgramForWomen>

还记得第六期提到的'DefCore'么？Mirantis又抢先一步，发布了[RefStack](http://refstack.org/)，从其信息尚未完善的主页来看，参与的厂商挺多。

如何参与翻译工作？这里有一些提示。  
<http://vmartinezdelacruz.com/make-openstack-speak-your-language-join-openstacks-i18n-community>

### Nova
2.27号，python-novaclient 2.16.0 released，修复的bug列表参见：  
<https://bugs.launchpad.net/python-novaclient>

OpenStack实现AWS的VPC功能，社区早就有人提过，一拖再拖，现在终于又被raised up。bp链接：  
<https://blueprints.launchpad.net/nova/+spec/aws-vpc-support>

Gantt（独立出Nova的Scheduler）项目目前还在开发中，后续可能会托管到stackforge，并且还需要至少一个development cycle

V3 API目前的开发状态：<https://etherpad.openstack.org/p/NovaV3APIDoneCriteria>，社区也在讨论Icehouse中是否要正式发布稳定版V3 API，但普遍认为，V3的特性并非那么的引人瞩目，并且V3 API也缺乏对nova-network的支持（后者在Icehouse并没有废弃），同时，V3 API还有很多工作要做。值得说明的是，两位来自IBM的童鞋对V3 API的实现做了很大贡献。V3 API目前比较尴尬的原因是，它并不兼容V2版本。或许，V2还会持续相当长的时间。  
Future of the Nova API:  
<http://lists.openstack.org/pipermail/openstack-dev/2014-February/027896.html>

另一种通过VNC连接VM的方式：<http://lizards.opensuse.org/2014/02/08/another-way-to-access-a-cloud-vms-vnc-console/>

众所周知，OpenStack兼容AWS EC2接口，但是EC2接口的测试，可能会用到相应的工具，euca2tools便是其中一个。关于euca2tools的安装其实不是什么难事，但有个指导总是好的。  
<https://www.pdc.kth.se/resources/computers/pdc-cloud/euca2ools-for-openstack>

Hyper-V对的OpenStack的支持越来越好，这个cloudbase团队确实比较牛逼，这不，Havana 2.2版本刚出来，对应Hyper-V自动化安装工具就[出来](http://www.cloudbase.it/openstack-havana-2013-2-2-hyper-v-compute-installer-released/)了，一键式啊~

看来在Nova中实现VM HA（非业务层面HA）是基本无望了，相关的讨论如下：  
<http://lists.openstack.org/pipermail/openstack-dev/2014-February/027753.html>  
<http://markmail.org/message/5zotly4qktaf34ei>

其实之前我是比较关注Nova是否会提供类似DRS的功能。这个话题以前讨论过，但都没有一个结论。而且社区有相关的项目（比如Gantt），社区外也有相关的实现（<http://openstack-neat.org/>）。但，最近一个新的相关的[bp](https://blueprints.launchpad.net/nova/+spec/resource-optimization-service)又出现了，我会持续跟踪进展。

### Heat
2.19, python-heatclient 0.2.7 released,该版本使用了requests库，而不是采用httpclient实现。  
<https://launchpad.net/python-heatclient/+milestone/v0.2.5>

目前对于heat的stack-update处理方式：如果update失败，在回滚使能的情况下，heat会尝试回滚到之前状态，但是如果回滚失败或者不允许回滚的情况下，heat没有处理保留failed的状态，用户此时就只能删除stack。有人提出一个bp解决该问题：https://blueprints.launchpad.net/heat/+spec/update-failure-recovery

### Keystone
OpenStack中关于Keystone的文档应该有很多，但还是有一些东西貌似文档没有涉及（比如命令行中使用debug，创建用户得不到密码等），但很多人经过实践摸索，形成了非官方的“共识”，这里有一篇[博客](http://mirandazhangq.wordpress.com/2014/02/10/wish-list-common-misunderstanding-undocumented-openstack-identity-api-authentication-add-user/)做了大概的说明。

### Horizon
OpenStack的界面到底设计的如何？只有对比过后才知道。下面文章的作者应该是Horizon的UI设计人员，文章中对比了Netflix Asgard, Windows Azure, PayPal Aurora, CloudStack, Google Compute Engine, Amazon Web Services，信息量还是很大的。  
<http://uxd-stackabledesign.rhcloud.com/?p=78>
