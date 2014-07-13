---
layout: post
title: Openstack VM live migration
category: blog
description: This post is based assumption that KVM as hypervisor, and Openstack is running in Grizzly on top of RHEL6.4.
---

Openstack VM live migration can have 3 categories:

* Block live migration without shared storage

* Shared storage based live migration

* Volume backed VM live migration

##Block live migration

Block live migration does not require shared storage among nova-compute nodes, it uses network(TCP) to copy VM disk from source host to destination host, thus it takes longer time to complete than shared storage based live migration. Also during migration, host performance will be degraded in network and CPU points of view.

To enable block live migration, we need to change libvirtd and nova.conf configurations on every nova-compute hosts:

->Edit /etc/libvirt/libvirtd.conf
```
listen_tls = 0
listen_tcp = 1
auth_tcp = “none”
```
->Edit /etc/sysconfig/libvirtd
```
LIBVIRTD_ARGS=”–listen”
```
->Restart libvirtd
```
service libvirtd restart
```
->Edit /etc/nova/nova.conf, add following line:
```
live_migration_flag=VIR_MIGRATE_UNDEFINE_SOURCE,VIR_MIGRATE_PEER2PEER,VIR_MIGRATE_LIVE
```

Adding this line is to ask nova-compute  to utilize libvirt’s true live migration function.

By default Migrations done by the nova-compute does not use libvirt’s live migration functionality, which means guests are suspended before migration and may therefore experience several minutes of downtime.

>Restart nova-compute service
```
service openstack-nova-compute restart
```
->Check running VMs
```
nova list
+————————————–+——————-+——–+———————+
| ID | Name | Status | Networks |
+————————————–+——————-+——–+———————+
| 5374577e-7417-4add-b23f-06de3b42c410 | vm-live-migration | ACTIVE | ncep-net=10.20.20.3 |
+————————————–+——————-+——–+———————+
```
->Check which host the VM in running on
```
nova show vm-live-migration
+————————————-+———————————————————-+
| Property | Value |
+————————————-+———————————————————-+
| status | ACTIVE |
| updated | 2013-08-06T08:47:13Z |
| OS-EXT-STS:task_state | None |
| OS-EXT-SRV-ATTR:host | compute-1 |
```
->Check all available hosts
```
nova host-list
+————–+————-+———-+
| host_name | service | zone |
+————–+————-+———-+
| compute-1 | compute | nova |
| compute-2 | compute | nova |
```
>Let’s migrate the vm to host compute-2
```
nova live-migration  –block-migrate vm-live-migration compute-2
```
>Later when we check vm status again, we cloud see the host changes to compute-2
```
nova show vm-live-migration
+————————————-+———————————————————-+
| Property | Value |
+————————————-+———————————————————-+
| status | ACTIVE |
| updated | 2013-08-06T09:45:33Z |
| OS-EXT-STS:task_state | None |
| OS-EXT-SRV-ATTR:host | compute-2 |
```
>We can just do a migration without specifying destination host, nova-scheduler will choose a free host for you
```
nova live-migration  –block-migrate vm-live-migration
```
Block live migration seems a good solution if we don’t have/want shared storage , but we still want to do some host level maintenance work without bring downtime to running VMs.

##Shared storage based live migration

This example we use GlusterFS as shared storage for nova-compute nodes. Since VM disk is stored on shared storage, hence live migration much much more faster than block live migration.

1.Setup GlusterFS cluster for nova-compute nodes

->Prepare 4 nodes, each node has an extra disk other than OS disk, dedicated for GlusterFS cluster
```
Node 1: 10.68.124.18 /dev/sdb 1TB
Node 2: 10.68.124.19 /dev/sdb 1TB
Node 3: 10.68.124.20 /dev/sdb 1TB
Node 4: 10.68.124.21 /dev/sdb 1TB
```
->Configure Node 1-4
```
yum install -y xfsprogs.x86_64
mkfs.xfs -f -i size=512 /dev/sdb
mkdir /brick1
echo “/dev/sdb /brick1 xfs defaults 1 2″ >> /etc/fstab 1014 cat /etc/fstab
mount -a && mount
mkdir /brick1/sdb
yum install glusterfs-fuse glusterfs-server
/etc/init.d/glusterd start
chkconfig glusterd on
```
->On Node-1, add peers
```
gluster peer probe 10.68.125.19
gluster peer probe 10.68.125.20
gluster peer probe 10.68.125.21
```
->On node-1, create a volume for nova-compute
```
gluster volume create nova-gluster-vol replica 2 10.68.125.18:/brick1/sdb 10.68.125.19:/brick1/sdb 10.68.125.20:/brick1/sdb 10.68.125.21:/brick1/sdb
gluster volume start nova-gluster-vol
```
->We can check the volume information

gluster volume info
Volume Name: nova-gluster-vol
Type: Distributed-Replicate
Status: Started
Number of Bricks: 2 x 2 = 4
Transport-type: tcp
Bricks:
	Brick1: 10.68.125.18:/brick1/sdb
	Brick2: 10.68.125.19:/brick1/sdb
	Brick3: 10.68.125.20:/brick1/sdb
	Brick4: 10.68.125.21:/brick1/sdb

->On each nova-compute node, mount the GluserFS volume to /var/lib/nova/instances
```
yum install glusterfs-fuse
mount.glusterfs 10.68.125.18:/nova-gluster-vol /var/lib/nova/instances
echo “10.68.125.18:/nova-gluster-vol /var/lib/nova/instances glusterfs defaults 1 2″ >> /etc/fstab
chown -R nova:nova  /var/lib/nova/instances
```
2.Configure libvirt and nova.conf on every nova-compute node

->Edit /etc/libvirt/libvirtd.conf
```
listen_tls = 0
listen_tcp = 1
auth_tcp = “none”
```
->Edit /etc/sysconfig/libvirtd
```
LIBVIRTD_ARGS=”–listen”
```
->Restart libvirtd
```
service libvirtd restart
```
->Edit /etc/nova/nova.conf, add following line:
```
live_migration_flag=VIR_MIGRATE_UNDEFINE_SOURCE,VIR_MIGRATE_PEER2PEER,VIR_MIGRATE_LIVE
```
3.Let’s try live migration

->Check running VMs
```
nova list
+————————————–+——————-+——–+———————+
| ID | Name | Status | Networks |
+————————————–+——————-+——–+———————+
| 5374577e-7417-4add-b23f-06de3b42c410 | vm-live-migration | ACTIVE | ncep-net=10.20.20.3 |
+————————————–+——————-+——–+———————+
```
->Check which host the VM in running on
```
nova show vm-live-migration
+————————————-+———————————————————-+
| Property | Value |
+————————————-+———————————————————-+
| status | ACTIVE |
| updated | 2013-08-06T08:47:13Z |
| OS-EXT-STS:task_state | None |
| OS-EXT-SRV-ATTR:host | compute-1 |
```
->Check all available hosts
```
nova host-list
+————–+————-+———-+
| host_name | service | zone |
+————–+————-+———-+
| compute-1 | compute | nova |
| compute-2 | compute | nova |
```
>Let’s migrate the vm to host compute-2
```
nova live-migration  vm-live-migration compute-2
```
>Migration should be finished in seconds, let’s check vm status again, we cloud see the host changes to compute-2
```
nova show vm-live-migration
+————————————-+———————————————————-+
| Property | Value |
+————————————-+———————————————————-+
| status | ACTIVE |
| updated | 2013-08-06T09:45:33Z |
| OS-EXT-STS:task_state | None |
| OS-EXT-SRV-ATTR:host | compute-2 |
```
>We can just do a migration without specifying destination host, nova-scheduler will choose a free host for us
```
nova live-migration vm-live-migration
```
3.Volume backed VM live migration

VMs booted from volume can be also easily and quickly migrated just like shared storage based live migration since both cases VM disks are on top of shared storages.

->Create a bootable volume from an existing image
```
cinder create  –image-id <id of registered image in glance>  –display-name bootable-vol-rhel6.3 5
```
->Launch a VM from the bootable volume
```
nova boot –image <id of any image in glance, not used but needed>  –flavor m1.medium –block_device_mapping vda=<id of bootable vol created above>:::0 vm-boot-from-vol
```
->Check which host the VM is running on
```
nova show vm-boot-from-vol
+————————————-+———————————————————-+
| Property | Value |
+————————————-+———————————————————-+
| status | ACTIVE |
| updated | 2013-08-06T10:29:09Z |
| OS-EXT-STS:task_state | None |
| OS-EXT-SRV-ATTR:host | compute-1 |
```
->Do live migration
```
nova live-migration vm-boot-from-vol
```
->Check VM status again
```
nova show vm-boot-from-vol
+————————————-+———————————————————-+
| Property | Value |
+————————————-+———————————————————-+
| status | ACTIVE |
| updated | 2013-08-06T10:30:55Z |
| OS-EXT-STS:task_state | None |
| OS-EXT-SRV-ATTR:host | compute-2 |
```
VM recovery from failed compute node

If shared storage used, or for VMs booted from volume, we can recovery them if their compute host is failed.

>Launch 1 normal VM and 1 volume backed VM, make they are all running on compute1 node. ( Use live migration if  they are not on compute-1)
```
nova show vm-live-migration
+————————————-+———————————————————-+
| Property | Value |
+————————————-+———————————————————-+
| status | ACTIVE |
| updated | 2013-08-06T10:36:35Z |
| OS-EXT-STS:task_state | None |
| OS-EXT-SRV-ATTR:host | compute-1 |

nova show vm-boot-from-vol
+————————————-+———————————————————-+
| Property | Value |
+————————————-+———————————————————-+
| status | ACTIVE |
| updated | 2013-08-06T10:31:21Z |
| OS-EXT-STS:task_state | None |
| OS-EXT-SRV-ATTR:host | compute-1 |
```
->Shutdown compute-1 node
```
[root@compute-1 ~]# shutdown now
```
->Check VM status
```
nova list
+————————————–+——————-+———+———————+
| ID | Name | Status | Networks |
+————————————–+——————-+———+———————+
| 75ddb4f6-9831-4d90-b291-48e098e8f72f | vm-boot-from-vol | SHUTOFF | ncep-net=10.20.20.5 |
| 5374577e-7417-4add-b23f-06de3b42c410 | vm-live-migration | SHUTOFF | ncep-net=10.20.20.3 |
+————————————–+——————-+———+———————+
```
Both VMs get into “SHUTOFF” status.

>Recovery them to compute-2 node
```
nova evacuate vm-boot-from-vol compute-2 –on-shared-storage
nova evacuate vm-live-migration compute-2 –on-shared-storage
```
->Check VM status
```
nova show vm-boot-from-vol
+————————————-+———————————————————-+
| Property | Value |
+————————————-+———————————————————-+
| status | SHUTOFF |
| updated | 2013-08-06T10:42:32Z |
| OS-EXT-STS:task_state | None |
| OS-EXT-SRV-ATTR:host | compute-2 |

nova show vm-live-migration
+————————————-+———————————————————-+
| Property | Value |
+————————————-+———————————————————-+
| status | SHUTOFF |
| updated | 2013-08-06T10:42:51Z |
| OS-EXT-STS:task_state | None |
| OS-EXT-SRV-ATTR:host | compute-2 |
```
Both VMs are managed by compute-2 now

-> Reboot 2 VMs
```
nova reboot vm-boot-from-vol
nova reboot vm-live-migration
```
-> Check VMs list
```
nova list
+————————————–+——————-+——–+———————+
| ID | Name | Status | Networks |
+————————————–+——————-+——–+———————+
| 75ddb4f6-9831-4d90-b291-48e098e8f72f | vm-boot-from-vol | ACTIVE | ncep-net=10.20.20.5 |
| 5374577e-7417-4add-b23f-06de3b42c410 | vm-live-migration | ACTIVE | ncep-net=10.20.20.3 |
+————————————–+——————-+——–+———————+
```
Both VMs are back to ACTIVE.

Notes

Currently Openstack has no mechanism to detect host and VM failure and automatically to recover/migrate VM to health host, this reply on 3rd party means to do so

There’s also an interesting project named openstack-neat, which is intended to provide an extension to OpenStack implementing dynamic consolidation of Virtual Machines (VMs) using live migration. The major objective of dynamic VM consolidation is to improve the utilization of physical resources and reduce energy consumption by re-allocating VMs using live migration according to their real-time resource demand and switching idle hosts to the sleep mode. This really like vSphere DRS function.