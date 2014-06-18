---
layout: post
title: OpenStack网络的前世今生
description: OpenStack网络的前世今生
category: blog
---

声明：  
本文转自OpenStack中国社区，原文链接：<http://www.openstack.cn/p353.html>，作者[Joshua](http://www.openstack.cn/pauthor/joshua)，转载请注明。

在OpenStack世界中，网络组件最初叫nova-network，它混迹于计算节点nova的代码库中。nova-network可以单独部署在一台机器上，为了高性能HA也可以和nova-compute一样部署在计算节点上（这也就是所谓的multi-host功能）。nova-network实现简单，bug少，但性能可不弱哦，直接采用基于Linux内核的Linux网桥少了很多层抽象应该算强大的。不足之处是支持的插件少（只支持Linux网桥），支持的网络拓扑少（只支持flat, vlan)。

为了支持更多的插件，支持更多的网络拓扑，更灵活的与nova交互，于是有了quantum工程。quantum插件不仅支持Linux网桥，也支持OpenvSwitch，一些SDN的插件以及其他商业公司的插件。在网络拓扑上，除了支持flat，vlan外，还支持gre, vxlan。但quantum不支持关键的multi-host特性。
quantum因为和一家公司的名称冲突，于是，改名为neutron。

neutron继续演进，quantum之前的实现存在这么一个问题。我们说道说道。在quantum中，实现一种类型的插件一般包括两个部分，一部分与数据库db打交道称之为plugin，一部分是调用具体的网络设备真正干活的称之为agent。像plugin就有linux bridge plugin, opevswitch plugin, hyper-v plugin等，这些plugin因为都是与db打交道，变化的字段并不多，所以代码绝大部分是重复的。这样也就出来了一个公共的plugin叫ml2 plugin(具体的代码实现就是TypeDriver)。

但这只是一个表象，ml2还有更大的作用，那就是它里面的MechanismDriver。我举个例子讲，之前没有ml2的时候，plugin只能支持一种，用了linux bridge，就不能用openvswitch了，想要都用那怎么办，那就需要MechanismDriver上场，MechanismDriver的作用是将agent的类型agent-type和vif-type关联，这样vif-type就可以直接通过扩展api灵活设置了，所以这时候你想用linux bridge你在vif-type里直接将port绑定成linux bridge就行了，同理，想用openvswitch就将vif-type将port绑定成openvswitch就行。

除了让openvswitch, linux bridge这些不同的插件共存之外，ml2还能让不同的拓扑如flat, vlan, gre, vxlan其乐融融和谐共存，直接在ml2_conf.ini这个配置文件里都配上即可。这样也就解决了一个问题，以前前端horizon中无法配置是用flat还是vlan，因为你配了也没有用，那时候后端还不支持flat和vlan同时存在了，现在皆大欢喜了。

上面的ml2解决的只是网络中L2层的问题，对于L3层的路由功能neturon又单独整出个l3-agent，对于dhcp功能又单独整出个dhcp-agent，不过它们归根结底仍属于实际真正干功能活的，对于和数据库db打交道的那部分则是通过提供扩展api来实现的。那么现在我们想加更多的网络服务该怎么办呢，如L4-L7层的FwaaS, VPNaaS, DNATaaS, DNSaaS，所以现在neutron又出来一个新的服务框架用于将所有这些除L2层以外的网络服务类似于上述ml2的思想在数据库这块一网打尽。并且这些网络服务可能是有序的，例如可能需要先过FwaaS防火墙服务再过DNATaaS服务来提供浮动IP，所以也就有了service chain的概念。目前这块代码只进去了firewall, loadbalance, metering, vpn几块，但万变不离其宗，未来neutron就是朝这个思想继续往下做。想用哪些服务可以通过neutron.conf这个配置文件中的service_provider项指定。

最后，需要一提的是，从性能角度讲，我认为目前neutron仍然不能用于大规模的生产环境，理由如下：  
1)虽然增加了更多的插件，更多的网络服务，更多的网络拓扑，但它仍然不支持multi-host功能，对性能是极大影响  
2)neutron-server不支持workers通过fork实现的利用cpu多核的多进程机制，仍然是单线程实现的，阻塞的话(python解释器是单进程的，使用greenlet保证每个线程在单进程下也是线性的），会延缓接受其他请求对性能产生影响。  
3)neutron-server是无状态的服务，理论上讲是可以在多台机器上运行前端再加haproxy实现HA的，但实际代码中存在众多竞争条件的bug。当然这个问题不仅在neturon有，其他组件我看一样存在。  
4)在遂道性能方面存在优化的可能，遂道如GRE,VXLAN等将虚拟L2层打通反而扩大了广播风暴的可能，实际上虚拟网桥或者hypervisor都是知道虚机的IP和MAC地址的映射的关系的，根本不需要仍使用传统的ARP广播方式来获得这种映射关系。  
5)其他…

成熟的路上漫漫其修远兮，这也正好给各位爱好openstack的童鞋们提供了大显身手的好机会 :)