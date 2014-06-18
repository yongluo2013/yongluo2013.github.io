---
layout: post
title: OpenStack社区动态第七期(01.17-02.02)
description: OpenStack社区动态第七期(01.17-02.02)
category: blog
---

声明：  
本周报由孔令贤收集整理。欢迎转发，但请保留原作者信息!  
新浪微博：[@孔令贤HW](http://weibo.com/lingxiankong)；  
博客地址：<http://lingxiankong.github.io/>  

## 业界动态
来自华为的[compass](http://www.syscompass.org/)自动化部署工具宣布代码可用（code base is available）

2014-01-17号，OpenStack个人独立董事评选结果[出炉](http://www.openstack.org/blog/2014/01/election-results-for-individual-and-gold-directors/)，来自趣游的杜玉杰成为14年中国区唯一的个人董事。

云计算解决方案服务商VMware拟将以15.4亿美金收购移动设备管理公司 AirWatch。AirWatch 主要为企业提供移动设备及设备内容方面的管理，这包括安全与监测等方面。目的是确保任何员工带入办公室的移动设备都是安全的、运行了正确的应用、可以获得正确的资源，同时当员工离职或者设备丢失时，可以迅速切断。

OpenStack Summit in Altlanta将于5月12-16号，在Georgia World Congress Center举行，topic的提交将在2.14号截止

## 社区跟踪

### Common
The second milestone of the Icehouse development cycle, "icehouse-2" is
now available for Keystone, Glance, Nova, Horizon, Neutron, Cinder,
Ceilometer, Heat, and Trove. The next development milestone, icehouse-3, is scheduled for March 6th.

中心化配置管理，通过oslo来集中管理nova，cinder，neutron的配置，并可通过API查询某个host上的某个配置项，[BP](https://blueprints.launchpad.net/oslo/+spec/oslo-config-centralized)当前还没有被接受。很多人的意见是配置管理应该使用Chef和Puppet这类工具实现，OpenStack需要做的是如何优化以更适应Chef和Puppet支持OpenStack场景的用例。

目前icehouse-3有153个bp，2.4号是bp被接纳的截止日期，如果届时没有被接纳，会被自动延迟到Juno。bp的代码提交截止日期是2.18号

### Nova
Hyper-V提供了Nova CI的接入  
<http://lists.openstack.org/pipermail/openstack-dev/2014-January/025428.html>

Nova将在V3版本的API中废弃对XML的支持。因为XML的支持在开发、维护、文档和验证都会耗费较大的精力。如果未来真有对XML支持的需求，社区会考虑以更简单的方式实现。但目前，废弃工作已近刚开始。  
<https://blueprints.launchpad.net/nova/+spec/remove-v3-xml-api>

一位Intel的同事提出一个bp，在nova-scheduler中提供基于Power/temperature的调度，貌似跟Ceilometer中的[kwapi](https://launchpad.net/kwapi)重合，尚无结论。  
<http://lists.openstack.org/pipermail/openstack-dev/2014-January/025184.html>

Nova早期的实现中cold-migration和resize是同一套代码，这也导致了两个接口行为上的一致性。具体来说就是对于cold-migration，在接口调用后，仍然需要再进行确认，这个流程对于操作员来说是不合适的。Jay Lau针对这个问题提了一个[bp](https://blueprints.launchpad.net/nova/+spec/auto-confirm-cold-migration)做修改，社区能够接受的做法是v2版本不做修改，v3做修改，而且为了baozbackward-compatible，在cold-migration时需要识别版本号。

### Tempest
negative test framework: ready for review  
<https://review.openstack.org/#/c/64733/>

tempest将废弃nosetests，很多新的特性nosetests并不满足（很多场景已经开始使用testscenarios），计划在Icehouse-3版本移除。当使用nose时，Tempest会抛出一个“不支持”的异常。

北京时间1.17号凌晨，stable/havana的gate由于netaddr版本的问题被阻塞

### Neutron
在使用Neutron + OVS + GRE/VXLAN模式时，每个neutron-agent节点都需要一个`local_ip`的配置，一般这个配置是在配置工具（Chef/Puppet等）指定。NOTSU Arata提出，是不是能不依赖安装部署工具，而采取更自动化的方式。  
<http://lists.openstack.org/pipermail/openstack-dev/2014-January/024516.html>

Partially Shared Networks:  
neutron当前network资源有一个shared字段可以表示共享，但是一旦共享就是被所有租户共享。
有人提出一个场景是需要多个租户共享一个网络，但这个网络并不是被所有租户共享。
已经有两个bp关于实现这一场景：  
https://blueprints.launchpad.net/neutron/+spec/sharing-model-for-external-networks  
https://blueprints.launchpad.net/neutron/+spec/vlan-aware-vms  
http://lists.openstack.org/pipermail/openstack-dev/2014-January/024005.html

### Keystone
Keystone当前支持password、token、external、oauth认证方式，有一个[BP](https://blueprints.launchpad.net/keystone/+spec/access-key-authentication)提出支持Keystone应该支持access-key和secret-key类型的认证，并将这种认证方式和现有的Credential后端整合。支持了access-key和sercet-key的认证方式，也可以统一nova-client和Ec2-client对于OpenStack的使用方式。

另一个相关的Keystone的BP也很强大，密码轮转，  <https://blueprints.launchpad.net/keystone/+spec/password-rotation>  
这个BP已经被规划在Icehouse-3，实现以下3个功能：  
1、密码过期失效  
2、同时支持两个密码的接入，以支持平滑的密码更换和升级场景  
3、新密码不能和以前的N次密码相同

keystone的token hash方式从MD5改为SHA256，原因是SHA256比MD5更加的安全，不容易暴力的生成同样的Hash之后的字符串，有人提出了一个很好的方式，效仿glibc的加密方式，通过前缀区分hash算法，例如：${id}${hash}，这样token使用的hash算法就包含在token本身，而不用keystone通知Client正在使用的算法是什么。

### Ceilometer
Mirantis的Nadya Privalova抱怨Ceilometer在数据较多的场景下的响应效率问题，提出要在DB中另起一表，暂存某种特殊查询的数据，或者周期任务的数据。Jay Pipes反驳说Ceilometer的重点是收集数据，预处理以及持久化。加速工作可以交由下游组件完成，如Pentaho。  
<http://lists.openstack.org/pipermail/openstack-dev/2014-January/023976.html>

samsung贡献者Deok-June Yi根据测试数据提出ceilometer告警延迟具有不可预知性，而Synaps的告警延迟稳定在2秒钟以内。原因在于ceilometer有大量数据库IO操作，而Synaps却是从内存以及流中读取数据。Deok-June Yi提议重新开始曾被搁置的Synaps并入ceilometer的特性，实现实时告警的需求。  
<http://lists.openstack.org/pipermail/openstack-dev/2012-October/001553.html>  
<http://lists.openstack.org/pipermail/openstack-dev/2012-December/003731.html>