---
layout: post
title: OpenStack社区动态第八期(02.03-02.15)
description: OpenStack社区动态第八期(02.03-02.15)
category: blog
---

声明：  
本周报由华为OpenStack团队出品，由孔令贤整理，欢迎转发，但请保留原作者信息!  
新浪微博：[@孔令贤HW](http://weibo.com/lingxiankong)；  
博客地址：<http://lingxiankong.github.io/>  

*编者按*  
由于过年的原因，跟踪不是很及时，各个模块的内容明显偏少，后续会补充到下期周报，请朋友们谅解。

## 业界动态
Cisco发布InterCloud，为企业提供私有云与公有云之间的迁移，同时支持不同类型的云，可以是基于OpenStack，也可以是AWS，甚至可以是 Microsoft Azure。InterCloud与Red Hat的CloudForm集成。在1月底米兰的展示会上，除了InterCloud，Cisco还发布了其他几个产品/服务：  
1、Cisco Services For OpenStack：帮助客户了解并部署OpenStack，包括：strategy and assessment; validation; design and deployment; software and application platform integration; and optimization  
2、Cisco Intelligent Automation for Cloud (IAC) 4.0：a comprehensive cloud management solution for enterprises and cloud service providers to deploy private, public, or hybrid clouds  
3、Cisco Validated Design (CVD) for Cisco/Red Hat integrated Cloud Management and Provisioning

模块化数据中心行家IO公司推出一个名为IO.Cloud的新服务，正式进入云计算供应商业务。IO.Cloud用到开放计算（Open Compute）服务器设计，运行的云计算操作系统是OpenStack。IO为IO.Cloud定下的基调是企业云产品。如果IO意欲与较大的云服务提供商争夺企业云的工作量。

RedHat上周动作不大，仅仅是发布了基于GlusterFS的Red Hat Storage Server支持增强的数据保护和卷快照，同时支持检查点（checkpoints）和虚拟机恢复

开源云服务 Docker 拿到了 1500 万美元B 轮融资，Docker将利用新融资推进Docker环境的通用性，开发与其开源技术配对的商业服务，并组建团队支持不断扩大的社区。Docker目前是世界上发展最快的开源项目之一，该社区帮助Docker获得了红帽公司的认可，后者正将Docker产品整合到它的PaaS（平台即服务）环境OpenShift。Docker还被谷歌计算引擎（Google Compute Engine）采用。eBay、Yandex和众多其它的公司都在生产环境中使用Docker。

IBM开发出一款工具，以解决云管理生态系统当中的碎片化问题，并希望其方案能够吸引更多供应商在无需运行OpenStack的前提下支持基于OpenStack的应用程序。IBM于2013年六月收购SoftLayer之时，随即发现了OpenStack在发展中面临的最大问题：SoftLayer并没有使用OpenStack技术，而且现有云控制层的开发工作早已进行了数年之久。[JumpGate](https://github.com/softlayer/jumpgate)通过为现代OpenStack模块提供API兼容性的方式解决问题，同时将命令转化为可插入库、从而将其转译为能够被其它非OpenStack或者碎片化OpenStack云所能接受的API语义。

2.10号，HP推出其基于OpenStack的混合云解决方案[HP Cloud OS](http://www8.hp.com/us/en/business-solutions/solution.html?compURI=1421776#.UvhTQ2KSyKI)，为企业用户提供安装升级、增强的服务生命周期管理以及混合云场景下的业务迁移。

Oracle自去年12月份加入OpenStack企业支持后，一直没有动静。但就在2.6号，Oracle与Pluribus Networks宣布[合作](http://www.oracle.com/us/corporate/press/2132552)，为现有的 Oracle Solaris 和Pluribus Networks’Netvisor以插件形式接入OpenStack

你知道的OpenStack发行版都有哪些？这篇文章告诉你。  
<http://yourstory.com/2014/02/6-openstack-distributions-help-enterprises-jumpstart-private-cloud-deployment/>  
有几个我还是是第一次听说，比如[Piston OpenStack](http://www.pistoncloud.com/openstack-cloud-software/technology/)

## 社区跟踪

### Common
havana版本2013.2.2于2.13号发布，release notes如下：  
<https://wiki.openstack.org/wiki/ReleaseNotes/2013.2.2>  
下一版本2013.2.3预计于4.3号发布

社区对于项目的孵化和集成规定了一系列[要求](https://github.com/openstack/governance/blob/master/reference/incubation-integration-requirements)，2.6号几个PTL讨论，准备着手对现有的项目进行重新审视以使其达到要求。  
<http://lists.openstack.org/pipermail/openstack-dev/2014-February/026450.html> 

2.4号，victor.stinner@enovance.com   提出了一个建议，replace eventlet with asyncio，引起了大家的关注和讨论。对于技术细节我不懂，感兴趣的读者可以参考下面的链接：  
<http://lists.openstack.org/pipermail/openstack-dev/2014-February/026237.html>

如何使用各个组件的client？社区有人提出，使用client编写代码访问OpenStack时比较困难，且没有指导文档。不过有人给出了非官方的指导可以暂作参考：  
<http://python-api-guide.cfapps.io/content/neutron.html>  
source: <https://github.com/rajdeepd/openstack-samples>

我们知道OpenStack每个工程中的每个文件都有License声明，但这些声明是不是必要的呢？OpenStack其实参照了Apache社区：  
License headers allow someone examining the file to know the terms for
the work, even when it is distributed without the rest of the
distribution. Without a licensing notice, it must be assumed that the
author has reserved all rights, including the right to copy, modify,
and redistribute."

新手的福音，Sandy Walsh（Rackspace）提出为不熟悉OpenStack社区参与方式的人，提供一种游戏化（gamification）的方式来引导，最终使其更顺利的参与社区。gamification可以参考:  <http://gamify.com/>  
<https://badges.fedoraproject.org/>

### Heat
增加一个DB的migration用来修复现存的HOT模板，原因是从Havana到Icehouse的一些修改可能导致之前的模板不可用，比如HOT模板之前对于parameter的类型定义可以为“String”，但是在I版的一些修改引入后，heat对于HOT模板的parameter的类型定义支持“string”，而不是“String”，类似这样的问题，社区讨论的结论是用数据库的迁移来解决。并且这将是heat的第一个db的migration。另外指出HOT目前还处于大量开发的阶段，随时会变化，如果对稳定性要求高的应用建议使用cfn模板，参考release  notes:  https://wiki.openstack.org/wiki/ReleaseNotes/Havana#Initial_support_for_native_template_language_.28HOT.29

I版的一个[bp](https://bugs.launchpad.net/heat/+bug/1274201)（heat_keystoneclient只支持keystone v3版本）合入导致Rackspace的认证被破坏，因为Rackspace目前只支持keystone的v2版本认证，这个问题对于只支持keystone v2的第三方插件都有影响。对于问题的修复方案目前仍在讨论。