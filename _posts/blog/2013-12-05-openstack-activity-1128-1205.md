---
layout: post
title: OpenStack社区动态第三期(11.28-12.05)
description: 来自华为OpenStack社区团队出品的周报
category: blog
---

声明：  
本博客由华为OpenStack团队出品，欢迎转发，但请保留原作者信息!  
新浪微博：[@孔令贤HW](http://weibo.com/lingxiankong)；  
博客地址：<http://lingxiankong.github.io/>  
内容系本人及本人团队学习、研究和总结，如有雷同，实属荣幸！

## 业界动态
欧洲最大的Openstack会议----action! [conference](http://www.eventbrite.com/e/openstack-in-action-4-tickets-7645801799)于12.5召开，

Gartner的数据中心会议  
 会议将深度覆盖与数据中心相关的移动、云、存储和其他IT变革议题。  
 <http://www.gartner.com/technology/summits/na/data-center/>

近日，一篇来自Gartner的文章[《Why vendors can’t sell OpenStack to enterprises》](http://blogs.gartner.com/alessandro-perilli/why-vendors-cant-sell-openstack-to-enterprises/)一石激起千层浪，列举了openstack在企业市场所面临的四个挑战，文章对openstack的未来发展持消极态度。然后RedHat立即对该文章进行了[回应](http://tentenet.net/2013/11/21/the-4th-tenet-of-openstack-open-source-projects-are-not-the-same-as-products/)，但该回应更像是一种对RedHat的宣传，其实对于openstack，大家仁者见仁，智者见智，如何基于开源赢得商业上的成功，不得不承认，RedHat确实给我们树立了榜样。  
在RedHat的文章中，有一些信息值得我们注意：  
1、RedHat在考虑开源其云管理平台CloudForms  
2、RedHat不会发布基于OpenStack的商业解决方案，所以不同的厂商才有与RedHat合作的机会  
3、从RedHat对openstack社区的贡献来看，主要聚焦于Heat, TripleO, Tuskar等新兴项目

这条跟OpenStack无关，但毕竟是IaaS界的一个爆炸新闻，所以拿出来分享一下。Google正式推出[GCE](http://googlecloudplatform.blogspot.com/2013/12/google-compute-engine-is-now-generally-available.html)(Google Compute Engine)，在GAE失败之后，Google终于大梦初醒。GCE最大的亮点应该是支持热迁移，故障重启以及维护模式，不知GCE的推出，会不会改变IaaS的格局呢？

## 社区跟踪

### Neutron
各个厂家积极参与LBaaS的Plugin开发：Brocade、Embrane、ArrayNetworks APV、LVS、Citrix NetScaler  
<https://blueprints.launchpad.net/neutron/+spec/neutron-brocade-lbaas-driver>  
<https://blueprints.launchpad.net/neutron/+spec/lbaas-array-apv-driver>  
<https://blueprints.launchpad.net/neutron/+spec/lbaas-lvs-driver>  
<https://blueprints.launchpad.net/neutron/+spec/embrane-lbaas-driver>  
<https://blueprints.launchpad.net/neutron/+spec/netscaler-lbaas-driver>    
目前，除了Brocade、Embrane都处于开始阶段

关于LBaaS使用虚拟机提供服务的BP，也就是LBaaS的系统虚拟机：  
<https://blueprints.launchpad.net/neutron/+spec/lbaas-service-instance>  
<https://wiki.openstack.org/wiki/Neutron/LBaaS/LoadbalancerInstance>

### Tempest
目前tempest中的一些用例由于种种原因是不跑的，skip。而对于skip的用例目前有两种方式："writing  cls.skipException statement in setUpClass method" or "skipIf annotation for each test method" .现在在考虑合并。

### Common
在没有获取别人允许下，尽量不要在IRC中找人私聊（当然，你跟他关系不错是另一说），review request也好，咨询问题也好，都尽量发到公共频道。不然会让一些Core心里不舒服的。  
<http://lists.openstack.org/pipermail/openstack-dev/2013-November/020747.html>

OpenStack Icehouse版本将尝试引入 [AngularJS](blueprint: https://blueprints.launchpad.net/horizon/+spec/angularjs-javascript-frontend-framework)统一 Frontend Javascript Framework，前端同学们对OpenStack社区也可以有贡献了

目前社区的代码编写和riview在对live upgrade的支持上还是欠缺考虑：  
<http://lists.openstack.org/pipermail/openstack-dev/2013-November/020534.html>

在OpenStack一路疯狂之后，社区对于新加入的事物开始谨慎起来，包括之前说的各个部件的driver接入，新的blueprint提交等，而目前，对于新的项目申请，也开始制定[约束](https://review.openstack.org/#/c/59454/)。

12.5号，社区创建了’icehouse-1’的Milestone-proposed分支，目前为止，实现了 69 blueprints and fixed 738 bugs，目前处于公测阶段。  
12.12号即将发布stable/havana 2013.2.1

### Nova
目前在Nova中有一些针对虚拟机的操作在admin_actions中，尚不属于标准API。社区在V3版本API中打算将这些API作为Core API。  
<http://lists.openstack.org/pipermail/openstack-dev/2013-December/020767.html>

将nova-scheduler独立出一个[服务](https://blueprints.launchpad.net/nova/+spec/remove-cast-to-schedule-run-instance)，路标是在I版就不建议使用，在J版完全废弃。

### Heat
Rackspace率先发布了Heat商店：[Heater](https://wiki.openstack.org/wiki/Heat/htr)




