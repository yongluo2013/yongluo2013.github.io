---
layout:     post
title:      OpenStack中的安全组织
category: blog
description: 安全对于一个OpenStack的发行版来说至关重要
tags:
- openstack
- security
---

声明：  
本博客欢迎转发，但请保留原作者信息!  
博客地址：<http://lingxiankong.github.io/>  
内容系本人及本人团队学习、研究和总结，如有雷同，实属荣幸！  
Author：华为云计算工程师 [孔令贤](http://weibo.com/lingxiankong)  

## OpenStack Vulnerability Management Team
OpenStack Vulnerability Management Team([VMT](https://launchpad.net/~openstack-vuln-mgmt))是负责处理OpenStack安全问题的组织，负责处理漏洞的修复和发布过程。为了防止漏洞的扩散，该组织最多只能有三个成员。该组织目前提供OpenStack前两个版本的漏洞补丁，下一版本正在开发中。

一个security bug的处理流程如下：  
![](/images/2014-01-01-openstack-security/1.png)  

- 漏洞的上报可以直接给VMT成员发送私人加密邮件，或者提交Launchpad security bug，前期主要是对漏洞及其影响的确认，一旦确认，就会指定OSSA，将ossa bugtask status置为Confirmed
- 补丁开发和检视。开发者可以是漏洞的提交者，或PTL，或PTL认为合适的人选。由core developer对补丁进行预检视，以便在漏洞公开时被快速合入。
- 漏洞描述的撰写和检视。在开发补丁的同时，VMT coordinator将按照特定的模板起草漏洞说明，并发送提交者和PTL检视
- 分配CVE号。在补丁即将被合入之前，ossa bugtask status是In progress，会以邮件的方式向CNA组织申请CVE；如果是已公开漏洞，则会向oss-security邮件列表申请CVE
- 有了补丁和CVE，下游的利益干系人会提前收到漏洞通知，漏洞的公开时间一般是3-5个工作日之后，此时，ossa bugtask status会被置为Fix committed. 在公开时间点，分支的维护者会将补丁提交到Gerrit，公开bug信息，快速合入代码。
- 代码合入后，会向OpenStack ML发送安全公告，[这里](http://lists.openstack.org/pipermail/openstack-announce/2013-December/000179.html)是最近的一个例子，最终，将ossa bugtask status置为Fix released.

## OpenStack Security Group
OpenStack Security Group（[OSSG](https://launchpad.net/~openstack-ossg)）做的事情稍有不同，该组织是通过对代码、架构、文档的优化和改进来提高用户使用OpenStack的安全性，如有必要，OSSG会向VMT提交并协助处理漏洞。该组织出品了[OpenStack Security Guide](http://docs.openstack.org/sec/)，发行了[OpenStack Security Notes](https://wiki.openstack.org/wiki/Security_Notes)，社区邮件列表[点击这里](http://lists.openstack.org/pipermail/openstack-security/)，如需订阅，请[点击这里](http://lists.openstack.org/cgi-bin/mailman/listinfo/openstack-security)。该组织目前尚不成熟。

------
参考文档：  
<https://wiki.openstack.org/wiki/Vulnerability_Management>

