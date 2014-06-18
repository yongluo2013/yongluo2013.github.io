---
layout: post
title: OpenStack社区动态第六期(12.26-01.16)
description: 来自华为OpenStack社区团队出品的周报
category: blog
---

声明：  
本博客由华为OpenStack团队出品，欢迎转发，但请保留原作者信息!  
新浪微博：[@孔令贤HW](http://weibo.com/lingxiankong)；  
博客地址：<http://lingxiankong.github.io/>  
内容系本人及本人团队学习、研究和总结，如有雷同，实属荣幸！

## 业界动态
VMWare加入OpenStack社区的意图是什么？（What Is VMware Up To With OpenStack?）
OpenStack被看做是在私有云领域对VMware的挑战者，人们期待她重演在操作系统领域Linux对微软Windows对抗的故事。因此，自从VMWare打算加入OpenStack社区并随后曲折地被批准以来，各种猜测、分析VMWare公司意图的文章和评论一直是OpenStack世界的热门话题之一。本周OpenStack官方blog分享了一篇从技术视角分析的文章，作者是OpenStack架构师 Kenneth Hui，前VMWare vSphere工程师。以Dan Wendlandt为首的VMWare技术团队自从加入社区以来在通过各种方式融入开源社区，也包括他们在香港openstack summit的一些技术活动。从Grizzly版本开始，VMWare开始推动OpenStack对vSphere的支持，从Grizzly到Havana，无论是从代码数量和功能覆盖，VMWare的贡献可以说是稳定和持续，而且会有更多的功能会在Icehouse加入。 Kenneth Hui的看法基于他对VMWare这些一线社区人员的社区贡献、动向的观察，而且可以说没有私心，因为他已经不是VMWare员工了。主要观点见中国社区中文意译<http://www.openstack.cn/p709.html>，详细内容请看英文  <http://cloudarchitectmusings.com/2013/12/18/what-is-vmware-up-to-with-openstack/>

12.28号，Mirantis发布Mirantis OpenStack 4.0: Updated for Havana，在同步代码的同时，加入了对Ceilometer和Heat的支持。另外，将Murano项目版本升级到version 0.4。  <http://software.mirantis.com/>

Mirantis出品的Fuel(<https://launchpad.net/fuel>)于12.28号宣布4.0 GA milestone，代码版本更新到了Havana 2013.2.1，支持Ceilometer和Heat。其他主要更新如下：  
- 其UI部分与99cloud公司合作完成汉化，同时支持其他语言的翻译。  
- Puppet Master server从Master Node Server中移除  
- 可以以IP地址段的方式配置public network  
- 增加了部分网络校验  
- 之前安装OS和OpenStack的部署是在一个动作中执行，新版本将这两个步骤分开  
- 在文档中增加了对NIC Bonding的说明（<http://docs.mirantis.com/fuel/fuel-4.0/reference-architecture.html#advanced-network-configuration-using-open-vswitch>）  

CY13-Q4 OpenStack, OpenNebula，Eucalyptus，CloudStack社区活跃度比较  
<http://www.qyjohn.net/?p=3431>

QingCloud获光速安振中国领投的2000万美元B轮融资，不仅未受AWS进中国的影响，还要打入美国去 | 融资后QingCloud会将资金用于扩大基础设施，建设覆盖华南、华东、北方、西部主要节点部署区，并建设亚太、美西两个重点海外节点，打造一个面向全球的云计算服务平台

来自国内的公有云对比，覆盖了新浪云、百度云、阿里云、腾讯云和青云。  
<http://blog.csdn.net/shaunfang/article/details/10899299>

CentOS 项目已加入红帽公司，作为红帽公司开源和标准团队( <http://community.redhat.com/> ) 的一部分，培养快速创新平台之外的下一代新兴技术。将于 Fedora 和 RHEL 生态系统一起工作，希望通过新的平台进一步扩大社区服务。  
<http://lists.centos.org/pipermail/centos-announce/2014-January/020100.html>

SoftEther VPN宣布开源（<http://www.softether.org/9-about/News/800-open-source>）。恰逢OpenStack的VPNaaS慢慢受到人们关注，让人不得不把这两件事情结合起来。虽然VPNaaS目前还弱爆了，但让不少曾经高高在上的商业产品感到威胁低下头，也算突出贡献。可以预见不少SoftEther VPN先进功能将会被各种方式吸收。

## 社区跟踪

### Common
Icehouse-2 milestone is coming up January 23rd

开源世界一直有着命名的逸闻趣事，OpenStack也不例外。OpenStack按照26个英文字母的顺序逐步为每个版本命名，具体的版本名字的产生方法是：在下一次OpenStack技术大会举办前，由社区人员提名和举办地或国家相关联的一个以下个版本英文字母开头的单词，然后由社区投票产生。比如IceHouse的命名取自香港一条街道——中环雪场街（Ice House Street)。J版本的名字最终定位Juno，他源自下届OpenStack技术大会举办地美国亚特兰大的一首风靡全球的一张电影和专辑《Juno》。

随着OpenStack在全世界范围内的蓬勃发展以及很多公司的参与，OpenStack基金会面临一个如何界定某一家公司的商业产品是否可以被授权使用OpenStack商标的难题。为了保护OpenStack品牌和推动OpenStack商业化的健康发展，基金会在刚刚举办的OpenStack香港技术大会上成立了一个名为“Define Core Committee”的委员会，简称DefCore，来提议什么代表OpenStack Core,以及如何通过技术手段认证一家公司的商业版符合OpenStack Core的标准，从而授权使用OpenStack商标。这件事情意义重大，会避免OpenStack产业出现鱼龙混杂的现象。现在号称兼容OpenStack的云计算产品原来越多，实际上从技术上和OpenStack的API是否兼容、是否采用了OpenStack的code、是否采用了OpenStack推荐的Driver或者Plugin，等等都不清楚，如果没有一个监督和认证机制，长期下去会逐步影响OpenStack的应用，甚至破坏OpenStack生态系统。  
<https://wiki.openstack.org/wiki/Governance/CoreDefinition>

数据库迁移对于OpenStack的开发运和运维人员来说一直是一件头疼的事情。迁移数据需要一定的停机时间，数据库大的话往往需要数个小时，这对于大的OpenStack生产环境是要必须想办法解决的。Openstack社区马上就要推出一个数据库迁移（Migration)测试工具Turbo-Hispster来初步处理这一棘手问题。Turbo-Hispster是一个基于OpenStack CI 框架Zuul的测试工具，他可以帮助技术人员在新的patchset提交时候针对真实的数据库备份来评估迁移需要的时间和可能存在的错误。  
<http://josh.people.rcbops.com/2013/12/third-party-testing-with-turbo-hipster/>  
<http://josh.people.rcbops.com/2013/09/building-a-zuul-worker/>  

Python 3从发布到现在已经5年了，已经足够成熟，新增了许多特性，也做了很多优化，后续也会逐渐放弃对Python2.7的维护。目前大部分的Linux发行版中已经支持Python 3。对于OpenStack来说，移植到Python3意味着后续的代码更加安全、更加高效，越晚移植就意味着越大的风险。目前社区已经开展了这部分工作。  
<http://techs.enovance.com/6521/openstack_python3>  
<http://techs.enovance.com/6722/status-of-the-openstack-port-to-python-3-2>

### Tempest
目前Neutron的Tempest测试的并行度很差，原因主要出在OVS agent与ML2 plugin的交互上，社区正在努力解决。目前Gate上的ovs-vswitchd版本是v1.4.x，并非多线程，但v2.0.0是多线程的，可以考虑作为解决方案之一。目前并行化还有很多任务要做。

目前基于H版的VPNaaS的测试已经加入了Gate，而H版中尚没有FWaaS tempest测试用例。

Neutron的Tempest用例目前仍是欠缺的。最新的情况通过下面的链接查看：  
<https://etherpad.openstack.org/p/icehouse-summit-qa-neutron>
特别的，对于新手，可以参照下面的链接学习：  
<https://wiki.openstack.org/wiki/Neutron/TempestAPITests>

有人提出ceilometer的tempes以及相关bp的review进度缓慢，由于ceilometer以及heat的tempest架构涉及到与其他模块之间的交互，tempest团队需要处理诸多blueprint，很多涉及到夸模块的tempest bp比较难以处理。

### Nova
Linux可以使用ssh免密码登录，window怎么办？开发Hyper-V driver的Cloudbase团队给出的方案，不过要依赖于他的cloudbase-init而不是cloud-init。  
<http://www.cloudbase.it/windows-without-passwords-in-openstack>

对于API的变更，如果是新的扩展，要保证在V2和V3同时支持。同时，社区建议在icehouse-2时，停止对V2 API的变更（<https://etherpad.openstack.org/p/icehouse-summit-nova-v3-api> ）。

社区有人提了“instance-level snapshots”，其实就是想提供为虚拟机所有卷做快照的能力（Nova和Cinder其实已经分别有了create-image和create-backup的能力）。John Griffith认为，从Cinder的角度来看，不提倡使用snapshot，在LVM driver场景下，snapshot对母卷的性能有较大影响（使用LVM Thin可能会缓解），他更提倡使用后端卷能力：  
BP：<https://blueprints.launchpad.net/nova/+spec/instance-level-snapshots>  
wiki：<https://wiki.openstack.org/wiki/Nova/InstanceLevelSnapshots>

目前cold migration的操作无法指定目的主机，Jay Lau实现了自定义的HA service(未开源)，提出了这个需求。Nova PTL Russell Bryant认为该建议合理

### Ceilometer
来自HP的贡献者Herndon, John Luke计划为ceilometer支持vertica数据库，由于这是个非开源软件（但有社区版），因此CI需要走第三方软件的方式。  
<http://ci.openstack.org/third_party.html>

同样是Herndon, John Luke，提出为ceilometer支持批量消费通知，由于目前的消息机制是每次只消费一个，存在一定的性能问题，如果在一定的超时时间窗内缓冲若干消息，然后再一次性入库则能提高插入性能。这个方案目前问题很多，例如错误处理机制等，尚在讨论中。

### Neutron
在OpenStack香港峰会上大家讨论了分布式路由，目前设计稿已经出炉：  
<https://docs.google.com/document/d/1iXMAyVMf42FTahExmGdYNGOBFyeA4e74sAO3pvr_RjA/edit>

高级服务虚拟机的话题一直在讨论中。  
bp：
<https://blueprints.launchpad.net/neutron/+spec/adv-services-in-vms>；  
相关文档：<https://docs.google.com/document/d/1pwFVV8UavvQkBz92bT-BweBAiIZoMJP0NPAO4-60XFY/edit>  
etherpad：  
<https://etherpad.openstack.org/p/NeutronServiceVM>

