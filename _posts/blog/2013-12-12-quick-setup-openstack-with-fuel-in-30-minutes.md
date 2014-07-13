---
layout: post
title: Fuel 30 分钟快速安装OpenStack
category: blog
description: 一直以来，对于openstack 的初学者来讲，安装往往是入门的头大难题。
---

声明    

本博客欢迎转发，但请保留原作者信息! 内容系本人及本人团队学习、研究和总结，如有雷同，实属荣幸!    
作者：>  [罗勇] 云计算工程师、敏捷开发实践者    
博客： [http://yongluo2013.github.io/](http://yongluo2013.github.io/)    
微博： [http://weibo.com/u/1704250760/](http://weibo.com/u/1704250760/)    


一直以来，对于openstack 的初学者来讲，安装往往是入门的头大难题。在E版本之前，要搭建一个基本能用的openstack 环境那是相当麻烦，自己要装机，自己搞源，自己照着文档敲命令，又没有靠谱的文档，官方给出的文档依旧有好多坑，还有语言问题往往用上好几天时间都装不起来，慢慢地就丧失了学习openstack 的信心！不过后来情况有了很大改观，从E版本开始，以后安装过程简化许多，文档质量提高不少。尽管如此对于初学者还讲还是比较复杂，其实很多时候，很多人只是想体会一下openstack，完全不关注安装这门子事情。还好openstack社区足够活跃，很快就有公司做出了比较友好的安装工具，比如今天要向大家介绍的Fuel这个工具，其实这里还可以叫她mirantis openstack，由Mirantis 公司开发。

![fuel overview](/images/2013-12-12-quick-setup-openstack-with-fuel-in-30-minutes/fuel_overview.jpg)

##关于 Mirantis

Mirantis，一家很牛逼的openstack服务集成商，他是社区贡献排名前5名中唯一一个靠软件和服务吃饭的公司（其他分别是Red Hat, HP, IBM, Rackspace）。相对于其他几个社区发行版，Fuel的版本节奏很快，平均每两个月就能提供一个相对稳定的社区版。

##Fuel 是什么？

Fuel 是一个为openstack 端到端”一键部署“设计的工具，其功能含盖自动的PXE方式的操作系统安装，DHCP服务，Orchestration服务 和puppet 配置管理相关服务等，此外还有openstack 关键业务健康检查和log 实时查看等非常好用的服务。

Fuel 3.2基于Grizzly版本，而最新将发布的4.0版本是基于Havana版本的技术预览版，不可用作生产环境使用，同时，4.0版本仍然不包含Heat和Ceilometer组件。

##Fuel 的优势

总结一下，Fuel 有以下几个优点：

* 节点的自动发现和预校验
* 配置简单、快速
*　支持多种操作系统和发行版，支持HA部署
×　对外提供API对环境进行管理和配置，例如动态添加计算/存储节点
×　自带健康检查工具
×　支持Neutron，例如GRE和namespace都做进来了，子网能配置具体使用哪个物理网卡等

##Fuel 的架构是怎样的呢？

![fuel master fuel arch](/images/2013-12-12-quick-setup-openstack-with-fuel-in-30-minutes/fuel_master_fuel_arch.png)

Fuel 主节点：用于提供PXE方式操作系统安装服务由开源软件Cobbler 提供，另外由Mcollective和puppet 分别提供orchestration服务和配置管理服务。Fuel iso 包发部的时候已经一同打包了Centos6.4 和ubuntu 12.04 安装包，如果需要使用红帽子企业版RHEL6.4 需要自己手动上传。

目前可以支持openstack SA 或者HA 的安装。现在我们已经对Fuel 有了大致了解，现在来看看用她来安装openstack有多么的方便！

##Fuel openstack 安装

首先要说明的是Fuel 针对目标就是生产环境openstack部署，这里为了讲解安装过程就在虚拟机上演示说明。我的环境是HP笔记本Folio 9470 ，其实是办公用的普通笔记本，读者可以根据实际机器情况自行修改虚拟机配置，我给出了我的配置仅供参考。

![fuel master bios](/images/2013-12-12-quick-setup-openstack-with-fuel-in-30-minutes/fuel_master_bios.jpg)

##安装说明

**硬件要求**

* 启用虚拟化技术支持：开启BIOS设置里的虚拟化技术支持相关选项，这个会很大程度上影响你的虚拟机性能。
* 最低硬件配置：cpu：双核2.6GHZ+；内存：4g+；磁盘：80G+
* 虚拟化工具：Oracle Virtualbox 4.2.18


**安装包准备**

* 下载virtualbox 包 https://www.virtualbox.org/wiki/Downloads/

* 下载fuel ios包，先要注册一个mirantis 用户账户，目前最新版本是3.2.1 这个版本， MirantisOpenStack-3.2.1.iso （1.8G)http://www.openstack.cn/p383.html

**安装步骤**

* 虚拟环境设置
* 安装Fuel 主节点
* 部署openstack节点
* 部署结果检查
* 虚拟环境设置

**网络拓扑**

首先在virtualbox 里面自定义如下3个网络:

```
›Net1:
–Network name: VirtualBox  host-only Ethernet Adapter#2
–Purpose: Fuel administrator network
–IP block: 10.20.0.0/24
–Linux device: eth0

›Net2:
–Network name: VirtualBox  host-only Ethernet Adapter#3
–Purpose: public/ floating network
–IP block: 172.16.0.0/24
–Linux device: eth1

›Net3
–Network name: VirtualBox  host-only Ethernet Adapter#4
–Purpose: Storage/ management/ internal network
–IP block: 192.168.4.0/24
–Linux device: eth2
```

**虚拟机创建**

```
›VM1
–Name: Fuel_3.2.1
–vCPU:1
–Memory :1G
–Disk:30G
–Networks: net1

›VM2
–Name : Fuel_3.2.1_controller
–vCPU:1
–Memory :1G
–Disk:30G
–Network:net1,net2,net3

›VM3
–Name: Fuel_3.2.1_compute1
–vCPU:2
–Memory :2G
–Disk:30G
–Networks:net1,net2,net3
```

**网络拓扑**

![network top](/images/2013-12-12-quick-setup-openstack-with-fuel-in-30-minutes/network_top.png)

创建网络Net1，注意不要启用dhcp，这个会干扰fuel 自己的dhcp服务。

![net1 setup](/images/2013-12-12-quick-setup-openstack-with-fuel-in-30-minutes/net1.png)

创建网络net2

![net2 setup](/images/2013-12-12-quick-setup-openstack-with-fuel-in-30-minutes/net2.png)

创建网络net3

![net3 setup](/images/2013-12-12-quick-setup-openstack-with-fuel-in-30-minutes/net3.png)

###安装fuel 主节点

创建fuel 主节点虚拟机,虚拟机名字为“fuel_3.2.1“。注意网卡选用net1，也就是virtualbox 的”VirtualBox  host-only Ethernet Adapter#2“ 网络。

![fuel master setup](/images/2013-12-12-quick-setup-openstack-with-fuel-in-30-minutes/fuel_master_setup.jpg)

设置完成后启动虚拟机，显示boot menu时候，如果需要修改ip地址可以自行修改，默认是不需要修改。


![fuel master boot menu](/images/2013-12-12-quick-setup-openstack-with-fuel-in-30-minutes/fuel_master_boot_menu.jpg)

开始安装操作系统

出现该画面时按任意键进入修改fuel 主节点相关配置，可以不修改使用默认值，几秒后进行软件包安装。

![fuel master boot menu](/images/2013-12-12-quick-setup-openstack-with-fuel-in-30-minutes/fuel_master_boot_menu.jpg)

puppet 安装fuel 相关软件，比如Cobbler 等。

![fuel master installation puppet](/images/2013-12-12-quick-setup-openstack-with-fuel-in-30-minutes/fuel_master_installation_puppet.jpg)

fuel 主节点安装完成。

![fuel master done](/images/2013-12-12-quick-setup-openstack-with-fuel-in-30-minutes/fuel_master_done.jpg)

看fuel 安装是否完成，就登录http://10.20.0.2:8000/ 显示如下页面。

![fuel master web home](/images/2013-12-12-quick-setup-openstack-with-fuel-in-30-minutes/fuel_master_web_home.jpg)

**可能的问题**

* 如果web 页面不能正常访问，可能是你本机的防火墙把本地的网络拒掉，请先禁用防火墙再试。
* 如果使用了浏览器http代理，请关闭代理直接访问。

接下来就开始安装openstack 环境了。

###安装openstack 环境

首先在Fuel web 上创建一个openstack 环境，名字为”demo“，这个环境是可以创多个的，可见fuel可以同时管理多个openstack 环境。这里选择的os 有三种，这里默认选择centos，当然你也可以选择ubuntu 和rhle ，不过rhle 需要手动上传镜像或者提供红帽子官网用户名和密码，fuel 为你自动下载，不过时间比较长，不推荐。

![create openstack env](/images/2013-12-12-quick-setup-openstack-with-fuel-in-30-minutes/create_openstack_env.jpg)

这里选择部署openstack 多节点非HA模式。

![create openstack sa](/images/2013-12-12-quick-setup-openstack-with-fuel-in-30-minutes/create_openstack_sa.jpg)

由于我们是在虚拟机中再跑虚拟机，这里选择hypervisor类型为”qemu“。

![create openstack qemu](/images/2013-12-12-quick-setup-openstack-with-fuel-in-30-minutes/create_openstack_qemu.jpg)

这里选择openstack 的网络部署模式，我们选最简单的方式也是目前最成熟的方式nova-network实现。

![create openstack network](/images/2013-12-12-quick-setup-openstack-with-fuel-in-30-minutes/create_openstack_network.jpg)

最后一路使用default 配置，不做更改完成环境创建。
![create openstack done](/images/2013-12-12-quick-setup-openstack-with-fuel-in-30-minutes/create_openstack_done.jpg)

创建openstack节点虚拟机VM2和VM3，分别命名为fuel_3.2.1_controller和fuel_3.2.1_compute1,注意计算节点多分配写cpu core ，至少2个，内存2G，当然如果机器配置不够也可以1个core 1G内存，至少后边创建openstack的instance比较慢。

![node vms setup](/images/2013-12-12-quick-setup-openstack-with-fuel-in-30-minutes/node_vms_setup.jpg)

设置系统由network启动

![node vms boot from network](/images/2013-12-12-quick-setup-openstack-with-fuel-in-30-minutes/node_vms_boot_from_network.jpg)

配置网卡1，接入net1，注意一定要选择 网卡类型为：Pcnet-PCI II，并且开启混杂模式：Allow All.node_vms_set_promiscuous.jpg

![node vms set promiscuous](/images/2013-12-12-quick-setup-openstack-with-fuel-in-30-minutes/node_vms_set_promiscuous.jpg)

配置网卡2，接入net2.

![node vms create net2](/images/2013-12-12-quick-setup-openstack-with-fuel-in-30-minutes/node_vms_create_net2.jpg)

配置网卡3，接入net3

![node vms create net3](/images/2013-12-12-quick-setup-openstack-with-fuel-in-30-minutes/node_vms_create_net3.jpg)

让后分别启动VM2和VM3
![node vms boot](/images/2013-12-12-quick-setup-openstack-with-fuel-in-30-minutes/node_vms_boot.jpg)

画面出现bootstrap login 后，在fuel web 页面才可以看到节点被fuel发现。
![node vms login](/images/2013-12-12-quick-setup-openstack-with-fuel-in-30-minutes/node_vms_login.jpg)

回到fuel web 可以看到两个节点被发现
![node vms found](/images/2013-12-12-quick-setup-openstack-with-fuel-in-30-minutes/node_vms_found.jpg)

接下来开始针对这两个被发现的节点VM2，VM3配置openstack环境了。
![node vms setup env](/images/2013-12-12-quick-setup-openstack-with-fuel-in-30-minutes/node_vms_setup_env.jpg)


首先需要配置VM2和VM3在openstack 中的角色。点击”add nodes“ 添加VM2作为openstack 的控制节点。

![node role for controller](/images/2013-12-12-quick-setup-openstack-with-fuel-in-30-minutes/node_role_for_controller.jpg)

在点击”add nodes“ 添加VM3作为openstack 的计算节点。
![node role for compute](/images/2013-12-12-quick-setup-openstack-with-fuel-in-30-minutes/node_role_for_compute.jpg)

修改两个节点的物理网卡和openstack 逻辑网络的映射关系，这里只需要拖拽就搞定。admin 网络已经设置到eth0不能再作修改10.20.0.0/24，public和  instance floating 网络共用eth1 且共用同一个地址块172.16.0.0/24，而private ，management 和storage 共用eth2 但是网络ip不同，需要通过vlan tag 方式实现二层网络隔离。

![node network mapping](/images/2013-12-12-quick-setup-openstack-with-fuel-in-30-minutes/node_network_mapping.jpg)

修改两个节点磁盘的分区情况，这里使用默认值，注意storage 分区不能小于10g，否则不能通过验证。
![node storage mapping](/images/2013-12-12-quick-setup-openstack-with-fuel-in-30-minutes/node_storage_mapping.jpg)


再来配置openstack 最复杂的一块网络，其实按照我给的网络拓扑使用默认值就可以安装啦，是不是很方便？不过还是要啰嗦一下：


public IP用于物理机器和外界通信，floating IP 用于动态分配给openstack instance 实现和外界通信。注意这里地址块不能重叠。
由于private，management和storage共用同一网卡且IP块不同,要实现二层隔离就需要打上vlan的tag，如果是接在真实的交换机，必须启用trunk 模式。
一旦网络配置完毕并安装完成，这个地址是永久不能改变的，所以生产环境下一定要先规划好在部署。
配置完成后点击 ”networking verification“ 按钮，检查网络设置是否正确。

![openstack network mapping](/images/2013-12-12-quick-setup-openstack-with-fuel-in-30-minutes/openstack_network_mapping.jpg)

验证通过后保持设置，开始部署节点。

![openstack start deploy](/images/2013-12-12-quick-setup-openstack-with-fuel-in-30-minutes/openstack_start_deploy.jpg)

此时可以发现两个VM开始自动重启开始安装OS。
![openstack start os installation](/images/2013-12-12-quick-setup-openstack-with-fuel-in-30-minutes/openstack_start_os_installation.jpg)

这里比较古怪，安装进度到33%时需要等很久才能往下走。这个时候两个节点的OS都已经安装完成。

![openstack start os installation 33](/images/2013-12-12-quick-setup-openstack-with-fuel-in-30-minutes/openstack_start_os_installation_33.jpg)

有什么办法能看到安装的log呢？当然有，这时候可以去log 标签视图查看安装log，选取”other server“，在选对应节点的puppet log 看log 跳动。
![openstack deploy log](/images/2013-12-12-quick-setup-openstack-with-fuel-in-30-minutes/openstack_deploy_log.jpg)


最后，一切顺利的话，大概20 几分钟安装就会完成了，不过具体时间取决于机器性能，这时候点击http://172.16.0.2 或者 http://10.20.0.4 都可以访问openstack 的dashboard .区别在于172.16.0.2 所谓的公网ip 地址，这个登后dashboard 可以直接使用vnc 访问instance，而10.20.0.4不能。

![openstack deploy done](/images/2013-12-12-quick-setup-openstack-with-fuel-in-30-minutes/openstack_deploy_done.jpg)

点击链接进入openstack登录页面，输入admin/admin

![openstack web login](/images/2013-12-12-quick-setup-openstack-with-fuel-in-30-minutes/openstack_web_login.jpg)

至此，openstack的环境部署完成，这里部署了一个计算节点，一个控制节点。没有部署cinder ，没有部署多计算节点。如果需要部署，请重复上述步骤即可。

最后，就是验证一下openstack环境是否正确部署。其实fuel 有个非常好的而一个功能，可以快速检测openstack 环境”健康“情况。进入healthcheck 标签，可以一键安全检测，注意不会全部都通过，应为cinder 没有安装，所以create volume 相关的服务会失败。
![openstack health check](/images/2013-12-12-quick-setup-openstack-with-fuel-in-30-minutes/openstack_health_check.jpg)

最后我们还是创个instance 来验证吧？

###安装openstack环境验证

先登录后进入openstack主管理界面，创建一个instance，进入project view – > 打开instances tab -> 点击右上方luanch 按钮。instance 名字为test0

![dashboard vm create](/images/2013-12-12-quick-setup-openstack-with-fuel-in-30-minutes/dashboard_vm_create.jpg)

instance 创建成功后，同时点击相应instance test0右边的”more“ 按钮，选择”allocation floating ip“，为其分配一个floating ip 地址。

![dashboard vm floating ip](/images/2013-12-12-quick-setup-openstack-with-fuel-in-30-minutes/dashboard_vm_floating_ip.jpg)

直接在web 页面访问instance： 点击 右端 ”more“ -> “console” 按钮进入该页面，这是是用web socket 技术实现的VNC 客户端，用它可以做一些简单instance 管理，不足是不能粘贴拷贝比较麻烦。

![dashboard vm vnc](/images/2013-12-12-quick-setup-openstack-with-fuel-in-30-minutes/dashboard_vm_vnc.jpg)

最后在笔记本上打开一个”cmd“ 终端看一下floating ip 是否通畅。

![dashboard vm access](/images/2013-12-12-quick-setup-openstack-with-fuel-in-30-minutes/dashboard_vm_access.jpg)

至此fuel web openstack 安装介绍结束，如果要安装更多节点请重复上面操作即可。

其他工具

当然，openstack安装工具不只是有fuel ，还有红帽子的packstack 也是不错的，并且支持最新版本的openstack 安装。这里有篇文章对二者做了比较全面的介绍 http://www.openstack.cn/p383.html。

参考文档：

http://openstack-huawei.github.io/mirantis-openstack/

http://software.mirantis.com/quick-start/

http://www.openstack.cn/p383.html