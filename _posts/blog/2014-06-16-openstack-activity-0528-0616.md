---
layout: post
title: OpenStack社区动态第十五期(0528-0616)
description: OpenStack社区动态第十五期(0528-0616)
category: blog
---

声明：  
本系列由华为OpenStack团队出品，由孔令贤整理，欢迎转发，但请保留原作者信息!  
新浪微博：[@孔令贤HW](http://weibo.com/lingxiankong)；  
博客地址：<http://lingxiankong.github.io/> 

## 业界动态
5.27号，Oracle发布了[MySQL Fabric](http://www.oracle.com/us/corporate/press/2208808?rssid=rss_ocom_pr)，为MySQL提供了HA和可扩展性的支持。这对于OpenStack来说是无疑是件好事。 

2014.6.2，第一个(至少是我所知道的第一个)使用在生产环境中，采用IPv6的OpenStack云平台部署成功，是一个叫Nephos6的公司主导。在之前与Luxembourg大学的试点中已经成功支持了德国选举。  
<http://www.businesswire.com/news/home/20140602005581/en/IPv6-only-OpenStack-Cloud-deliver-production-services-deployed>

Cloud OS for Data Center?  
<http://www.dogeos.net/>

2014.6.3，Red Hat, Intel和Telefonica宣布在NFV领域建立战略合作伙伴关系，基于开源软件建立虚拟基础设施管理平台。该开源解决方案将会基于Intel Xeon E5-2600 V2、Red Hat Enterprise Linux、KVM、Red Hat’s OpenStack platform以及基于OpenFlow的网络设备。  
<http://www.datacenterknowledge.com/archives/2014/06/03/telefonica-red-hat-and-intel-collaborate-to-build-an-open-digital-telco>

2014年6月9号，Docker社区正式发布了Docker 1.0，注册商业公司Docker Inc.同时宣布提供企业级服务支持Docker的大规模商用；同一天，第一届Docker技术大会在San Francisco高调开幕，预定500人的会场有超过900多人报名，赞助商包括IBM、Redhat、Rackspace等IT巨头；6月10，谷歌宣布与Docker的全新整合方式，涉及的云服务包括Google App Engine和Google Compute Engine。  
<http://www.csdn.net/article/2014-06-12/2820209-Docker-1.0>

## 社区跟踪
### Common
在前几期中，已经多次提到社区正在进行的DefCore项目，这里有三个案例可以看一下：  
<http://robhirschfeld.com/2014/05/20/defcore-three-cases>

社区出现了一个新的Monitoring as a Service，目前是准备在Ceilometer实现：  
<https://blueprints.launchpad.net/ceilometer/+spec/monitoring-as-a-service>  
<https://wiki.openstack.org/wiki/MONaaS>  
<https://etherpad.openstack.org/p/MONaaS>

Mirantis的Rally，想必大家都有所耳闻，Mirantis为了推广，已经在其官方上发表了很多博客，最近的一篇：  
<http://www.mirantis.com/blog/rally-openstack-tempest-testing-made-simpler/>

2014.5.25，Designate(DNS as a Service)项目申请孵化  
<https://wiki.openstack.org/wiki/Designate/Incubation_Application>

根据社区doc team PTL Anne Gentle 的峰会总结，后续的文档会做如下几件事情：  
1、Architecture design guide  
2、优化Documentation/HowTo wiki  
3、Security Guide会独立出来，成立一个独立的core team  
4、添加Heat相关的文档内容  
5、也在计划把training guides独立出来

Mirantis的Trove教程又来了：  
<http://www.mirantis.com/blog/openstack-database-trove-native-replication-part-1/>

怎样才能成为一个理想的OpenStack程序员？Mark McLoughlin谈了他的想法。  
<http://blogs.gnome.org/markmc/2014/06/06/an-ideal-openstack-developer/>

虽然现在OpenStack的文档在慢慢成熟，但作为面向OpenStack的应用开发者来说依然很多地方苦不堪言。  
<http://engineeredweb.com/blog/2014/state-app-dev-with-openstack/>

2014.6.9，OpenStack Icehouse 2014.1.1 发布，供修复79个bug，2014.1.2预计会在8.7号发布。  
2014.1.1 release notes：<https://wiki.openstack.org/wiki/ReleaseNotes/2014.1.1>

镜像是OpenStack中的一个重要的资源，你可以选择自己定制，也可以图方便直接使用现成的。那么这些镜像可以从哪里获取？下面的文章做了一个总结。  
<http://thornelabs.net/2014/06/01/where-to-find-openstack-cloud-images.html>

2014.6.13号，juno-1版本release，实现了51个bp，修复了910+ bug。下一个版本juno-2，预计在7.24号发布。

在亚特兰大峰会上，User Experience 小组正式成立。这个小组是干什么的？简单的说就是对OpenStack相关的用户接口(命令行、界面、API等)的定义进行讨论、跟踪。  
<http://uxd-stackabledesign.rhcloud.com/moving-forward-user-experience-team-openstack-juno-release-cycle/>

OpenStack技术委员会对于大家是比较神秘的，因为很多人不知道这个组织是干啥的。2014.6.3号的TC会议上，大家决定，为了工作透明和彰显社区的开放性，后续TC会议相关的内容会收录在OpenStack blog中。6.11号是第一期，主要说明背景，以及对项目的孵化申请和集成申请的阐述。主要涉及两个项目，一是Glance，大家对该项目目标和范围有些争议；一个是最近刚申请孵化成功的Designate，估计会在L版正式发布。  
<http://www.openstack.org/blog/2014/06/openstack-technical-committee-update/> 

2014.6.12，来自HP的工程师在ML中发起一个[讨论](http://lists.openstack.org/pipermail/openstack-dev/2014-June/037413.html)，关于OpenStack安全问题。在安全团队的7月份中期审视中会详细讨论这个[话题](https://etherpad.openstack.org/p/ossg-juno-meetup)。前期已经有人做了一些[工作](https://wiki.openstack.org/wiki/Security/Threat_Analysis)。

rsyslog在OpenStack中的使用。  
<http://openstack.prov12n.com/openstack-lumberjack-part-1-rsyslog/>

关于TripleO的介绍，主要介绍Golden Images：  
<http://blog-slagle.rhcloud.com/?p=182>

### Nova
把Ironic 作为第三方nova driver测试的投票项，几位core都表示ok
https://www.mail-archive.com/openstack-dev@lists.openstack.org/msg25279.html

一个quota class的问题引起了社区对Nova API修改的讨论，之前的做法确实不严谨。Nova PTL Michael Still建议，后续对API的修改，必须提交spec，而且相应的Tempest测试用例也已准备好。  
<http://www.mail-archive.com/openstack-dev@lists.openstack.org/msg26750.html>

### Cinder
峰会上，Cinder PTL John Griffith组织了一个session，讨论SDS和Cinder的关系，John在会后也发了一篇[博客](http://griffithscorner.wordpress.com/2014/05/16/the-problem-with-sds-under-cinder/)再次阐述自己的观点，他认为SDS与Cinder在架构、目标、代码等方面有较多重合，且不利于Cinder的未来发展，因此他本人是不愿接纳类似的bp。但作为一个开源社区，他也会尊重大家的意见。有意思的是，来自EMC的一名工程师也发了一篇[博客](http://theruddyduck.typepad.com/theruddyduck/2014/05/openstack-cinder-and-software-defined-storage-sds.html)阐述SDS接入Cinder的必要性。

Cinder中关于容灾工作已经开始逐步开展起来，目前社区开始细化容灾类议题的设计细化工作，如drbd-volume议题。   
<https://etherpad.openstack.org/p/juno-cinder-DRBD>

### Neutron
在H版，一个L3 agent只能关联一个external network实现三层功能。但这个问题在Icehouse已经得到[优化](https://review.openstack.org/#/c/59359/)，如何操作？   
<http://blog.oddbit.com/2014/05/28/multiple-external-networks-wit/>

在Neutron中，虚拟机单网卡如何分配两个IP？下面的方法其实比较土的，但我也没有找到更好的办法  
<http://blog.oddbit.com/2014/05/28/booting-an-instance-with-multi/>

Neutron目前的发展，提供给各个厂商足够的灵活性和定制化，在大家争相称赞其架构的同时，来听一听不同的声音，来自[@北京-小武](http://weibo.com/u/2548340317)：  
<http://www.sdnap.com/sdnap-post/4527.html>

Diving into OpenStack Network Architecture：   
<https://blogs.oracle.com/ronen/entry/diving_into_openstack_network_architecture>  
<https://blogs.oracle.com/ronen/entry/diving_into_openstack_network_architecture1>

### Heat
两个Heat的核心开发者开发了一个协助租户自动生成模板的工具，[Flame](http://dev.cloudwatt.com/en/blog/introducing-flame-automatic-heat-template-generation.html)，基于该工具，租户可以将自己在现有环境上的资源，转换成一个Heat可以使用的模板，然后可以在其他环境上重新构建自己的程序。

目前，heat已经集成trove，可以通过heat对trove进行resize的操作  
<https://www.mail-archive.com/openstack-dev%40lists.openstack.org/msg26478.html>

### Ceilometer
Atlanta峰会结束之后，大家开始纷纷总结。当然，来自各个项目PTL的总结更加的有参考价值和指导意义，来自enovance的Ceilometer项目的PTL针对峰会期间的design session进行了总结。可以看到，Ceilometer后面还会在接口和架构上有些大的动作：  
<http://techs.enovance.com/6994/openstack-design-summit-juno-from-a-ceilometer-point-of-view>

Juno-1会在2014.6.12号发布，目前blueprint milestone实现情况：

* [切换到oslo.message](https://blueprints.launchpad.net/ceilometer/+spec/switch-to-oslo.messaging)
* [SQL后端处理大量数据时候性能的改善](https://blueprints.launchpad.net/ceilometer/+spec/big-data-sql)
* [改善Hbase后端对于查询resource列表接口的性能的改善](https://blueprints.launchpad.net/ceilometer/+spec/hbase-resource-rowkey-enhancement)
* [关于改进测试用例](https://blueprints.launchpad.net/ceilometer/+spec/grenade-upgrade-testing)
* [Hbase后端支持event](https://blueprints.launchpad.net/ceilometer/+spec/hbase-events-feature)
* [在虚拟机resource metadata中包含虚拟机的状态信息](https://blueprints.launchpad.net/ceilometer/+spec/ceilometer-instance-state-measurement)
* [支持计量LoadBalancer](https://blueprints.launchpad.net/ceilometer/+spec/ceilometer-meter-lbaas)
* [ceilometer-api扩展性的优化](https://blueprints.launchpad.net/ceilometer/+spec/declarative-filters)

最近值得关注的一个已经合入specs的bp：分离alarm的存储后端和ceilometer的存储后端：https://review.openstack.org/#/c/94857/

Ceilometer Juno版本主要任务是解决性能问题，新对象模型以及新接口将被开发，项目名称gnocchi，详见:  
<https://github.com/openstack/ceilometer-specs/blob/master/specs/juno/gnocchi.rst>  
<https://github.com/stackforge/gnocchi>

### Horizon
为Icehouse版本的Horizon配置SSL：  
<https://raymii.org/s/tutorials/Openstack-Set-Up-Horizon-Dashboard-on-Ubuntu.html> 