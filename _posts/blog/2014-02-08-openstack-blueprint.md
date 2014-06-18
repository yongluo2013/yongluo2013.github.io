---
layout: post
title: OpenStack中的blueprint
description: 主要讲解了blueprint的概念和生命周期管理，以及各个项目插件要求，项目孵化和集成的要求等。
category: blog
---

声明：  
本博客欢迎转发，但请保留原作者信息!  
新浪微博：[@孔令贤HW](http://weibo.com/lingxiankong)；   
博客地址：<http://lingxiankong.github.io/>  
内容系本人学习、研究和总结，如有雷同，实属荣幸！

## blueprint简介和生命周期
简单来说，blueprint主要阐述了一些想法，例如新功能或组件，跟踪相关开发人员完成的进度，通常用来维护新特性实现的完整记录。从第一个想法到完整实现，有序管理版本发布。OpenStack使用launchpad作为协作开发平台，每个项目都有自己页面。与Bug的区别：A bug is a description of a problem, and a blueprint is a description of a solution。

对于一个bp来讲，标题和描述是必要的，但对于复杂的流程或功能，最好准备一个wiki文档，并将链接贴在描述区。同时，最好指定该bp的milestone（即版本计划），这样就更加便于跟踪进度。一个BP的生命周期包含以下几个过程：

### Creation
如果要实现一个特性，那么增加一个bp就很简单。到项目的bp页面，点击“Register a Blueprint”，然后填入必要的信息和相关的链接（如果有的话）

一旦PTL准备review，则需设置Milestone，即准备在哪个发布点前合入代码。bp被接纳（Approved）后，往往会被指定一个优先级。

> 注意，为bp设置Milestone后，该bp就会自动进入review等待队列，不用在ML或IRC做任何骚扰的事情。对于大型项目（比如Nova），往往会由PTL带领一个团队共同进行bp的review

Icehouse版本的发布周期如下图：  
![](/images/2014-02-08-openstack-blueprint/1.png)  
有几个点需要注意：  
1、FeatureProposalFreeze：这个点之后，对于新特性的代码提交不予review，这样保证了早期提交review的特性能够及时合入。  
2、FeatureFreeze：这个点之后，不允许合入包含新特性的代码。FF主要为QA工作预留时间窗。  
3、StringFreeze：这个点之后，不允许修改面向用户的描述信息（比如API的错误描述，配置项的已有描述）

### Inclusion in the release roadmap (PTLs)
PTL（或者bp review团队成员），为bp设置优先级。如果经讨论认为bp不合适，会取消bp的Milestone，并将bp的Definition置为Obsolete，优先级包含如下几种：

- Essential: 通俗的讲，如果该bp没有实现，将延迟版本发布
- High：重要的特性，尽可能在下一版本中包含
- Medium：作为路标规划的一部分的可选特性
- Low：其他几种优先级除外的特性
- Undefined：bp尚没有被指定优先级

优先级设置对于Nova来说有些特殊，所有Nova的bp开始时都是Low，当至少两个core reviewer决定或有兴趣review该bp的代码时，优先级才会提高。

### During development (assignee)
在bp的开发阶段，开发者要及时更新Implementation，状态包含如下：

- Unknown：尚没有被设置
- Not started：未开始
- Started：已开始
- Blocked：暂停，建议在下一个release meeting讨论
- Slow progress：可能会错过目标milestone
- Good progress：进展正常，会在目标milestone前实习
- Beta available：基本完成
- Needs code review：所有改动都已提交review
- Implemented：所有改动都已合入

### When merged (assignee)
当bp的代码被合入，开发者要及时将Implementation置为Implemented

## blueprint的review
在等待队列中的bp需要满足的约束：

- 已经指定了开发者
- 已经指定了milestone
- bp符合项目的路标或规划，这一点的确定相对主观，但会讨论达成一致
- 包含了足够的设计思路或实现方法。通常，会在ML中讨论bp的实现，那么在bp被approved前，讨论必须结束，并且在白板处标明ML链接
- 如果bp涉及文档改动，必须包含描述信息，在提交代码review时也要注明DocImpact
- bp的范围，能够容易的识别该bp何时算是实现

## Neutron plugin
在早期的代码提交中，第三方插件的提交往往是由core project contributor提出，代码review过程也仅仅考虑代码风格和UT用例测试。随着Neutron的发展，特别是对扩展机制的使用，越来越多的插件功能需要依赖专有的硬件或软件，这使代码的review变的困难，core reviewer无法确定代码的变动是否对功能甚至性能都无影响。如何Neutron Core Team对第三方插件接入提出了要求。

### Point of Contact Requirements
要求提供第三方插件的提供商或开源社区，必须指派人员：

- 参加Neutron team的每周例会
- 成为活跃的reviewer和contributor
- 在openstack-dev maillist和IRC中保持活跃
- 协助core team对第三方插件的bug进行分析及确认

### Testing Requirements
要求提供第三方插件的提供商或开源社区，必须提供外部CI与社区CI集成，OpenStack Infrastructure team已经提供了指导：<http://ci.openstack.org/third_party.html>，在Icehouse版本发布时，对于未提供外部CI的插件将被标示为不建议使用，如果在Juno开始迭代时还未提供，则将从代码库中删除。

当开发者对该插件提交代码review时，会自动触发执行外部CI，并反馈结果（+/-1），当然，为了提供用例失败时的分析，最好提供日志下载功能。

关于OpenStack CI和Vendor CI的关系可以参考下图：  
![](/images/2014-02-08-openstack-blueprint/2.png)

## Nova driver
Nova对于新增driver的要求同Neutron一样，要求提供第三方的CI接入。目前（20140207），Nova中的driver按照功能满足度以及CI接入要求，大致分为三级：  
GroupA：包含UT和社区功能测试，仅有libvirt (qemu/KVM on x86)满足  
GroupB：包含UT和第三方CI功能测试，目前[Hyper-V](http://wiki.cloudbase.it/hyperv-tempest-exclusions)和[VMware](https://wiki.openstack.org/wiki/NovaVMware/Minesweeper)满足要求  
GroupC：没有提供第三方CI的driver都属于该类。对于GroupC的driver将在I版发布时标示为不建议使用并逐步废弃。

对于Nova的第三方CI的要求：

- 任务可以不反馈+/-1，但要提供足够的信息
- 最大负载下，必须在patch提交的4个小时内反馈结果
- 测试任务应当至少包含Tempest用例集
- 日志归档至少保存近6个月
- 公开对应的Tempest配置文件（对于不支持的特性，可以通过配置文件过滤）

## Cinder driver
对于Cinder的插件接入，目前尚未看到官方的wiki，但从之前的讨论中，可以看到Cinder也已经开始考虑类似的约束：  <http://markmail.org/message/k7c63nsfsbrtizwx#query:+page:1+mid:hedymzw5elcnqgn7+state:results>

## projects related to openstack
一个OpenStack的周边项目，从出生之日起就想着有朝一日能够与OpenStack正式版本集成，得到来自全世界各地开发者的共同维护，从项目的产生到成功进入OpenStack官方项目，有很长的路要走。  
![](/images/2014-02-08-openstack-blueprint/3.png)  
项目产生之后，经过自身的发展，如果足够成熟，就能[申请孵化](https://wiki.openstack.org/wiki/Governance/Incubation_Request)，孵化是项目进入正式项目前必经的阶段，旨在缩小与正式项目的管理和开发模式上的差距，这个过程至少要经历两个开发周期。在开发周期的最后，由TC对项目进行review，由各个项目的PTL进行投票决定是否将其纳入正式项目。

### Incubation
OpenStack TC（Technical Commitee）考虑一个孵化项目，主要从它的范围和对已有项目的互补性，以及项目的技术选型方面考虑，并且需要满足一定的要求（包括但不限于）。

1、项目范围  

- 项目必须有一个清晰、明确的范围
- 项目范围能够体现OpenStack的整体进展和方向
- 项目在功能特性上不能与已有项目重复或冲突
- 应该尽可能的利用已有项目的功能

2、项目成熟度

- 项目需要有一个活跃的开发团队
- 项目的架构稳定
- 项目的API接口应保持稳定

3、项目进入孵化后的要求

- 项目必须托管在stackforge上
- 项目必须遵循OpenStack的协作接口（比如tox，pbr，global-requirements等）
- 尽可能使用oslo或oslo-incubator库
- 项目必须有一个相对稳定的代码review团队，并且review的要求和约束与OpenStack其他项目一致
- 项目使用OpenStack官方邮件列表进行讨论

4、QA   
项目必须建立基本的devstack-gate任务

5、文档和用户支持  

- 项目必须为开发者提供开发者文档
- 必须提供API文档

6、法律要求  

- 项目必须遵循Apache License v2
- 项目的依赖库不影响该项目的分发或部署
- 项目的开发者必须签署CLA
- 项目没有商标法务问题

### Graduation to integrated
在OpenStack每一个发布周期，TC都会重新审视孵化项目是否满足集成（integrated）要求，如果满足，就会纳入下一个开发周期并随下一版本发布。要求包括但不限于：

1、项目范围  
项目不能与其他OpenStack项目功能重复或冲突

2、项目成熟度

- 项目有足够多的开发者
- 项目已经完成与其他项目的集成（比如功能已经集成到了horizon）


3、项目集成后的要求

- 必须有至少4名reviewer
- 与市场团队确定合适的官方项目名称
- 遵循OpenStack的运作

4、QA

- 必须有可以运行的devstack-gate任务，使用devstack安装，执行tempest用例。
- UT用例和功能测试的覆盖度满足要求
- 必须兼容OpenStack支持的python版本
- bug跟踪

5、文档和用户支持

- 项目文档必须包含API使用文档，CLI使用文档、界面使用文档、安装部署文档以及配置描述信息等
- 项目应该是有用户支持记录（比如在ML或ask.openstack）

6、发布管理/安全

- 版本周期必须包含至少两个milestones 
- 其中一个milestone的发布应由release management team管理
- 必须提供至少2个人，协助进行项目[漏洞管理](https://wiki.openstack.org/wiki/Vulnerability_Management)

-----------
参考链接：  
<https://wiki.openstack.org/wiki/FeatureProposalFreeze>  
<https://wiki.openstack.org/wiki/FeatureFreeze>  
<https://wiki.openstack.org/wiki/StringFreeze>  
<https://wiki.openstack.org/wiki/Neutron_Plugins_and_Drivers>  
<https://wiki.openstack.org/wiki/HypervisorSupportMatrix>  
<https://wiki.openstack.org/wiki/HypervisorSupportMatrix/DeprecationPlan>  
<https://github.com/openstack/governance/blob/master/reference/incubation-integration-requirements>  
<https://wiki.openstack.org/wiki/Governance/NewProjects>  