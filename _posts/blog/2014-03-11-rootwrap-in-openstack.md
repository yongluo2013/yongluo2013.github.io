---
layout: post
title: OpenStack中的sudo
description: 
category: blog
---

声明：  
本博客欢迎转发，但请保留原作者信息!  
新浪微博：[@孔令贤HW](http://weibo.com/lingxiankong)；   
博客地址：<http://lingxiankong.github.io/>  
内容系本人学习、研究和总结，如有雷同，实属荣幸！

## 什么是sudo
关于什么是sudo，网上有很多讲解的文章，我不多说了，写在这里也只是方便自己参考。当然，在说sudo前，不可避免要提一下su，但su有一些缺点：  
* 不安全，su工具在多人参与的系统管理中，并不是最好的选择，su只适用于少数人参与管理的系统，毕竟su并不能让普通用户受限的使用  
* 需要把root密码告知每个需要root权限的人，这显然是不安全的。  

sudo比su就先进在可以为不同的用户配置不同的命令运行权限，并且不需要普通用户知道root密码，所以sudo相对于权限无限制性的su来说，还是比较安全的。sudo执行命令的流程是当前用户切换到root（或其它指定切换到的用户），然后以root（或其它指定的切换到的用户）身份执行命令，执行完成后，直接退回到当前用户，而这些的前提是要通过sudo的配置文件/etc/sudoers来进行授权，该文件默认属性0411。sudo的优点：  
* 限制用户只在某台主机上运行某些命令。  
* 提供了丰富的日志，详细地记录了每个用户干了什么。它能够将日志传到中心主机或者日志服务器。  
* 用时间戳文件来执行类似的“检票”系统。当用户调用sudo并且输入密码时，用户获得了一张存活期为5分钟的票（这个值可以在编译的时候修改）。例如，在执行了sudo cat /etc/issue后，下次（有效期内）只需要输入cat /etc/issue即可。

sudo最常用的命令估计就是：`sudo -l`，列出当前用户的权限。

## sudo在OpenStack中使用
OpenStack中的组件在运行底层命令时，不可避免的要运行一些管理员权限的命令，在早期，就是通过sudo来实现。但这种方式存在一些问题：  

* 随着需要运行的命令越来越多，造成sudoers配置文件越来越臃肿，维护越来越困难  
* 毕竟对于sudoers文件的管理和操作，属于操作系统打包机制，不应该与OpenStack联系过于紧密  
* 作为权限管理的控制，sudo本身的机制并不能严格控制到命令的参数，做不到更为精细化的控制

于是，社区中一些牛逼的开发者发明了rootwrap机制来解决上述的问题。现在，作为公共的机制，rootwrap代码已经在oslo-incubator中维护。

### rootwrap的使用和配置
rootwrap的使用很简单，如果之前你需要运行`sudo ls -l`，那么现在你需要执行`sudo nova-rootwrap /etc/nova/rootwrap.conf ls -l`

该命令运行时的流程是这样的：

* nova用户（这里以Nova为例）以root权限执行`nova-rootwrap /etc/nova/rootwrap.conf ls -l`
* 找到/etc/nova/rootwrap.conf文件中定义的filters_path配置目录，加载该目录中的filter文件
* 根据filter文件的定义，判断是否能够执行ls -l

需要的配置：

* 在打包时，在sudoers文件中允许nova用户（这里以Nova为例）执行nova-rootwrap命令  

		nova ALL = (root) NOPASSWD: /usr/bin/nova-rootwrap /etc/nova/rootwrap.conf *

* nova.conf中，`rootwrap_config=/etc/nova/rootwrap.conf`，与sudoers文件中保持一致
* rootwrap.conf文件，以及filters_path目录（及包含的文件）的属主是root
* filters_path目录并非所有节点都一模一样，根据节点上部署的进程，放置相应的filter文件

## rootwrap的性能
虽然rootwrap用python语言实现了类似于sudo，但又比sudo更高级的功能，但其python的执行，以及过滤的过程，比直接执行sudo的效率低了不少，当然执行的命令的本身（比如最令人头疼的`ip netns exec`）也有诸多因素影响了整体的性能，社区已经有了针对该问题的讨论。总结一下，解决方式大概有如下几种（很多解决方法都会面临相当大的挑战）：

* 优化命令的执行(减少不必要的锁的使用，较少不必要的命令的执行，命令聚合执行)
* 还是使用`root_helper=sudo`，并且在sudoers文件中放开nova用户的权限，当然，失了便利性
* 将nova-rootwrap编译成C代码
* 将nova-rootwrap作为deamon进程执行，通过unix domain socket通信

当然，在笔者写这篇blog时，讨论还在继续，还未有一个最终的结论。但解决问题的过程，对于每个OpenStack相关的开发人员都是有价值的。

------------
参考链接：  
<https://wiki.openstack.org/wiki/Nova/Rootwrap>  
<http://fnords.wordpress.com/2011/11/23/improving-nova-privilege-escalation-model-part-1/>  
<http://fnords.wordpress.com/2011/11/25/improving-nova-privilege-escalation-model-part-2/>  
<http://fnords.wordpress.com/2011/11/30/improving-nova-privilege-escalation-model-part-3/>  
<https://etherpad.openstack.org/p/neutron-agent-exec-performance>
