---
layout: post
title: OpenStack社区动态第二期(11.22-11.28)
description: 来自华为OpenStack社区团队出品的周报
category: blog
---

声明：  
本博客由华为OpenStack团队出品，欢迎转发，但请保留原作者信息!  
新浪微博：[@孔令贤HW](http://weibo.com/lingxiankong)；  
博客地址：<http://lingxiankong.github.io/>  
内容系本人及本人团队学习、研究和总结，如有雷同，实属荣幸！

## 业界动态
VMware与Mirantis之前已经达成合作关系，由Mirantis帮助VMware进行openstack发行版的制作。自动化部署工具依然使用Mirantis的Fuel，底层使用VMware的虚拟化技术，其中：  
Nova使用VMware vCenter Server driver ；  
Neutron使用VMware NSX driver (之前是Nicira NVP driver)；  
Cinder使用VMware VMDK datastore driver；  
同时支持H版特性

因有用户需求，Rackspace Private Cloud 将OpenStack版本切换到了Havana，目前只是Early Access版本，仍需要做足充分测试之后才能全面切换。已经同时支持Heat和Ceilometer。[下载链接](http://www.rackspace.com/cloud/private/openstack_software/)

## 社区跟踪

### Common
tenant or project? 过去我们都习惯使用“租户”的概念，但keystone V3 API中引入了project和domain等概念，更加接近真实用户场景，社区中大部分开发者也都倾向于使用project。  
<http://lists.openstack.org/pipermail/openstack-dev/2013-November/020222.html>

用来判明权限的 policy.json，我想大家都不陌生。但由于之前的设计缺陷，导致权限判断不仅在API处通过读取policy执行一次，还会在DB层根据代码重新判断一次。两次判断不仅职责不明，还会导致不一致的问题。就这个问题团队成员 吴江 之前提过[Bug](https://bugs.launchpad.net/bugs/1240831)，今日收到社区邮件通知，整个policy部分会基于v3 API 重新设计，计划于icehouse-3 完成。目前刚刚完成[BP立项](https://blueprints.launchpad.net/nova/+spec/v3-api-policy)，设计工作尚未启动。

OpenStack董事会 个人董事选举提名 已经展开，时间为11月25日--12月9日。之后社区会根据名单，进行个人董事投票，时间为明年1月13--1月17日。任期一年。  
如果你之前加入了OpenStack基金会，你会收到一封参与选举的邮件，里面会给出参与链接(独立的) 。大家可以检查下自己登记的邮箱

### Cinder
H版中vmware新增了一个cinder driver(vmdk driver for vSphere)，那么它是如何实现的？与G版中iSCSI有何区别？参见：  
<http://cloudarchitectmusings.com/2013/11/19/laying-cinder-block-volumes-in-openstack-part-2-integration-with-vsphere/>  

Ceph在OpenStack的存储中扮演了重要的角色，那么在H版中Ceph实现了什么特性？在未来版本中有什么规划？一篇[博客](http://techs.enovance.com/6424/back-from-the-summit-cephopenstack-integration)告诉你所有

### Nova
上一周报提到目前社区正在重新制定driver接入策略，对已有的driver也提出了一些要求。目前vmware进展最快，已经是Class B，hyperv和libvirt+xen正在寻找解决方案，还没有看到docker和libvirt+lxc的方案，而IBM原来的powervm的driver已经被删除，因为IBM后续会支持[powervc](https://review.openstack.org/#/c/57774/)。可以参考如下[链接](http://ci.openstack.org/third_party.html)

FreeBSD想将自己的虚拟化平台接入openstack作为一个新的virt driver, [FreeBSD hypervisor (bhyve) driver](https://blueprints.launchpad.net/nova/+spec/freebsd-compute-node)，但社区认为应该先作为libvirt的下游接入openstack，而并非作为一个新的独立的driver接入，因为这样有益于openstack和libvirt的共同发展，同时也减少了社区对driver的维护量。

### Neutron
社区考虑将在I版release时不建议使用nova-network，但是否强制这样做，需要考虑两个因素：  
1、Neutron是否在特性上包含nova-network；  
2、在质量保障上，Neutron是否达到要求；  
当然，目前尚未有正式的结论。

峰会上关于Neutron未来的规划是什么？目前来看，SDN最火热。参见：  
<http://techs.enovance.com/6413/summit-openstack-neutron-point-of-view>

关于IPv6的支持，社区已经有了[规划](https://blueprints.launchpad.net/neutron/+spec/ipv6-feature-parity)。目前大部分开发者来自comcast公司。

### Tempest
之前我们增加的tempest用例如果涉及到增加配置项，我们需要同时在patch中修改tempest.conf.sample文件。现在不用这么麻烦了，对于新增的配置项，可以自动生成tempest.conf.sample文件，方法就是在tox中新增一个任务：  
<https://review.openstack.org/#/c/53870/>

### Keystone
keystone 的token 存在安全隐患，废弃的可能会被非法利用，Token revocation 会被优化，来自Keystone Core：  
<http://adam.younglogic.com/2013/11/token-revocations-in-keystone/#more-2903>








