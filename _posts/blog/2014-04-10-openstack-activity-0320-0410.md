---
layout: post
title: OpenStack社区动态第十一期(0320-0410)
description: OpenStack社区动态第十一期(0320-0410)
category: blog
---

声明：  
本动态跟踪系列由华为OpenStack团队出品，由孔令贤整理，欢迎转发，但请保留原作者信息!  
新浪微博：[@孔令贤HW](http://weibo.com/lingxiankong)；  
博客地址：<http://lingxiankong.github.io/>  

## 业界动态
tonido公司发布了一个网络文件夹共享和同步工具，[FileCloud 5.0](http://www.tonido.com/blog/index.php/2014/03/19/filecloud-5-0-network-folder-sync-endpoint-backup-openstack-and-more)，实现了与Swift的对接。

3.27号，微软中国宣布由世纪互联负责运营的Microsoft Azure公有云服务正式商用。这是国内首个正式商用的国际公有云服务平台。而开正式开放商用，意味着任何企业现在都可以付费使用服务，而不用再通过微软中国的甄选。

2014年2月，根据RightScale发布的云市场[调查报告](http://assets.rightscale.com/uploads/pdfs/RightScale-2014-State-of-the-Cloud-Report.pdf)，94%的企业已经在云上部署业务或处于试验阶段，74%的企业计划走混合云路线；而在私有云市场，除了VMware的50%的占用率，OpenStack占了48%

爱立信与Mirantis签署了为期5年的战略合作（5年投资3000万），双方合作，致力于OpenStack的可靠性，稳定性和性能。

4.2号，RedHat和Cisco合作，将Red Hat Enterprise Linux OpenStack Platform与Cisco的网络结合，同时，两家巨头将会在代码开发、接口定义等社区方面合作。

除了Mirantis的stackalytics统计网站，可能很多人都不知道，Bitergia也有一个统计页面  <http://activity.openstack.org/dash/releases/index.html>

## 社区跟踪
### Common
3.21号，OpenStack 2013.1.5(Grizzly)发布，6个项目共修复44个bug。依照官方的说法，尚未规划Grizzly未来版本的发布计划。  
Release notes:   
<https://wiki.openstack.org/wiki/ReleaseNotes/2013.1.5>   
各个项目情况如下：  
<https://launchpad.net/cinder/grizzly/2013.1.5>  
<https://launchpad.net/glance/grizzly/2013.1.5>  
<https://launchpad.net/horizon/grizzly/2013.1.5>  
<https://launchpad.net/keystone/grizzly/2013.1.5>  
<https://launchpad.net/nova/grizzly/2013.1.5>  
<https://launchpad.net/neutron/grizzly/2013.1.5>  

3.21号，OpenStack的消息队列服务[Marconi](https://wiki.openstack.org/marconi‎)的核心团队宣布，经过与社区委员会讨论后，取消日前的集成项目申请，继续孵化一个开发周期，原因有以下几点：  
1. 项目的目标尚不清晰；  
2. 门槛用例测试框架尚有一些严重问题需要解决；  
3. 目前项目支持的drivers有为解决的部署上的问题；  
4. 孵化周期对项目的发展是有利的；  
5. 日前在ML中针对Marconi有一些重要的问题需要日后解决；  
6. 集成的一些要求尚未满足；  

3.27，Mirantis发布了[MIRANTIS OPENSTACK EXPRESS](http://express.mirantis.com/)，企业级的Data-Center-as-a-Service

如何加快devstack的部署？[这篇](http://blog.nemebean.com/content/using-pypi-mirror-devtest)文章介绍如何使用pypi-mirror做到的。

如何通过Android访问OpenStack？有兴趣的话可以了解一下一个新的周边项目：  
<https://launchpad.net/openstackdroid>  
<http://git.openstack.org/cgit/stackforge/openstackdroid/>

OpenStack Icehouse Release Candidate于3月底4月初陆续发布：  
[Nova](https://launchpad.net/nova/icehouse/icehouse-rc1)发布，修复131个bug  
[Cinder](https://launchpad.net/cinder/icehouse/icehouse-rc1)发布，修复51个bug   
[Keystone](https://launchpad.net/Keystone/icehouse/icehouse-rc1)发布   
[Ceilometer](https://launchpad.net/Ceilometer/icehouse/icehouse-rc1)发布，修复38个bug   
[Heat](https://launchpad.net/Heat/icehouse/icehouse-rc1)发布，修复63个bug   
[Neutron](https://launchpad.net/Neutron/icehouse/icehouse-rc1)发布，修复146个bug 
版本经理Thierry Carrez说，如果木有严重bug，那么此次的RC版就是4.17号的正式发布版本。

如何使用SUSECloud安装多节点OpenStack环境？可参考如下教程：  
<http://ehaselwanter.com/en/blog/2014/03/22/susecloud-part-1-install-the-multi-node-openstack-ceph-environment>

4.3号，社区宣布，针对Grizzly版本将不再维护（最终版本是2013.1.5），而对于Havana的维护策略可以参见[这里](https://etherpad.openstack.org/p/stable-havana-ideas)

4.4号，2013.2.3 stable Havana发布，修复了106个bug，release notes:  
<https://wiki.openstack.org/wiki/ReleaseNotes/2013.2.3>

4.8号，[cinder](https://launchpad.net/cinder/icehouse/icehouse-rc2), [keystone](https://launchpad.net/keystone/icehouse/icehouse-rc2), [ceilometer](https://launchpad.net/ceilometer/icehouse/icehouse-rc2), [Neutron](https://launchpad.net/neutron/icehouse/icehouse-rc2) RC2版本发布。

J版峰会在即，disign sessions目前还在收集，4.20号结束提交。可以登录<http://summit.openstack.org/>查看目前提交的所有议题。

### Nova
为了更有效的支持Nova的bp review，Nova团队重新审视了bp的[review过程](https://wiki.openstack.org/wiki/Blueprints#Nova)，并与gerrit系统结合，新建了一个名为[openstack/nova-specs](  http://git.openstack.org/cgit/openstack/nova-specs)的新[项目](https://github.com/openstack/nova-specs)。所以，所有面向Juno的bp都要使用这种方式，即便之前已经被Approved，launchpad平台依然作为bp的跟踪平台。新的bp提交模板可以参考[这里](https://github.com/openstack/nova-specs/blob/master/specs/template.rst)

看一下老外对迁移虚拟机做的代码走读笔记  
<http://bodenr.blogspot.com/2014/03/openstack-nova-vm-migration-live-and.html>

以为来自Rackspace的工程师对OpenStack API设计的思考：  
<http://www.tildedave.com/2014/04/02/client-challenges-for-infrastructure-apis.html>  

* Which operations are supported on a resource at any given time corresponding to a specific hypervisor?  One possibility to improve this API design is to explicitly make a resource corresponding to each kind of action that is supported on a server, and provide links in the server resource. 
* How Do We Track Long-Running Requests?  
Do this by adding a /requests resource to the server resource.

如何制作OpenStack使用的Windows镜像？  
<http://www.florentflament.com/blog/windows-images-for-openstack.html>

### Horizon
OpenStack Horizon: Controlling the Cloud Using Django  
<http://www.metacloud.com/2014/03/17/openstack-horizon-controlling-cloud-using-django>  
<https://github.com/openstack/keystone/tree/milestone-proposed>

### Keystone
社区针对废弃 v2 API 的讨论：  
<http://lists.openstack.org/pipermail/openstack-dev/2014-March/031016.html>

### Tempest
Juno中，Tempest用例会放弃对XML的支持，相关的BP：  
<https://blueprints.launchpad.net/openstack/?searchtext=deprecate-xml-in-v2-api>