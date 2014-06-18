---
layout: post
title: Icehouse Release Notes简单翻译
description: Icehouse Release Notes
category: blog
---

声明：  
本博客欢迎转发，但请保留原作者信息!  
新浪微博：[@孔令贤HW](http://weibo.com/lingxiankong)；   
博客地址：<http://lingxiankong.github.io/>  
内容系本人学习、研究和总结，如有雷同，实属荣幸！

OpenStack Icehouse于4.17正式发布，release notes也在第一时间发布。但毕竟是开源项目，release notes在质量上无法与大公司的版本发布时相比较，至少没有我司看着规整。罢了，没有参与没有发言权，还是感谢来自全球的开发者的努力。

这里，我就我自己比较关心的几个模块做一个简单翻译，英文能力强的童鞋可以无视。

## Nova
目前支持在线升级（即先升级controller，然后依次升级计算节点），避免整个系统的宕机时间。

### API
* V3 API不再支持OS-DCF:diskConfig；
* 对于XML的支持目前已被废弃（depracated），在下个版本会被正式废除；
* 提供了ExtendedServicesDelete在扩展API，彻底在系统中删除计算结点；
* 在V3 API中，允许暴露部分而不是全部的管理员操作；
* 由于在Keystone中允许租户名称不唯一，所以Nova与Neutron交互时，采用租户标识而不是租户名称；
* nova hypervisor-show可以得到hypervisor的IP地址；

### 调度
* 为了提升调试的性能，nova-scheduler支持caching scheduler driver来缓存可用主机；
* 增加了新的AggregateImagePropertiesIsolation过滤器，根据镜像属性和AG属性过滤主机。增加了两个配置项：`aggregate_image_properties_isolation_namespace`和`aggregate_image_properties_isolation_separator`
* 在进行权重计算时，将乘数因子标准化，参考[这个](https://review.openstack.org/#/c/27160/)review
* 调度器支持虚拟机组的亲和性和反亲和性

### Driver增强
#### Libvirt
* 创建虚拟机时，libvirt支持动态修改内核参数。从Glance中镜像的metadata中获取`os_command_line`的键值
* 支持virtio-scsi，是虚拟机可直接访问块存储设备，比virtio-blk更先进，扩展性和性能更好
* 支持Virtio RNG设备（没太看懂这个这个设备是干啥的），貌似也是虚拟机访问物理主机上的某个设备，也是在镜像的metadata中定义，`hw_rng`；
* 视频驱动的支持，在镜像的metadata中配置，`hw_video_model`, `hw_video_vram`, 和`hw_video_head`
* 支持Watchdog设备（i6300esb），可以在镜像的metadata中配置，`hw_watchdog_action`，或者定义flavor的extra_specs。
* 不再使用High Precision Event Timer (HPET)，使用Windows镜像，在大负载下，会引起时钟偏移
* 在创建虚拟机时，等待Neutron的事件通知。

#### VMware
* 支持diagnostics API
* 支持ISO创建虚拟机
* 支持镜像缓存老化

#### XenServer
* 所有XenServer相关的配置项被统一划分到xenserver段中
* 增加对[PCI passthrough](https://blueprints.launchpad.net/nova/+spec/pci-passthrough-xenapi)的支持
* 引入XenServer CI
* 增强对临时卷的支持
* 其他一些性能和稳定性的增强

### 其它特性和变更
* 创建和删除keypairs时发送通知
* 主机操作发送通知
* Nova进程支持优雅退出，不再接受新的请求，但正在处理的请求不受影响
* 根据`running_deleted_instance_action`的配置，Nova允许定义如何处理那些已被上层删除，但底层仍在运行的虚拟机
* 默认不再支持文件注入，推荐使用ConfigDrive或metadata服务。如果要继续使用，需要配置`inject_key`和`inject_partition`。在未来版本，可能会彻底删除文件注入机制。
* Docker和PowerVM的driver被移除，后续Docker可能会与Heat关系更紧密
* `compute_api_class `配置项被删除
* 一些配置项被重命名，更易懂
* 对libguestfs的依赖为必选项

### 注意事项
虽然Nova支持调用其他组件新的API版本，但在Icehouse承诺交付的是Keystone v2，Cinder v1，Glance v1

## Neutron
Icehouse版本，Neutron主要致力于提高稳定性和代码质量。新增plugin： IBM SDN-VE, Nuage, OneConvergence, OpenDaylight，新的LB plugin：Embrane, NetScaler, Radware，新的VPN driver：Cisco CSR。Midokura的plugin不在Neutron的代码库维护。

OVS plugin和LinuxBridge plugin将被废弃，被ML2 plugin取代，官方提供了升级[脚本](http://git.openstack.org/cgit/openstack/neutron/tree/neutron/db/migration/migrate_to_ml2.py)，但该脚本没有回退功能，用之前请做足测试。

对XML的API支持被废弃

## Cinder
### 新增特性
* 修改卷类型
* 为Backup对象提供metadata支持
* API多进程化
* 删除配额
* 导入导出backups
* 挂卸卷时想Ceilometer发送通知
* 增加了几种driver

### 遗留问题
* 暂不支持Glance v2
* 继续支持Cinder v1，因为Nova暂不支持v2

## Heat
### 新增特性
* [HOT模板](http://docs.openstack.org/developer/heat/template_guide/hot_spec.html)被推荐使用
* 资源对象的支持
* 新增Software configuration API
* 去除创建stack时对管理员凭据的依赖
* 提供维护API
* 事件通知（比如stack状态的变更，伸缩的触发）
* heat-engine的多实例化
* 支持stack的abandon和adopt操作。abandon即删除stack，保留资源；adopt是根据已有资源创建stack。两个特性不交付。
* stack的预览（preview）功能
* 使用trusts作为延迟的用户鉴权

### 遗留问题
* stack-update操作时的异常会导致stack状态不可恢复
* CFN API对于任何的失败均返回500错误，Juno-1已修复
* 若stack中虚拟机有挂在用户卷，则删除stack可能需要删除多次，Juno-1已修复

## 文档
* Operations Guide提供了更新指导和网络排错指导
* 提供了命令行使用说明
* 用户手册提供了对python sdk的说明


----------
参考链接：  
<https://wiki.openstack.org/wiki/ReleaseNotes/Icehouse>