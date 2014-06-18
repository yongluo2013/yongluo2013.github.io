---
layout: post
title: Huawei Icehouse All in One Operation Guide
description: Huawei Icehouse All in One Operation Guide
category: blog
---

声明：  
本博客欢迎转发，但请保留原作者信息!  
新浪微博：[@孔令贤HW](http://weibo.com/lingxiankong)；   
博客地址：<http://lingxiankong.github.io/>  
内容系本人及本人团队学习、研究和总结，如有雷同，实属荣幸！

----------------

之前一篇[博客](http://lingxiankong.github.io/blog/2014/04/29/openstack-icehouse-allinone/)介绍了华为发布的针对Icehouse版本的all in one安装镜像，因为OpenStack安装后还需要进行一系列的初始配置(大家的环境和需求各异，因此没有直接集成在自动化中)。对于小白用户来说，可能还不能满足需求，今天我就一步一步教大家入门。

## 准备和注册镜像、Keypair  
拷贝所需的镜像到服务器的任意目录(比如/home/images)，我这里有3个镜像

    UVP:/home/images # ll
    total 683292
    -rw-r--r-- 1 root root 476704768 Mar 24 20:33 F17-x86_64-cfntools.qcow2
    -rw-r--r-- 1 root root   9159168 Jan 14  2013 cirros-0.3.0-i386-disk.img
    -rw-r--r-- 1 root root 213123072 Jan 14  2013 precise-server-cloudimg-i386-disk1.img
    
注册(注意，在最后一步，我添加了一个密钥并设置了正确的权限)：

    glance image-create --name="Ubuntu 12.04 cloudimg i386" --public --container-format=ovf --disk-format=qcow2 < /home/images/precise-server-cloudimg-i386-disk1.img
    glance image-create --name=cirros --public  --container-format=bare --disk-format=qcow2 < /home/images/cirros-0.3.0-i386-disk.img
    glance image-create --name="F17-x86_64-cfntools" --public --container-format=ovf --disk-format=qcow2 < /home/images/F17-x86_64-cfntools.qcow2
    nova keypair-add heathey > heatkey.pem
    chmod 600 heatkey.pem
    
## 创建网络  
all in one的安装要求服务器有三个网卡，我的服务器eth0所在的网段是172.25.0.0/16，eth2(默认br-ex连接的网卡，all in one配置下，eth1其实用不到)所连接汇聚交换机端口上配置的网关为200.200.1.1/16，因此给Neutron分配的floatingIP段就比较随意了，我这里是200.200.200.100-120

    UVP:/home/kong # neutron net-create external_net1 --router:external=True
    Created a new network:
    +---------------------------+--------------------------------------+
    | Field                     | Value                                |
    +---------------------------+--------------------------------------+
    | admin_state_up            | True                                 |
    | id                        | ded0dd2b-440a-4e9c-ba38-6df184747741 |
    | name                      | external_net1                        |
    | provider:network_type     | vlan                                 |
    | provider:physical_network | physnet                              |
    | provider:segmentation_id  | 51                                   |
    | router:external           | True                                 |
    | shared                    | False                                |
    | status                    | ACTIVE                               |
    | subnets                   |                                      |
    | tenant_id                 | 57314a8980b34c6ba3ece5cd19ff6214     |
    +---------------------------+--------------------------------------+
    UVP:/home/kong # neutron subnet-create external_net1 200.200.0.0/16 --name=external_subnet1 --allocation-pool start=200.200.200.100,end=200.200.200.120 --gateway_ip 200.200.1.1 --enable_dhcp=False
    Created a new subnet:
    +------------------+--------------------------------------------------------+
    | Field            | Value                                                  |
    +------------------+--------------------------------------------------------+
    | allocation_pools | {"start": "200.200.200.100", "end": "200.200.200.120"} |
    | cidr             | 200.200.0.0/16                                         |
    | dns_nameservers  |                                                        |
    | enable_dhcp      | False                                                  |
    | gateway_ip       | 200.200.1.1                                            |
    | host_routes      |                                                        |
    | id               | 5e011976-d15b-45e3-8a63-62f98f8d4a42                   |
    | ip_version       | 4                                                      |
    | name             | external_subnet1                                       |
    | network_id       | ded0dd2b-440a-4e9c-ba38-6df184747741                   |
    | tenant_id        | 57314a8980b34c6ba3ece5cd19ff6214                       |
    +------------------+--------------------------------------------------------+
    UVP:/home/kong # neutron net-create demo_net1
    Created a new network:
    +---------------------------+--------------------------------------+
    | Field                     | Value                                |
    +---------------------------+--------------------------------------+
    | admin_state_up            | True                                 |
    | id                        | 10504fd5-9c8e-4781-856f-a6d575be88a7 |
    | name                      | demo_net1                            |
    | provider:network_type     | vlan                                 |
    | provider:physical_network | physnet                              |
    | provider:segmentation_id  | 52                                   |
    | shared                    | False                                |
    | status                    | ACTIVE                               |
    | subnets                   |                                      |
    | tenant_id                 | 57314a8980b34c6ba3ece5cd19ff6214     |
    +---------------------------+--------------------------------------+
    UVP:/home/kong # neutron subnet-create demo_net1 10.1.1.0/24 --name=demo_subnet1 --gateway_ip 10.1.1.1
    Created a new subnet:
    +------------------+--------------------------------------------+
    | Field            | Value                                      |
    +------------------+--------------------------------------------+
    | allocation_pools | {"start": "10.1.1.2", "end": "10.1.1.254"} |
    | cidr             | 10.1.1.0/24                                |
    | dns_nameservers  |                                            |
    | enable_dhcp      | True                                       |
    | gateway_ip       | 10.1.1.1                                   |
    | host_routes      |                                            |
    | id               | a03be99b-b78f-46d0-9311-e45fd8bcf92d       |
    | ip_version       | 4                                          |
    | name             | demo_subnet1                               |
    | network_id       | 10504fd5-9c8e-4781-856f-a6d575be88a7       |
    | tenant_id        | 57314a8980b34c6ba3ece5cd19ff6214           |
    +------------------+--------------------------------------------+
    UVP:/home/kong # neutron router-create demo_router1
    Created a new router:
    +-----------------------+--------------------------------------+
    | Field                 | Value                                |
    +-----------------------+--------------------------------------+
    | admin_state_up        | True                                 |
    | external_gateway_info |                                      |
    | id                    | 2e9945aa-f636-468e-8d76-8386e3dfabd0 |
    | name                  | demo_router1                         |
    | status                | ACTIVE                               |
    | tenant_id             | 57314a8980b34c6ba3ece5cd19ff6214     |
    +-----------------------+--------------------------------------+
    UVP:/home/kong # neutron router-interface-add 2e9945aa-f636-468e-8d76-8386e3dfabd0 a03be99b-b78f-46d0-9311-e45fd8bcf92d
    Added interface 87d22f0f-ba93-49f5-99e7-eb8a5bcb90a3 to router 2e9945aa-f636-468e-8d76-8386e3dfabd0.
    UVP:/home/kong # neutron router-gateway-set 2e9945aa-f636-468e-8d76-8386e3dfabd0 ded0dd2b-440a-4e9c-ba38-6df184747741
    Set gateway for router 2e9945aa-f636-468e-8d76-8386e3dfabd0
    
## 创建虚拟机并验证VNC登陆  

    UVP:/home/kong # nova boot cirros_vm --image 414bcb4d-8e68-4dfe-8d21-31e9bb3d26fd --flavor 1 --nic net-id=10504fd5-9c8e-4781-856f-a6d575be88a7 --key-name heathey
    UVP:/home/kong # nova get-vnc-console 735f0e66-19f9-40b1-9c13-4588051d7270  novnc
    +-------+-----------------------------------------------------------------------------------+
    | Type  | Url                                                                               |
    +-------+-----------------------------------------------------------------------------------+
    | novnc | http://172.25.150.8:6080/vnc_auto.html?token=a4b0e28f-da83-464d-bbe0-0de97bbb22dd |
    +-------+-----------------------------------------------------------------------------------+

通过VNC登录虚拟机后，查看虚拟机的IP(证明系统的DHCP功能正常)：  
![](/images/2014-05-12-huawei-allinone-operation-guide/1.png)  

配置安全组允许PING和SSH登录：

    UVP:/home/kong # nova secgroup-add-rule default tcp 22 22 0.0.0.0/0
    +-------------+-----------+---------+-----------+--------------+
    | IP Protocol | From Port | To Port | IP Range  | Source Group |
    +-------------+-----------+---------+-----------+--------------+
    | tcp         | 22        | 22      | 0.0.0.0/0 |              |
    +-------------+-----------+---------+-----------+--------------+
    UVP:/home/kong # nova secgroup-add-rule default icmp -1 -1 0.0.0.0/0
    +-------------+-----------+---------+-----------+--------------+
    | IP Protocol | From Port | To Port | IP Range  | Source Group |
    +-------------+-----------+---------+-----------+--------------+
    | icmp        | -1        | -1      | 0.0.0.0/0 |              |
    +-------------+-----------+---------+-----------+--------------+
    
还记得之前添加的密钥么，现在对虚拟机进行PING和SSH登录操作(10.1.1.4是我的虚拟机的私有IP)，功能OK：

    UVP:/home/kong # ip netns list
    qrouter-2e9945aa-f636-468e-8d76-8386e3dfabd0
    qdhcp-10504fd5-9c8e-4781-856f-a6d575be88a7
    UVP:/home/kong # ip netns exec qdhcp-10504fd5-9c8e-4781-856f-a6d575be88a7 ping 10.1.1.4
    PING 10.1.1.4 (10.1.1.4) 56(84) bytes of data.
    64 bytes from 10.1.1.4: icmp_seq=1 ttl=64 time=0.683 ms
    64 bytes from 10.1.1.4: icmp_seq=2 ttl=64 time=0.540 ms
    ^C
    --- 10.1.1.4 ping statistics ---
    2 packets transmitted, 2 received, 0% packet loss, time 1002ms
    rtt min/avg/max/mdev = 0.540/0.611/0.683/0.075 ms
    UVP:/home/kong # ip netns exec qdhcp-10504fd5-9c8e-4781-856f-a6d575be88a7 ssh -i heatkey.pem cirros@10.1.1.4
    $ ifconfig
    eth0      Link encap:Ethernet  HWaddr FA:16:3E:5C:F7:1B  
              inet addr:10.1.1.4  Bcast:10.1.1.255  Mask:255.255.255.0
              inet6 addr: fe80::f816:3eff:fe5c:f71b/64 Scope:Link
              UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
              RX packets:111 errors:0 dropped:0 overruns:0 frame:0
              TX packets:106 errors:0 dropped:0 overruns:0 carrier:0
              collisions:0 txqueuelen:1000 
              RX bytes:16457 (16.0 KiB)  TX bytes:12468 (12.1 KiB)
              Interrupt:11 Base address:0xa000 
    
    lo        Link encap:Local Loopback  
              inet addr:127.0.0.1  Mask:255.0.0.0
              inet6 addr: ::1/128 Scope:Host
              UP LOOPBACK RUNNING  MTU:16436  Metric:1
              RX packets:0 errors:0 dropped:0 overruns:0 frame:0
              TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
              collisions:0 txqueuelen:0 
              RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)


