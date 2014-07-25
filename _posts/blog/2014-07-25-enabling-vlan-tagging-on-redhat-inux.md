---
layout: post
title: Enabling VLAN tagging on Redhat Linux
category: blog
description: I needed a physical machine with one NIC to be able to have two different IP addresses on two different VLAN’s.
---

声明：本博客欢迎转发，但请保留原作者信息!      
作者：[罗勇] 云计算工程师、敏捷开发实践者    
博客：[http://yongluo2013.github.io/](http://yongluo2013.github.io/)    
微博：[http://weibo.com/u/1704250760/](http://weibo.com/u/1704250760/)  

The post from [http://technodrone.blogspot.com/2011/10/enabling-vlan-tagging-on-redhat-linux.html](http://technodrone.blogspot.com/2011/10/enabling-vlan-tagging-on-redhat-linux.html)    

I came across this one today, and am putting it here to document it for my own benefit. I needed a physical machine with one NIC to be able to have two different IP addresses on two different VLAN’s.

On Windows I am not sure if that is possible by default.

So how would you do it on Redhat Linux (taken from Howto: Configure Linux Virtual Local Area Network (VLAN))

This is the scenario. One network card with an IP of 192.168.10.1 (VLAN 10). I needed another interface on the same physical NIC with an IP address of 192.168.20.x (VLAN 20).

First you copy the network configuration

cp /etc/sysconfig/network-scripts/ifcfg-eth0 /etc/sysconfig/network-scripts/ifcfg-eth0.10

My original file looked like this:

```
DEVICE=eth0
ONBOOT=yes
BOOTPROTO=none
NETMASK=255.255.255.0
IPADDR=192.168.10.1
USERCTL=no
PEERDNS=yes
TYPE=Ethernet
IPV6INIT=no

```

I needed to add in the VLAN info and change the device name (add VLAN=yes) 

```
DEVICE=eth0.10
ONBOOT=yes
VLAN=yes
BOOTPROTO=none
NETMASK=255.255.255.0
IPADDR=192.168.10.1
USERCTL=no
PEERDNS=yes
TYPE=Ethernet
IPV6INIT=no
```
I then copied the file again to my second interface 

cp /etc/sysconfig/network-scripts/ifcfg-eth0.10 /etc/sysconfig/network-scripts/ifcfg-eth0.20

and changed the IP address and device name

```
DEVICE=eth0.20
ONBOOT=yes
VLAN=yes
BOOTPROTO=none
NETMASK=255.255.255.0
IPADDR=192.168.20.1
USERCTL=no
PEERDNS=yes
TYPE=Ethernet
IPV6INIT=no
```

I then removed the original IP address information from 
/etc/sysconfig/network-scripts/ifcfg-eth0

```
DEVICE=eth0
ONBOOT=yes
BOOTPROTO=none
USERCTL=no
PEERDNS=yes
TYPE=Ethernet
IPV6INIT=no
```

And restart the network service

```
service network restart

```
Of course the configuration has to be done as well on the switch side as well to allow the trunk of both VLAN’s

Switch#(config)interface Gi3/41
Switch#(config-if)no switchport mode access
Switch#(config-if)switchport mode trunk
Switch#(config-if)switchport trunk allowed vlan 10,20     
