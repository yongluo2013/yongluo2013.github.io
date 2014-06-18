---
layout: post
title: OpenStack社区动态第一期(11.17-11.22)
description: 来自华为OpenStack社区团队出品的周报
category: blog
---

声明：  
本博客由华为OpenStack团队出品，欢迎转发，但请保留原作者信息!  
新浪微博：[@孔令贤HW](http://weibo.com/lingxiankong)；  
博客地址：<http://lingxiankong.github.io/>  
内容系本人及本人团队学习、研究和总结，如有雷同，实属荣幸！

## 业界动态
Mirantis在上周末发布了Mirantis OpenStack 4.0 Technical Preview for Havana，仅仅是一个技术预览版，不能作为生产环境使用。4.0预览版：  

-    支持H版（但如果部署在RedHat上，仍然是G版）
-    将健康检查工具集成到命令行中
-    仍然没有包含Ceilometer和Heat
-    没有SLA的支持

关于Mirantis的稳定版3.2，请参见我的另一篇博客：
<http://lingxiankong.github.io/blog/2013/11/19/mirantis-openstack/>

还记得之前的那本由社区的大师们在5天时间内写出的书籍《OpenStack Operations Guide》和《OpenStack Security Guide》么，现在前者第二版的电子版出来了，[由此](http://docs.openstack.org/ops/)获取

O'Reilly即将出版关于openstack部署的书籍《Deploying Openstack》，基于Havana版本，目前的收费电子版只有前四个章节，我想该书会对openstack用作产品化，提供一整套理论与实践体系的支撑，共同期待后续。  
<http://shop.oreilly.com/product/0636920032601.do?sortby=publicationDate>

[stackinsider](http://www.stackinsider.com/index.html), 专门做Deployment as a Service(DaaS)的一家公司，可以看下他们提供安装服务的形式，供我们借鉴。

## 社区跟踪

### Common
记得上个月社区的Nova PTL Russell Bryant已经对blueprint的提交有过讨论，本周官方的wiki就有了[更新](https://wiki.openstack.org/wiki/Blueprints#Blueprint_Review_Criteria).

关于UT，到底是使用mock, mox 还是stub呢？目前大家意见不一，但大部分人选择mock，因为它基于mox，且得到python3.0的支持。而且有些项目已经开始改变（但目前没有官方的wiki或hacking doc统一说明）：  
<https://blueprints.launchpad.net/neutron/+spec/remove-mox>  
<https://blueprints.launchpad.net/nova/+spec/mox-to-mock-conversion>
 
国内openstack实践者陈沙克又发力了，详细介绍了[在centos上安装openstack（多节点）](http://www.chenshake.com/how-node-installation-centos-6-4-openstack-havana-ovsgre/)，网络采用OVS+GRE：

本次的HongKong Summit上，大家对自动化部署都比较感兴趣，UnitedStack作为一家国内的openstack startup公司，也有自己的自动部署产品，以下是UnitedStack在自动化部署方面的总结：  
<http://www.ustack.com/blog/openstack-deployment-openstack-summit-2013/#more-1946>

### Tempest
北京时间本周四，openstack的Jekins又发生了gate failure现象，导致在18个小时的时间内门槛用例队列中堆积了超过130个任务。社区已经暂停了代码的合入，待修复Gate之后，会重启Zuul，之前已经排队的任务会优先。  
<http://lists.openstack.org/pipermail/openstack-dev/2013-November/019931.html>

### Heat
作为一个新的项目，Heat目前还处于很多特性的讨论中  
对multi-region的支持：  
https://wiki.openstack.org/wiki/Heat/Blueprints/Multi\_Region\_Support\_for\_Heat  
<http://lists.openstack.org/pipermail/openstack-dev/2013-November/019138.html>
 
AutoScaling，基本思想仍然是参照与补齐AWS的AutoScaling功能：  
<https://wiki.openstack.org/wiki/Heat/AutoScaling>  
<http://lists.openstack.org/pipermail/openstack-dev/2013-November/019153.html>

### Nova
关于Nova中（nova-scheduler）使用db的问题，到底是使用内存数据库还是关系数据库？nova-scheduler的数据如何保存？社区目前仍在[讨论](http://lists.openstack.org/pipermail/openstack-dev/2013-November/019616.html)
 
在大规模环境下，对db的访问，仍然是nova-scheduler的瓶颈，mirantis团队此前对此有很详细的[研究和总结](http://www.mirantis.com/blog/nova-scheduler-database-interactions-how-to-nail-those-scalability-thwarters/)：
 
Container想自立门户？到底是hypervisor还是platform？社区目前尚无定论：  
<http://lists.openstack.org/pipermail/openstack-dev/2013-November/019637.html>
 
大家之前应该都遇到过创建VM失败，结果提示 "No valid host" 的问题。——这是由于之前Nova重调度导致的问题，它覆盖了原有的错误原因，造成定位困难。经过Nova的调度改进，该问题会逐步在[这个BP](https://blueprints.launchpad.net/nova/+spec/remove-cast-to-schedule-run-instance)完成后得到解决

### Neutron
继Nova推出对driver的要求后，本周，Neutron的PTL也发出了Neutron plugin/driver的[合入规范](https://wiki.openstack.org/wiki/Neutron_Plugins_and_Drivers)
 
Neutron在H版之后，会继续扩充高级网络服务。关于LoadBalancer的L7 Switching：  
<https://wiki.openstack.org/wiki/Neutron/LBaaS/l7>  
LoadBalancer Instance：  
<https://wiki.openstack.org/wiki/Neutron/LBaaS/LoadbalancerInstance>
 
关于在IPv6环境下的网络部署，社区刚刚成立了专门的讨论组，每周会定期组织IRC meeting：  
时间内：UTC时间的每周4的21:00  
IRC channel: #openstack-meeting-alt  
主席: sc68cal (Sean M. Collins)  

### Keystone
不会使用keystone V3？别担心，keystone的Core member： Adam Young，给我们提供了一些例子和介绍：  
<http://adam.younglogic.com/2013/09/keystone-v3-api-examples/>  
<http://adam.younglogic.com/2013/11/more-keystone-v3-api-examples/>  



