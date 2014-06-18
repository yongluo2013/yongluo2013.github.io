---
layout:     post
title:      Neutron中的iptables
category: blog
description: iptables在Neutron的开源实现中功不可没
tags: openstack, neutron, iptables
---

声明：  
本博客欢迎转发，但请保留原作者信息!  
博客地址：<http://lingxiankong.github.io/>  
内容系本人及本人团队学习、研究和总结，如有雷同，实属荣幸！  
Author：华为云计算工程师 [孔令贤](http://weibo.com/lingxiankong) 

写在前面：  
因为目前的时间和精力都有限，每天团队里有很多杂事需要处理，博客更新的速度明显慢了许多。好在公司为我们提供了宽松、自由的办公环境，访问外网更加方便，并且因为跟内网隔离，也因此少了不少打扰。记得我在之前的博客中有提到，我不知道我还能坚持多久，但我会尽力。一方面是我促使自己不断学习的过程，另一方面，也希望能给初学者一些帮助，高手，尽量绕行吧，我不敢班门弄斧，但如果能提供一些建议和指正，定当感激不尽！OpenStack目前的形式仍然是如火如荼，各种新的组件，各种培训，各种交流，各种教程，大家的技能水平都有了明显提升，我的教程，也许受众会越来越少，呵呵，这是好事，当大家都是高手的时候，也许交流会更加高效。另外，那些加我QQ或微博咨询问题的朋友，实在抱歉，我来不及一一回答，因为有些问题确实需要登录环境排错，有些问题Google之就会找到比我说出来更加合适的答案，希望你们见谅，也希望你们学会如何自己解决问题，早日修成正果！

作为OpenStack中的网络组件，Neutron提供了面向租户的，L2-L7层的各种服务。作为一种实现原型，Neutron中的linuxbridge和openvswitch（两种plugin），以及其他一些高级服务，都尽可能借用了Linux自身的一些特性，来实现一些复杂的网络模型。其中，iptables功不可没。

## iptables模型
下面是一张iptables的模型图，关于iptables的知识不属于本文的讲解范畴，请自行google之。  
![](/images/2013-11-19-iptables-in-neutron/1.jpg)

## l3-agent
Provides L3/NAT forwarding to provide external network access for VMs on tenant networks。l3 agent接收来自plugin（H版中默认已经使用router service plugin）的RPC消息，来构建3层功能模型，主要处理Router和Floatingip资源对象模型。
 
当系统中的namespace打开时，一个l3 agent可以同时处理多个router，但需要注意的是，一个l3 agent只能处理一个external net上的routers。l3 agent在处理router时，会初始化一些iptables的配置。具体步骤如下：  
1、针对ipv4和ipv6分别都有对应的表，但只有ipv4中有nat表，也说明了目前neutron至少在三层功能上不支持ipv6；  
2、对filter表，增加一个链neutron-filter-top，规则：  

	-A FORWARD -j neutron-filter-top  
	-A OUTPUT -j neutron-filter-top  

增加一个链neutron-l3-agent-local，规则：

	-A neutron-filter-top -j neutron-l3-agent-local  

3、对系统中的内置链进行包装：  
filter表中的三个链：INPUT, OUTPUT, FORWARD  
nat表中的三个链：PREROUTING, OUTPUT, POSTROUTING  
将到达原链的数据包转发到包装链。  
4、对ipv4的nat表，增加一个链neutron-postrouting-bottom，规则：  

	-A POSTROUTING -j neutron-postrouting-bottom  

增加一个链neutron-l3-agent-snat，规则：  

	-A neutron-postrouting-bottom -j neutron-l3-agent-snat 

增加一个链neutron-l3-agent-float-snat，规则：  

	-A neutron-l3-agent-snat -j neutron-l3-agent-float-snat 

5、metadata proxy：  
在ipv4的filter表增加规则：  

	-A neutron-l3-agent-INPUT -s 0.0.0.0/0 -d 127.0.0.1 -p tcp -m tcp --dport 9697 -j ACCEPT 

在ipv4的nat表增加规则：

	-A neutron-l3-agent-PREROUTING -s 0.0.0.0/0 -d 169.254.169.254/32 -p tcp -m tcp --dport 80 -j REDIRECT --to-port 9697

6、根据router中enable_snat属性，看是否有必要设置snat规则：  
首先将neutron-l3-agent-POSTROUTING和neutron-l3-agent-snat链的规则清空，恢复如下规则：  

	-A neutron-l3-agent-snat -j neutron-l3-agent-float-snat

对IPv4的nat表设置：
  
	-A neutron-l3-agent-POSTROUTING ! -i qg-XXX ! -o qg-XXX -m conntrack ! --ctstate DNAT -j ACCEPT   
	-A neutron-l3-agent-snat -s XXX.XXX.XXX.XXX/XX -j SNAT --to-source XXX.XXX.XXX.XXX  

7、处理floatingip  
在IPv4的nat表中增加规则：  

	-A neutron-l3-agent-PREROUTING -d <floatingip> -j DNAT --to <fixedip>  
	-A neutron-l3-agent-OUTPUT -d <floatingip> -j DNAT --to <fixedip>   
	-A neutron-l3-agent-float-snat -s <fixedip> -j SNAT --to <floatingip>

8、应用规则  
iptables-save -c，获取当前所有iptables信息；  
iptables-restore -c *new_input*，应用最新的iptables配置；  

初始化之后，l3 agent节点上的iptables配置如下：  
![](/images/2013-11-19-iptables-in-neutron/2.png)

## security group
目前在neutron中，实现安全组功能的plugin有：Nicira NVP, Open vSwitch, Linux Bridge, NEC, and Ryu。其中，linux bridge和ovs都使用了iptables来实现底层的安全组模型。
 
创建一个port时，可以指定port所属的安全组（若不指定，则加入默认的安全组），此时，因为系统中某个安全组有成员变化，所以需要通知到各个节点，传递这样一个信息：一些安全组中的成员有变化，如果你有对这些安全组的引用，请更新对应的iptables规则。对于linux bridge和ovs来说，需要由neutron l2 agent处理更新请求。
 
首先，l2 agent初始化时，在加载IptablesFirewallDriver时就会初始化一些iptables的配置。  
在v4和v6都会增加自定义链，很多链的命名规则与l3 agent节点相似，此外会新增：  
neutron-l2-agent-sg-fallback，默认丢弃所有包；  
其次，l2 agent发现有新增设备（port）时，会调用plugin的security-group-rules-for-devices方法获取port的安全组规则信息（规则信息里面已经考虑了允许dhcp, address-pair, ），返回的信息比较全，包括了：

	{
		id
		name
		network_id
		tenant_id
		mac_address
		admin_state_up
		status
		fixed_ips
		device_id
		device_owner
		security_groups
		security_group_rules:
			{
				security_group_id
				direction
				ethertype
				protocol
				port_range_min
				port_range_max
				source_ip_prefix/dest_ip_prefix
				remote_group_id
			}
		security_group_source_groups
		device
	}

，然后重新配置本节点的iptables的filter表，主要关注INPUT和FORWARD链：  
1、增加neutron-l2-agent-sg-chain链  
2、每个port对应两条链(iX-XXXX和oX-XXXX)，同时:  

	-A neutron-l2-agent-FORWARD -m physdev --physdev-out <port-id> --physdev-is-bridged -j neutron-l2-agent-sg-chain  
	-A neutron-l2-agent-sg-chain -m physdev --physdev-out <port-id> --physdev-is-bridged -j iX-XXXX  
	-A neutron-l2-agent-FORWARD -m physdev --physdev-in <port-id> --physdev-is-bridged -j neutron-l2-agent-sg-chain  
	-A neutron-l2-agent-sg-chain -m physdev --physdev-in <port-id> --physdev-is-bridged -j oX-XXXX  
	-A neutron-l2-agent-INPUT -m physdev --physdev-in <port-id> --physdev-is-bridged -j oX-XXXX  
	-A iX-XXXX … -j RETURN  
	-A iX-XXXX -j neutron-l2-agent-sg-fallback  
	-A oX-XXXX … -j RETURN  
	-A oX-XXXX -j neutron-l2-agent-sg-fallback  
	-A neutron-l2-agent-sg-chain -j ACCEPT

引用《Iptables Security Group Implementation》中的截图：  
![](/images/2013-11-19-iptables-in-neutron/3.jpg)

举一个简单的例子。环境上信息如下：  

	root@C16-RH2285-01-openstack-controlor-H:~# nova list
	+--------------------------------------+----------+--------+------------+-------------+--------------------------------+
	| ID                                   | Name     | Status | Task State | Power State | Networks                       |
	+--------------------------------------+----------+--------+------------+-------------+--------------------------------+
	| abb62736-506e-45fb-ad40-1a8b2a76d314 | shz_vm01 | ACTIVE | None       | Running     | test=10.10.10.2, 200.200.128.3 |
	+--------------------------------------+----------+--------+------------+-------------+--------------------------------+
	root@C16-RH2285-01-openstack-controlor-H:~# neutron port-list -- --device_id=abb62736-506e-45fb-ad40-1a8b2a76d314
	+--------------------------------------+------+-------------------+-----------------------------------------------------------------------------------+
	| id                                   | name | mac_address       | fixed_ips                                                                         |
	+--------------------------------------+------+-------------------+-----------------------------------------------------------------------------------+
	| 6bc18a88-9619-4fce-bf4e-49cae6df1715 |      | fa:16:3e:31:e3:11 | {"subnet_id": "e049dac2-dc76-4187-bf37-d0675ac44491", "ip_address": "10.10.10.2"} |
	+--------------------------------------+------+-------------------+-----------------------------------------------------------------------------------+
	root@C16-RH2285-01-openstack-controlor-H:~# neutron port-show 6bc18a88-9619-4fce-bf4e-49cae6df1715
	+-----------------------+-----------------------------------------------------------------------------------+
	| Field                 | Value                                                                             |
	+-----------------------+-----------------------------------------------------------------------------------+
	| admin_state_up        | True                                                                              |
	| allowed_address_pairs |                                                                                   |
	| binding:capabilities  | {"port_filter": true}                                                             |
	| binding:host_id       | C17-T6000-S11-1-nova-compute-H                                                    |
	| binding:vif_type      | ovs                                                                               |
	| device_id             | abb62736-506e-45fb-ad40-1a8b2a76d314                                              |
	| device_owner          | compute:nova                                                                      |
	| extra_dhcp_opts       |                                                                                   |
	| fixed_ips             | {"subnet_id": "e049dac2-dc76-4187-bf37-d0675ac44491", "ip_address": "10.10.10.2"} |
	| id                    | 6bc18a88-9619-4fce-bf4e-49cae6df1715                                              |
	| mac_address           | fa:16:3e:31:e3:11                                                                 |
	| name                  |                                                                                   |
	| network_id            | 46e11e98-0275-4a54-ac1a-bd1c71308be7                                              |
	| security_groups       | f01c3cea-31f7-4659-ae14-eb7e97af2175                                              |
	| status                | ACTIVE                                                                            |
	| tenant_id             | 1b6e214ab4594fd9b57017fa0af0ff62                                                  |
	+-----------------------+-----------------------------------------------------------------------------------+
	root@C16-RH2285-01-openstack-controlor-H:~# neutron security-group-show f01c3cea-31f7-4659-ae14-eb7e97af2175
	+----------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
	| Field                | Value                                                                                                                                                                                                                                                                                                                                                            |
	+----------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
	| description          | default                                                                                                                                                                                                                                                                                                                                                          |
	| id                   | f01c3cea-31f7-4659-ae14-eb7e97af2175                                                                                                                                                                                                                                                                                                                             |
	| name                 | default                                                                                                                                                                                                                                                                                                                                                          |
	| security_group_rules | {"remote_group_id": null, "direction": "egress", "remote_ip_prefix": null, "protocol": null, "tenant_id": "1b6e214ab4594fd9b57017fa0af0ff62", "port_range_max": null, "security_group_id": "f01c3cea-31f7-4659-ae14-eb7e97af2175", "port_range_min": null, "ethertype": "IPv6", "id": "097570bf-16e2-4dcc-8cfc-77a6407dc952"}                                    |
	|                      | {"remote_group_id": null, "direction": "ingress", "remote_ip_prefix": "0.0.0.0/0", "protocol": "tcp", "tenant_id": "1b6e214ab4594fd9b57017fa0af0ff62", "port_range_max": 65535, "security_group_id": "f01c3cea-31f7-4659-ae14-eb7e97af2175", "port_range_min": 1, "ethertype": "IPv4", "id": "2bf104fc-7f08-4de1-afdf-ee6ad175b16b"}                             |
	|                      | {"remote_group_id": null, "direction": "egress", "remote_ip_prefix": null, "protocol": null, "tenant_id": "1b6e214ab4594fd9b57017fa0af0ff62", "port_range_max": null, "security_group_id": "f01c3cea-31f7-4659-ae14-eb7e97af2175", "port_range_min": null, "ethertype": "IPv4", "id": "30e7ae12-28e9-42a8-a004-3382b9a204ca"}                                    |
	|                      | {"remote_group_id": "f01c3cea-31f7-4659-ae14-eb7e97af2175", "direction": "ingress", "remote_ip_prefix": null, "protocol": null, "tenant_id": "1b6e214ab4594fd9b57017fa0af0ff62", "port_range_max": null, "security_group_id": "f01c3cea-31f7-4659-ae14-eb7e97af2175", "port_range_min": null, "ethertype": "IPv4", "id": "692482b7-6842-46bf-b234-1b005387be30"} |
	|                      | {"remote_group_id": "f01c3cea-31f7-4659-ae14-eb7e97af2175", "direction": "ingress", "remote_ip_prefix": null, "protocol": null, "tenant_id": "1b6e214ab4594fd9b57017fa0af0ff62", "port_range_max": null, "security_group_id": "f01c3cea-31f7-4659-ae14-eb7e97af2175", "port_range_min": null, "ethertype": "IPv6", "id": "7433cfaf-3fba-45a4-a00f-32378588ae0f"} |
	|                      | {"remote_group_id": null, "direction": "ingress", "remote_ip_prefix": "0.0.0.0/0", "protocol": "icmp", "tenant_id": "1b6e214ab4594fd9b57017fa0af0ff62", "port_range_max": null, "security_group_id": "f01c3cea-31f7-4659-ae14-eb7e97af2175", "port_range_min": null, "ethertype": "IPv4", "id": "9319c51b-a21d-4872-b5bc-5e6d084eed5c"}                          |
	| tenant_id            | 1b6e214ab4594fd9b57017fa0af0ff62  

登录到虚拟机所在的节点，查看iptables规则（主要关注filter表）：

	*filter
	:INPUT ACCEPT [1247508:199602603]
	:FORWARD ACCEPT [60:7484]
	:OUTPUT ACCEPT [1460711:279980783]
	:neutron-filter-top - [0:0]
	:neutron-openvswi-FORWARD - [0:0]
	:neutron-openvswi-INPUT - [0:0]
	:neutron-openvswi-OUTPUT - [0:0]
	:neutron-openvswi-i6bc18a88-9 - [0:0]
	:neutron-openvswi-local - [0:0]
	:neutron-openvswi-o6bc18a88-9 - [0:0]
	:neutron-openvswi-s6bc18a88-9 - [0:0]
	:neutron-openvswi-sg-chain - [0:0]
	:neutron-openvswi-sg-fallback - [0:0]
	-A INPUT -j neutron-openvswi-INPUT
	-A FORWARD -j neutron-filter-top
	-A FORWARD -j neutron-openvswi-FORWARD
	-A OUTPUT -j neutron-filter-top
	-A OUTPUT -j neutron-openvswi-OUTPUT
	-A neutron-filter-top -j neutron-openvswi-local
	-A neutron-openvswi-FORWARD -m physdev --physdev-out tap6bc18a88-96 --physdev-is-bridged -j neutron-openvswi-sg-chain
	-A neutron-openvswi-FORWARD -m physdev --physdev-in tap6bc18a88-96 --physdev-is-bridged -j neutron-openvswi-sg-chain
	-A neutron-openvswi-INPUT -m physdev --physdev-in tap6bc18a88-96 --physdev-is-bridged -j neutron-openvswi-o6bc18a88-9
	-A neutron-openvswi-i6bc18a88-9 -m state --state INVALID -j DROP
	-A neutron-openvswi-i6bc18a88-9 -m state --state RELATED,ESTABLISHED -j RETURN
	-A neutron-openvswi-i6bc18a88-9 -p tcp -m tcp -m multiport --dports 1:65535 -j RETURN
	-A neutron-openvswi-i6bc18a88-9 -p icmp -j RETURN
	-A neutron-openvswi-i6bc18a88-9 -s 10.10.10.3/32 -p udp -m udp --sport 67 --dport 68 -j RETURN
	-A neutron-openvswi-i6bc18a88-9 -j neutron-openvswi-sg-fallback
	-A neutron-openvswi-o6bc18a88-9 -p udp -m udp --sport 68 --dport 67 -j RETURN
	-A neutron-openvswi-o6bc18a88-9 -j neutron-openvswi-s6bc18a88-9
	-A neutron-openvswi-o6bc18a88-9 -p udp -m udp --sport 67 --dport 68 -j DROP
	-A neutron-openvswi-o6bc18a88-9 -m state --state INVALID -j DROP
	-A neutron-openvswi-o6bc18a88-9 -m state --state RELATED,ESTABLISHED -j RETURN
	-A neutron-openvswi-o6bc18a88-9 -j RETURN
	-A neutron-openvswi-o6bc18a88-9 -j neutron-openvswi-sg-fallback
	-A neutron-openvswi-s6bc18a88-9 -s 10.10.10.2/32 -m mac --mac-source FA:16:3E:31:E3:11 -j RETURN
	-A neutron-openvswi-s6bc18a88-9 -j DROP
	-A neutron-openvswi-sg-chain -m physdev --physdev-out tap6bc18a88-96 --physdev-is-bridged -j neutron-openvswi-i6bc18a88-9
	-A neutron-openvswi-sg-chain -m physdev --physdev-in tap6bc18a88-96 --physdev-is-bridged -j neutron-openvswi-o6bc18a88-9
	-A neutron-openvswi-sg-chain -j ACCEPT
	-A neutron-openvswi-sg-fallback -j DROP
	COMMIT

## firewall as a service
既然有了安全组的功能，为什么还要firewall呢？下面引用自fwaas-spec-v0.1：  
>the security group rules lack the ability to express application characteristics which some of the next generation firewalls do. The Security Groups feature also does not support a workflow that allows providers to publish a collection of audited rules, and which a tenant can choose to apply in his network (say for security compliance reasons). In general, the Security Groups feature is Quantum port-centric which does not address the need to manage and leverage the richer security features provided by an edge Firewall service. 

firewall功能借用了l3 agent来实现，当前firewall有两种实现：Linux Iptables和varmour，这里仅关注Linux。创建firewall时，会对租户的所有router关联的iptables规则进行配置，firewall在接口和操作上同时支持IPv4和IPv6。要使用firewall功能最好启用namespace，否则就失去了面向租户的意义。即：一个租户可以创建firewall，来限制他的所有network中的设备的数据通信。  
1、首先移除与firewall相关联的链；  
2、在filter表增加链neutron-l3-agent-fwaas-default-policy及规则：  
-A neutron-l3-agent-fwaas-default-policy -j DROP  
3、操作filter表  
增加链：neutron-l3-agent-iv4XXXXXXXX-XXXX(XXX为firewall的ID)，和规则：  

	-A neutron-l3-agent-iv4XXXXXXXX-XXXX -m state --state INVALID -j DROP  
	-A neutron-l3-agent-iv4XXXXXXXX-XXXX -m state --state ESTABLISHED,RELATED -j ACCEPT  
	-A neutron-l3-agent-iv4XXXXXXXX-XXXX … -j ACCEPT/DROP  

增加链：neutron-l3-agent-ov4XXXXXXXX-XXXX，和规则：  

	-A neutron-l3-agent-ov4XXXXXXXX-XXXX -m state --state INVALID -j DROP  
	-A neutron-l3-agent-ov4XXXXXXXX-XXXX -m state --state ESTABLISHED,RELATED -j ACCEPT  
	-A neutron-l3-agent-ov4XXXXXXXX-XXXX … -j ACCEPT/DROP  

4、设置其他规则  

	-A neutron-l3-agent-FORWARD -i qr-+ -j neutron-l3-agent-iv4XXXXXXXX-XXXX  
	-A neutron-l3-agent-FORWARD -o qr-+ -j neutron-l3-agent-ov4XXXXXXXX-XXXX  
	-A neutron-l3-agent-FORWARD -o qr-+ -j neutron-l3-agent-fwaas-default-policy  
	-A neutron-l3-agent-FORWARD -i qr-+ -j neutron-l3-agent-fwaas-default-policy  

可见，目前firewall的实现是在每个租户所拥有的路由的边缘进行iptables配置，关注filter表的FORWARD链，后续的I版可能会推出Zone的概念。

我们举个简单例子更能说明问题。假设系统中租户有一个虚拟机abb62736-506e-45fb-ad40-1a8b2a76d314，对应的port-id：6bc18a88-9619-4fce-bf4e-49cae6df1715，fixedip：10.10.10.2，floatingip：200.200.128.3，登录虚拟机内对floatingip所在的域内任意存在的IP地址进行ping操作，可以ping通。然后，租户创建firewall，禁止所有的icmp数据包，步骤如下：

	root@C16-RH2285-01-openstack-controlor-H:~# neutron firewall-rule-create --protocol icmp --action deny
	Created a new firewall_rule:
	+------------------------+--------------------------------------+
	| Field                  | Value                                |
	+------------------------+--------------------------------------+
	| action                 | deny                                 |
	| description            |                                      |
	| destination_ip_address |                                      |
	| destination_port       |                                      |
	| enabled                | True                                 |
	| firewall_policy_id     |                                      |
	| id                     | 138cd8cd-b83d-433e-b673-9c8af3109c74 |
	| ip_version             | 4                                    |
	| name                   |                                      |
	| position               |                                      |
	| protocol               | icmp                                 |
	| shared                 | False                                |
	| source_ip_address      |                                      |
	| source_port            |                                      |
	| tenant_id              | 1b6e214ab4594fd9b57017fa0af0ff62     |
	+------------------------+--------------------------------------+
	root@C16-RH2285-01-openstack-controlor-H:~# neutron firewall-policy-create --firewall-rules 138cd8cd-b83d-433e-b673-9c8af3109c74 kong_test_policy
	Created a new firewall_policy:
	+----------------+--------------------------------------+
	| Field          | Value                                |
	+----------------+--------------------------------------+
	| audited        | False                                |
	| description    |                                      |
	| firewall_rules | 138cd8cd-b83d-433e-b673-9c8af3109c74 |
	| id             | 77ea7104-557c-47bb-9b2f-810d70897c43 |
	| name           | kong_test_policy                     |
	| shared         | False                                |
	| tenant_id      | 1b6e214ab4594fd9b57017fa0af0ff62     |
	+----------------+--------------------------------------+
	root@C16-RH2285-01-openstack-controlor-H:~# neutron firewall-create  77ea7104-557c-47bb-9b2f-810d70897c43
	Created a new firewall:
	+--------------------+--------------------------------------+
	| Field              | Value                                |
	+--------------------+--------------------------------------+
	| admin_state_up     | True                                 |
	| description        |                                      |
	| firewall_policy_id | 77ea7104-557c-47bb-9b2f-810d70897c43 |
	| id                 | 80daec7b-8167-459e-b915-da02832a248b |
	| name               |                                      |
	| status             | PENDING_CREATE                       |
	| tenant_id          | 1b6e214ab4594fd9b57017fa0af0ff62     |
	+--------------------+--------------------------------------+
	root@C16-RH2285-01-openstack-controlor-H:~# neutron firewall-show 80daec7b-8167-459e-b915-da02832a248b
	+--------------------+--------------------------------------+
	| Field              | Value                                |
	+--------------------+--------------------------------------+
	| admin_state_up     | True                                 |
	| description        |                                      |
	| firewall_policy_id | 77ea7104-557c-47bb-9b2f-810d70897c43 |
	| id                 | 80daec7b-8167-459e-b915-da02832a248b |
	| name               |                                      |
	| status             | ACTIVE                               |
	| tenant_id          | 1b6e214ab4594fd9b57017fa0af0ff62     |
	+--------------------+--------------------------------------+

执行上述步骤之后，在登录虚拟机内ping同样的IP地址，发现已经不通了。登录l3-agent所在节点，在对应的namespace内查看iptables规则（仅关注filter表）：

	root@network-H:~# ip netns exec qrouter-b4124c09-680e-4427-b758-767c8ffe794b iptables-save
	# Generated by iptables-save v1.4.12 on Tue Oct 29 05:55:51 2013
	*filter
	:INPUT ACCEPT [236:61046]
	:FORWARD ACCEPT [84:10200]
	:OUTPUT ACCEPT [31:2396]
	:neutron-filter-top - [0:0]
	:neutron-l3-agent-FORWARD - [0:0]
	:neutron-l3-agent-INPUT - [0:0]
	:neutron-l3-agent-OUTPUT - [0:0]
	:neutron-l3-agent-fwaas-defau - [0:0]
	:neutron-l3-agent-iv480daec7b - [0:0]
	:neutron-l3-agent-local - [0:0]
	:neutron-l3-agent-ov480daec7b - [0:0]
	-A INPUT -j neutron-l3-agent-INPUT
	-A FORWARD -j neutron-filter-top
	-A FORWARD -j neutron-l3-agent-FORWARD
	-A OUTPUT -j neutron-filter-top
	-A OUTPUT -j neutron-l3-agent-OUTPUT
	-A neutron-filter-top -j neutron-l3-agent-local
	-A neutron-l3-agent-FORWARD -o qr-+ -j neutron-l3-agent-iv480daec7b
	-A neutron-l3-agent-FORWARD -i qr-+ -j neutron-l3-agent-ov480daec7b
	-A neutron-l3-agent-FORWARD -o qr-+ -j neutron-l3-agent-fwaas-defau
	-A neutron-l3-agent-FORWARD -i qr-+ -j neutron-l3-agent-fwaas-defau
	-A neutron-l3-agent-INPUT -d 127.0.0.1/32 -p tcp -m tcp --dport 8775 -j ACCEPT
	-A neutron-l3-agent-fwaas-defau -j DROP
	-A neutron-l3-agent-iv480daec7b -m state --state INVALID -j DROP
	-A neutron-l3-agent-iv480daec7b -m state --state RELATED,ESTABLISHED -j ACCEPT
	-A neutron-l3-agent-iv480daec7b -p icmp -j DROP
	-A neutron-l3-agent-ov480daec7b -m state --state INVALID -j DROP
	-A neutron-l3-agent-ov480daec7b -m state --state RELATED,ESTABLISHED -j ACCEPT
	-A neutron-l3-agent-ov480daec7b -p icmp -j DROP

## 总结
随着Neutron中的高级服务越来越多，也许每个节点（特别是网络节点）上的iptables规则会越来越复杂，定位一个网络问题的时间成本也会随之增加。但iptables作为Linux内核默认提供的功能，是可以满足一些简单场景的要求的，同时也对快速部署提供了可能性。当然，我想这是大部分网络厂商的机会，大家可以在自有产品之上提供自己的实现，借助OpenStack，来带动自身产品的销售，甚至可以提供Neutron没有的功能作为差异化特性，提升自己的竞争力。Linux bridge和openvSwitch（乃至后面的ML2）其实是提供一种参考实现，大家在实现自己Plugin或Driver的同时，其实是有大把的现成的代码可以拷贝的，:)