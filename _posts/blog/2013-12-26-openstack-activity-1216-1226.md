---
layout: post
title: OpenStack社区动态第五期(12.16-12.26)
description: 来自华为OpenStack社区团队出品的周报
category: blog
---

声明：  
本博客由华为OpenStack团队出品，欢迎转发，但请保留原作者信息!  
新浪微博：[@孔令贤HW](http://weibo.com/lingxiankong)；  
博客地址：<http://lingxiankong.github.io/>  
内容系本人及本人团队学习、研究和总结，如有雷同，实属荣幸！

## 业界动态
Mirantis于12.12号正式向社区发布Fuel，后续考虑Fuel, TripleO and Tuskar and Heat的结合。

Stackalytics团队于12.12号发布了version 0.4。增加了：  
1. 查询项目的top reviewer，比如<http://stackalytics.com/report/reviews/neutron-group/30>  
2. 帮助core reviewer查询等待较长时间的patch，比如<http://stackalytics.com/report/reviews/nova/open>等

关于基于OpenStack的灾备，RedHat一直很积极  
<http://redhatstackblog.redhat.com/2013/11/26/disaster-recovery-enablement-in-openstack/>

近日，在戴尔全球企业用户大会上戴尔[宣布](http://www.theregister.co.uk/2013/12/16/dell_red_hat_openstack_team/)与RedHat达成合作伙伴关系，两家公司将联合向企业用户销售OpenStack。该产品将于明年年初上市，主要是面向构建私有云系统的大型企业。该系统将采用戴尔的硬件和下一代RedHat企业Linux OpenStack平台。在代码贡献方面，两家企业将在Neutron和Ceilometer组件合作。Neutron方面会针对Dell的网络硬件和虚拟网卡，Ceilometer对聚焦服务监控和计费。

12月18日凌晨消息，亚马逊公有云服务AWS(Amazon Web Services)已经与宽带资本旗下云基地达成[战略合作](http://aws.amazon.com/cn/about-aws/whats-new/2013/12/18/announcing-the-aws-china-beijing-region/)，实现了AWS在中国的正式落地。双方在宁夏建立数据中心，但是运营中心放在北京。同时，亚马逊宣布，继悉尼、新加坡、东京之后，北京成为亚太地区第四个数据中心，世界第10个数据中心。

红帽宣布Red Hat Enterprise Linux OpenStack Platform 4.0稳定版[正式发布](http://www.redhat.com/about/news/archive/2013/12/red-hat-announces-general-availability-of-red-hat-enterprise-linux-openstack-platform-4)，包括红帽对 OpenStack Havana 代码的加强以及红帽企业 Linux 6.5 和红帽基于 KVM 的企业虚拟化系统管理。Red Hat Enterprise Linux OpenStack Platform 为 IT 基础设施团队、云应用开发者在不影响可用性、安全性和性能上的一个混合云平台。  
Red Hat Enterprise Linux OpenStack Platform 4.0 包含如下特性：  
完全支持 Foreman 物理和虚拟基础设施资源的生命周期管理工具  
完全支持 OpenStack Orchestration (Heat).  
完全支持 OpenStack Networking (Neutron).  
完全支持 OpenStack Telemetry (Ceilometer).  
集成 Red Hat CloudForms.  
增强与红帽存储服务器的集成

## 社区跟踪

### Tempst
怎么在Tempest中测试IPv6？目前可以看到的方式是使用metadata service或使用config drive，但并没有就此达成一致，需要继续跟踪。  
<http://lists.openstack.org/pipermail/openstack-dev/2013-December/021149.html>

### Common
OpenStack有一个[Travel Support Program](https://wiki.openstack.org/wiki/Travel_Support_Program)，是依据Open Design的协议为关键贡献者提供参加开源峰会的免费机票和住宿服务。本次的香港峰会是该项目的第一次运作，共为来自世界各地的18位关键贡献者提供免费服务。

又一个来自Mirantis的项目：[Mistral](https://wiki.openstack.org/wiki/Mistral)，Workflow as a Service。

继香港峰会之后，OpenStack的各个PTL将会召开线上技术讨论会，介绍项目目前的进展和路标
时间：Tuesday and Thursday at 7 a.m. Pacific/10 a.m. Eastern  
<http://www.openstack.org/blog/2013/12/openstack-project-update-webinars/>

2013接近尾声，2014将至，新的一年OpenStack将如何发展? [Mirantis Three OpenStack Predictions for 2014](http://www.mirantis.com/blog/three-openstack-predictions-2014/)提到明年将是OpenStack进军企业级市场的一年，同时随着原生的Paas平台接入，OpenStack生态圈将进一步完整，而看不清趋势的Startup将被清理出竞争之外。

基于时间的资源管理[系统](https://wiki.openstack.org/wiki/Cafe)，来自Cyber Security Lab in University of Waikato的实际需求，刚刚提交到maillist中讨论。  
但该目标与社区的Climate（Reservation-as-a-Service）的范畴吻合，因此大家建议直接参与到Climate的讨论或开发中。

i18n团队完成了fuel UI的本地化，在当前主干和即将发布的4.0版本中都可见到完善的中文界面

Barbican项目的孵化  
Barbican项目为云应用程序提供密钥管理服务，其对称密钥管理可协助Cinder进行数据加密。  
>Symmetric key management - These keys are used for encryption of data at
rest in various places including Swift, Nova, Cinder, etc. Keys are
resources that roll up to a project, much like servers or load balancers,
but they have no direct relationship to an identity.

### Nova
从E版开始Nova就有针对虚拟机诊断信息的查询接口，但不同的driver实现返回的信息不一致，导致无法对该接口进行统一的测试。已经有人提了[bp](https://blueprints.launchpad.net/nova/+spec/diagnostics-namespace)，在V3 API中解决, [wiki](https://wiki.openstack.org/wiki/Nova_VM_Diagnostics)

IBM向其OpenStack部署新增了一个更高级的调度器，这个平台资源调度器最初由Platform Computing公司开发，IBM在2012年收购了这家公司。在OpenStack部署中，这个平台资源调度器可以选择最合适的服务器，并在其中放置新的虚拟机——基于管理员制定的政策。这些政策可以包括每台机器正在使用的CPU利用率、内存利用率等其他因素。该调度器能够与OpenStack内置调度器Nova一起使用。Nova的局限性是，它是基于静态信息来做决定，它只在资源最初被放置时对资源进行调度，它并不知道如何不断优化环境，以及重新调整放置决定。该软件已经集成在了11月份的IBM SmartCloud Orchestrator的技术预览版中，该产品被作为Orchestrator or IBM SmartCloud Entry的增值部件，Orchestrator和IBM SmartCloud Entry都是OpenStack云管理软件。该软件并不会作为单独的产品出售，可以媲美VMware云部署中分布式资源调度器(DRS)的复杂性。

### Ceilometer
有人提出，目前ceilometer默认的轮询周期是10min，对于tempest测试来说这个写在配置文件里的值不够灵活，提出bp可以指定轮询周期测试接口。

### Neutron
Neutron Distributed Virtual Router  
--在实现分布式路由时，L3 agent怎么区分普通的router（centralized）和Distributed，有两种实现，一种是给l3添加新的attribute，一种是使用'provider'（类似network的'provider'属性），暂时还没确定下来，但Neutron的高级服务(LB, FW, VPN)也会定义"provider" 属性，所以采用第一种的可能性很大

### Heat
Q:  
a.Heat的资源只对一个租户可见，是这样设计的吗？  
b.我想给某个租户创建镜像、网络、路由和防火墙规则。我查看了doc文档但是发现没有resource可以创建一个新租户或者上传镜像？  
A:  
a.设计如此，heat stack各资源作用于同一个租户（即创建stack的租户）。  
b.写一个创建租户的resource不会困难，但是在stack的资源创建之前，需要修改你要创建的租户的context。所以现在的话，建议不用heat创建租户和用户。至于镜像上传，可以提交一个bp，用glance resource（用URL注册的镜像资源）来实现。







