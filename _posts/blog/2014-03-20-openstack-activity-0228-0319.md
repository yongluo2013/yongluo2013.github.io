---
layout: post
title: OpenStack社区动态第十期(02.28-03.19)
description: OpenStack社区动态第十期(02.28-03.19)
category: blog
---

声明：  
本动态跟踪系列由华为OpenStack团队出品，由孔令贤整理，欢迎转发，但请保留原作者信息!  
新浪微博：[@孔令贤HW](http://weibo.com/lingxiankong)；  
博客地址：<http://lingxiankong.github.io/>  

## 业界动态
[GoDaddy][1]，著名的网络域名注册商，以前是CloudStack支持者，现在是OpenStack基金会的最新赞助商，GoDaddy在全球中小企业当中有着很大的影响力，相信它的加入，OpenStack生态系统必将进一步壮大。

Cloud Foundry基金会上周从Pivotal剥离，向现有的方案下了挑战书，旨在创建开源平台即服务（PaaS）产品，包括针对OpenStack和红帽的OpenShift平台开发的产品。Cloud Foundry在Apache 2.0下已经拥有了开源PaaS许可，但是现在其以前的持有方Pivotal允许其他厂商协作，参与其Cloud Foundry的治理。基金会成员包括EMC、惠普、IBM、Rackspace、SAP和VMware。目前比较火的PAAS有：Cloud Foundry, Solum, OpenShift

Piston发布[Piston OpenStack 3.0](http://www.pistoncloud.com/press-releases/piston-openstack-3-0-the-last-openstack-product-youll-ever-try/)，在存储、网络和API支持方面有增强。Pistone的OpenStack发行版预置了部分部署配置，且属于高度集成版本，因此，在灵活性方面可能会有不足。此外，其块存储和对象存储后端都使用了Ceph统一管理，网络方面使用SDN方案（uniper Contrail™, PLUMgrid™, and VMware NSX™）。整个系统的管理是在被称作是Piston OpenStack's Moxie Runtime Environment下实现，Piston实现了自己的管理UI，远程管理采用了反向SSH隧道技术。

3.11号，Mirantis发布了OpenStack 4.1，主要的特性如下：  
* 支持OpenStack Havana 2013.2.2 Release  
* 修复一些bug  
* 操作界面支持网卡绑定  
* 升级Murano和Savana到最新版本  
* 升级Fuel工具

IBM 发布了全新的 PaaS 服务平台，代号为 BlueMix，该平台目前还处于 Beta 阶段，开始接受试用。BlueMix 目前提供对 Java、Ruby 和 Node.js 应用的支持。可通过 https://ace.ng.bluemix.net/ 来访问该平台。

Windows Azure开放了SDK，<http://windowsazure.github.io/> 

3.11号，Rackspace发布[Gophercloud 0.1](http://gophercloud.io/)，是为go语言开发的OpenStack SDK。

## 社区跟踪
### Common
OpenStack官方没有提供Java SDK，但坊间已经有不少了，为大家搜集了下.  
<https://github.com/woorea/openstack-java-sdk>  
<http://mvnrepository.com/artifact/com.woorea/openstack-java-sdk/3.2.1>  
<http://jclouds.apache.org/documentation/quickstart/openstack/>

OpenStackClient release 0.3.1  
<https://pypi.python.org/pypi/python-openstackclient>  
<http://tarballs.openstack.org/python-openstackclient/python-openstackclient-0.3.1.tar.gz>

3.8,  Icehouse-3 development milestone available for Keystone, Glance, Nova, Horizon,
Neutron, Cinder, Ceilometer, Heat, and Trove.  在过去6个月，共实现了 203 blueprints，修复了780 bugs fixed  
<https://launchpad.net/keystone/icehouse/icehouse-3>  
<https://launchpad.net/glance/icehouse/icehouse-3>  
<https://launchpad.net/nova/icehouse/icehouse-3>  
<https://launchpad.net/horizon/icehouse/icehouse-3>  
<https://launchpad.net/neutron/icehouse/icehouse-3>  
<https://launchpad.net/cinder/icehouse/icehouse-3>  
<https://launchpad.net/ceilometer/icehouse/icehouse-3>  
<https://launchpad.net/heat/icehouse/icehouse-3>  
<https://launchpad.net/trove/icehouse/icehouse-3>  

3.8号，python-novaclient 2.17.0 released，支持新的API：[os-server-external-events](https://blueprints.launchpad.net/nova/+spec/admin-event-callback-api)

扫盲贴，OpenStack的版本周期中为什么会有Feature Freeze？  
<http://fnords.wordpress.com/2014/03/06/why-we-do-feature-freeze/>

### Nova
可以使用LXC在一个服务器上起多个nova-compute进程，模拟大规模环境进行一些场景的测试，简单步骤如下：  
1. Creating a LXC with logical volume  
2. Installing a fake nova-compute inside the LXC  
3. Make a booting script that modifies its nova.conf to use its IP address & starts nova-compute  
4. Using the LXC above as the master, clone as many compute as you like!  
(Note that while cloning the LXC, the nova.conf is copied with the former's IP address, that's why we need the booting script.)

GPU在OpenStack中的应用示例：  
<http://blog.xlcloud.me/post/2014/02/27/Running-an-OpenGL-application-on-a-GPU-accelerated-Nova-Instance-%28Part-2%29>

虚拟机如何在OpenStack和VMware环境下迁移？这个命题看起来可能比较高级，但实际实现还是比较土的（基于镜像），姑且做个参考吧。  
<http://blog.activeeon.com/2014/02/openstack-vmware-vm-disk-migration.html>

在Windows虚拟机下如何使用Puppet进行软件管理？Nova的user-data是一种选择，使用Heat是另外一种选择（但在Windows虚拟机下都要用到[Cloudbase-Init](http://www.cloudbase.it/cloud-init-for-windows-instances/)）。来自cloudbase：  
<http://www.cloudbase.it/openstack-windows-and-puppet>

### Neutron
如果在neutron中使用CIDR/32(即只有一个IP)的subnet会怎么样呢？这个subnet中会没有用户可使用的IP address，因为这个IP会被用作这个subnet的gateway。因此有人认为这种subnet没有使用的必要，neutron代码中应该限制掩码过大的subnet的创建，这样省得subnet创出来了也没法使用，可以增加用户体验。而这种subnet真的没有使用的意义了吗？其实作为external network的subnet还是可以用的，用作建立仅有路由功能的三层网络。硬性的在代码中限制subnet的掩码，缺失了这种创造性的使用方式，因此可以在UI中增加友好的提示表明掩码过大会导致没有IP可用，但是不应该在neutron中直接限制死。从这个案例，我们可以看出在系统中少一些强硬性的限制，那么系统可能会以一种创造性的反馈给你惊喜。

在lbaas中即将增加ssl的实现，将使用已有的一种方式实现：  
<http://docs.openstack.org/security-guide/content/ch038_transport-security.html>  
当前由于从keystone取KDS(Key Distribution Server)受阻，keystone会在I版实现KDS，因此neutron会在J版实现  
<https://wiki.openstack.org/wiki/MessageSecurity>

关于IPv6实现场景的总结：  
<https://www.dropbox.com/s/9bojvv9vywsz8sd/IPv6%20Two%20Modes%20v3.0.pdf>  
1) If an IPv6 subnet does NOT have gateway port on neutron router (i.e. either private or provider network), then only the first two highlighted combinations are considered as valid. Because the rest five options requires RA announcement.  
2) If an IPv6 subnet does have gateway port on neutron router (i.e public network), then only the last five highlighted combinations are considered as valid. Because the first two options turn off RA announcement, which makes existing gateway port on neutron router useless.

关于IPv6场景下，需要一个dhcp方式的参数。当前需要以下四种可能的值：  
1) off (i.e. address is assigned by external devices out of OpenStack control)  
2) slaac (i.e. address is calculated based on RA sent by OpenStack dnsmasq)  
3) dhcpv6-stateful (i.e. address is obtained from OpenStack dnsmasq acting as DHCPv6 stateful server)  
4) dhcpv6-stateless (i.e. address is calculated based on RA sent from either OpenStack dnsmasq, or external router, and optional information is retrieved from OpenStack dnsmasq acting as DHCPv6 stateless server)  
最终决定使用A pair of mode keywords ：ipv6-ra-mode & ipv6-address-mode。  
(<https://www.dropbox.com/s/9bojvv9vywsz8sd/IPv6%20Two%20Modes%20v3.0.pdf>，主要还区分了使用openstack自身的DHCPv6 server和外部server的场景)

neutron中很多资源都有admin-state-up这个属性，当为false时，其实很多资源没有down掉，之前很多人对这个属性的理解有偏差，所以没有关注这个bug，现在有人在修改它了：  
<https://bugs.launchpad.net/neutron/+bug/1237807>

来自Cisco的LBaaS文档：<http://docwiki.cisco.com/wiki/OpenStack:Havana:LBaaS>

### Cinder
Ceph后端下如何配置使用cinder-multi-backend？  
<http://openstack-in-production.blogspot.com/2014/03/enable-cinder-multi-backend-with.html>

### Heat
python-heatclient 0.2.8 released，相关链接如下：  
<http://tarballs.openstack.org/python-heatclient>  
<https://pypi.python.org/pypi/python-heatclient>  
<https://launchpad.net/python-heatclient/+milestone/v0.2.5>


  [1]: http://www.godaddy.com/