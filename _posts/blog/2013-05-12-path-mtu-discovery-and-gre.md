---
layout: post
title: Path MTU discovery and GRE
category: blog
description: When using the Open vSwitch Quantum plugin with GRE there can be problems connecting to some sites while other sites work fine. 
---

The post from  http://techbackground.blogspot.co.uk/2013/06/path-mtu-discovery-and-gre.html

##The ISSUES

When using the Open vSwitch Quantum plugin with GRE there can be problems connecting to some sites while other sites work fine. E.g. ‘git clone’ hangs or ‘apt-get update’ takes over an hour. This is because the GRE tunnel can reduce the path MTU to a value less than the common 1500 bytes, and some sites do not engage in standard techniques for path MTU discovery.

In a multinode installation there is a GRE tunnel between the network node running the Quantum router and the compute node running the instance. Layer-2 packets between the router and the instance are encapsulated in IP/GRE packets and put on the physical network. They are received by the other node, de-encapsulated, tagged and put onto bridge br-int. This process affects the path MTU – see here for a good explanation of PMTUD (Path MTU Discovery).

Running traceroute with ‘–mtu’ shows the PMTU between the router and the instance to be 1454 bytes:

```
 root@netnode:/# ip netns exec qrouter-7a44de32-3ac0-4f3e-92cc-1a37d8211db8 traceroute -n 172.17.17.3 -p 44444 --mtu 

```
 traceroute to 172.17.17.3 (172.17.17.3), 30 hops max, 65000 byte packets 1 172.17.17.3 80.197 ms F=1454 2.334 ms 44.973 ms 
Note: I ran traceroute from the router here as the traceroute on the Cirros instance is very basic. Also, the instance’s security group had to be changed to allow ICMP and a range of UDP ports for traceroute to work in this direction.

The MTU on the network and compute node physical Ethernet interfaces is 1500 – this is the maximum payload they can carry. The payload contains the following:

![Eth encaped in IPGRE](/images/2013-05-12-path-mtu-discovery-and-gre/Eth-encaped-in-IPGRE.png)

First the IPv4 header which includes the source and destination IPs of the physical nodes – its length is 20B. Then theGRE header follows which is 8B and includes the 4B GRE key option used for the Quantum segmentation_id. Then the Ethernet header of the original packet which is 18B (includes the 4B 802.1Q field). So the remaining space for the inner packet’s payload is capped at 1454 bytes (1500 – 20 – 8 – 18).

Now programs hang when connecting to some sites:

```
cirros$ curl -L github.com 


```
Note: connections to github.com:80 are redirected to port 443 and the ‘-L’ tells curl to follow redirects.

Running tcpdump on the network node’s interface for external access (the one in br-ex) shows lines like these repeating:

```
 ... 16:13:42.868082 IP 204.232.175.90.443 > 192.168.101.2.35016: Flags [.], seq 1:1449, ack 225, win 7, options [nop,nop,TS val 147466117 ecr 242857], length 1448 16:13:42.868542 IP 192.168.101.2 > 204.232.175.90: ICMP 192.168.101.2 unreachable - need to frag (mtu 1454), length 556 ... 

```
Here 192.168.101.2 is the SNATed address of the instance. Wireshark shows github.com sending an IP packet with total length 1500 (TCP segment data 1448 + TCP header in this case is 32 +  IPv4 header 20). This is too large to be encapsulated and so an ICMP (type 3, code 4) is sent back to github.com.

Github.com ignores these ICMPs and continues to retry sending the 1500 byte packet. I think in this case the ICMP received by github.com does not contain valid data. The address 192.168.101.2 is not a public IP and packets are SNATed again by a home router. The ICMP contains the non-public IP address in its data and I don’t think the home router is changing that. Indeed this setup cannot connect to sites that are supposed to be okay. But some sites are known to block valid ICMP for security reasons so a workaround is needed anyway.

##SOLUTIONS

One way is to reduce the MTU on the instances to 1454. This post shows how DHCP can be used to push this out to the instances.
 cirros$ sudo ip link set mtu 1454 dev eth0 
Another option is to increase the MTU to 1546 on the network and compute node interfaces that carry the GRE traffic. Then the PMTU should increase to 1500. This option depends on the physical equipment in-between being able to support a MTU greater than 1500.

Update: another solution - a patch that disables the pmtud option on the GRE ports.
UPDATE (OVS >= 1.9.0)
The way Open vSwitch handles MTU on tunnels has changed from version 1.9.0 and you will not see the behaviour described above. From 1.9.0 path MTU discovery is disabled by default. See here.
For the Quantum OVS plugin this means that packets that exceed the MTU on the physical interfaces will now get fragmented and ICMPs will not be returned to the sender. You may still want to configure the MTUs (on instances or nodes) to prevent this IP fragmentation, as it reduces overall performance due to the overhead of the many small additional packets it creates.