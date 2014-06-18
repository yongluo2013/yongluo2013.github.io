---
layout: post
title: OpenStack社区动态第四期(12.06-12.16)
description: 来自华为OpenStack社区团队出品的周报
category: blog
---

声明：  
本博客由华为OpenStack团队出品，欢迎转发，但请保留原作者信息!  
新浪微博：[@孔令贤HW](http://weibo.com/lingxiankong)；  
博客地址：<http://lingxiankong.github.io/>  
内容系本人及本人团队学习、研究和总结，如有雷同，实属荣幸！

## 业界动态
IBM本周五表示，已经开发出一款能够让客户在多个云之间迁移数据的云存储软件——InterCloud。据称，IBM正在为InterCloud申请专利，这项技术旨在向云计算中增加弹性，并提供更好的信息保护。IBM还开发了名为InterCloud Storage的软件，利用存储系统、第三方私有云和公有云用于备份、共享和数据迁移。

Mirantis发布了OpenStack Release 3.2.1，修复了3.2版本的一些bug，Release Note[点此](http://docs.mirantis.com/fuel/fuel-3.2.1/pdf/Mirantis-OpenStack-3.2.1-RelNotes.pdf)

一份IDG公司的[调查](http://www.idgconnect.com/view_abstract/16509/openstack-the-platform-choice-cloud)，有84%的美国企业考虑计划部署openstack

UnitedStack宣布加入Open Invention Network(开源发明网络组织)，成为国内第一家加入该组织的公司，支持云计算专利互不侵犯。详见Yahoo Finance对此的[报道](http://finance.yahoo.com/news/unitedstack-joins-open-invention-network-140000257.html;_ylt=ArKUrg4m.m.WugQ3kPK29erQtDMD;_ylu=X3oDMTBsOWZnNDlhBGNvbG8DYWM0BHBvcwMxBHNlYwNzcg--)  
OIN于2005年成立，由IBM, NEC, Novell, Philips, Red Hat and Sony投资并成立

UCloud成功实现SDN交换机在云计算IaaS中的运营！12月11日，UCloud将在国家会议中心举行了UCloud SDN发布会。

百度建成下一代数据中心，供应商为浪潮。日前，由浪潮集团设计制造完成的SmartRack整机柜服务器在百度内蒙古数据中心部署完成。浪潮与百度因此实现了单日近3000台服务器节点的交付，共同创造了业内服务器部署速度的新纪录，这也是此类创新形态国产服务器在国内首次大规模应用。  
<http://tech.gmw.cn/2013-12/09/content_9745410.htm>

OpenDaylight SDN项目迎来新领导 | ODL到现在为止还没有自己的执行主管，但这种情况已经发生改变。Linux基金会12.10号宣布前任VMware高管Nicolas “Neela” Jacques现在是这个OpenDaylight项目的第一任执行主管。Jasques表示在开源项目中，执行主管的作用就是促进者。 OpenDaylight的目的和目标是为最终用户解决SDN和NFV的挑战，无论这个最终用户是是网络运营商、企业还是供应商。

在对超过2300位IT硬件买家的调查中，Forrester发现55%的受访者计划在未来12个月内构建内部私有云；而Forrester在调研后称：惠普私有云服务遥遥领先，其次是思科和微软，惠普的核心私有云平台是惠普CloudSystem企业套件，该套件是用于构建私有云的基于OpenStack的管理平台。  
<http://news.cnw.com.cn/news-international/htm2013/20131126_286825.shtml>

google[发布](https://play.google.com/store/apps/details?id=com.n3infinity.stack_mobile)了一个Android app ——“Stack mobile”, 该应用程序提供任何地点、任何时间管理OpenStack的云基础设施，适配OpenStack的G版和H版  
  当前提供的功能为：  
  1. Managing multiple accounts  
  2. Instance management  
  3. Volume management  
  5. Security group management  
  6. Usage report  
  7. Limit summary  
  8. Network/subnet management  
  9. Floating IP management  
  10. Multiple projects  
可以通过这个链接看一下界面，但是该app的使用需要购买。

甲骨文宣布自身已经正式成为OpenStack基金会的“赞助厂商”，甲骨文希望借助OpenStack之力在各项存储、虚拟化、计算服务及其OpenStack版本之间实现广泛的数据兼容性。  
<http://www.openstack.cn/p687.html>

## 社区跟踪

### Ceilometer
为什么Ceilometer当前不能得到一些和instance操作相关的消息通知（boot、delete、power_off）的消息通知，而能收到另一些（update）？  
Ceilometer当前会忽略很多的消息通知，当前正在努力做到获取全部的通知，如果当前想要获取所有的通知，建议使用[StackTach](https://github.com/rackerlabs/stacktach)，一个和Ceilometer类似的通过RabbitMQ获取通知的监控系统。

### Common
Openstack一直在讨论如何做一个agent，技术从来不是问题，因为Openstack上牛人太多。比较有意思的是讨论的半天，发现Saltstack的agent，基本满足需求，好像就不需要重复造轮。（题外话，随着DevOps的发展，涌现了越来越多的运维工具，Puppet，Chef等。SaltStack是2011年才出来的新项目，貌似现在只要沾上stack的字样的项目都会流行）。  
<http://www.mail-archive.com/openstack-dev@lists.openstack.org/msg10937.html>

OpenStack 2013.2.1 candidate，于2013.12.08发布。icehouse-2的BP实现到12.09号截止，届时，处于Review, Drafting, or Discussion状态的BP都会被自动移到icehouce-3.

Steven Hardy对Heat Core价值和review提出的一些建议，如果你有意成为Heat的Core，下面几条就要注意了：  
  a.需要code们更好质量的review，聚焦于代码的逻辑错误，而不是对简单bug做+1 或者 -1  
  b.提交patch，而不是简单的给别人review代码，因为review别人代码需要对代码有更深的了解，光凭review别人的代码是做不到的  
  c.解决bug，通过测试或者其他手段发现bug，或者修复bug  
  d.积极参加各种讨论，尽可能的参加每周会议   

### Heat
Heat于11.26号提出的[自治愈bp](https://blueprints.launchpad.net/heat/+spec/stack-convergen)（bp描述：希望heat有一套机制能够自治愈或者集中管理，比如对基础资源，不管出于什么原因如果基础资源消失或者处于错误状态，希望通过用户的操作或者内部调度对资源删除、重创建......总之希望heat能够自治愈）

有人提出Heat需要提供预览模板的能力，确保模板所列资源可以成功创建，带来的两点重要的意义：  
   a） 能够为模板的作者提供校验功能  
   b） 其他使用者可以使用别人可用的模板创建资源（因为可能涉及付费）  
   已经提出BP:<https://blueprints.launchpad.net/heat/+spec/preview-stack>, 目前处于讨论状态  
   有人提出可以借鉴AWS的[相关功能](http://docs.aws.amazon.com/AWSCloudFormation/latest/APIReference/API_EstimateTemplateCost.html)

### Cinder
Ceph and Swift: [Why we are not fighting](http://techs.enovance.com/6427/ceph-and-swift-why-we-are-not-fighting) 在Ceph大热之后，很多人认为Ceph比Swift好，事实上他们之间差别较大，在不同的应用场景下各领风骚。




