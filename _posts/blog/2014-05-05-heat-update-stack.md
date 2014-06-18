---
layout: post
title: Heat中更新stack
description: Heat中更新stack
category: blog
---

声明：  
本博客欢迎转发，但请保留原作者信息!  
新浪微博：[@孔令贤HW](http://weibo.com/lingxiankong)；   
博客地址：<http://lingxiankong.github.io/>  
内容系本人学习、研究和总结，如有雷同，实属荣幸！

Heat中的更新stack行为，基本就是参照CloudFormation设计和实现的。所以，这里的主要参考来源就是CloudFormation的官方文档，另外就是Heat Icehouse版本的实现代码。

## 一个示例
首先我们需要一个初始模板，<https://s3.amazonaws.com/cloudformation-templates-us-east-1/UpdateTutorial+Part1.template>，该模板展示的是一个php应用，Apache Web Server、PHP 和简单的 PHP 应用程序全部都由默认安装在 Amazon Linux AMI 上的 AWS CloudFormation 帮助程序脚本进行安装。

模板启用和配置 cfn-hup 后台程序以侦听 Amazon EC2 实例元数据中所定义配置的更改。可以通过使用 cfn-hup 后台程序来更新应用程序软件（如 Apache 或 PHP 的版本），也可以通过 AWS CloudFormation 更新 PHP 应用程序文件本身。来自于模板中同一个 EC2 资源的以下代码段显示的是，在侦测到元数据的任何更改时配置 cfn-hup 调用 cfn-init 来更新软件所需的部件：

    "# Start up the cfn-hup daemon to listen for changes to the Web Server metadata\n",
    "/opt/aws/bin/cfn-hup || error_exit 'Failed to start cfn-hup'\n",

默认状态下，cfn-hup 后台程序每 15 分钟运行一次，因此最多要等待 15 分钟在堆栈更新后更改应用程序。

> 注意：如果您通过 AWS Management Console
> 更新堆栈，您将会注意到创建初始堆栈所用参数已经预填充到更新堆栈向导的参数页上。如果您使用命令行或API接口，请务必提供原来创建 stack 时的参数值。

## 更新简介
利用CloudFormation的更新stack的功能，能够更改或添加资源或资源属性（包括虚拟机的镜像）、增加或更新应用程序的软件配置。

### 更新虚拟机
根据Heat中的配置，虚拟机中可以更新的字段有：  
名称、镜像、规格、规格更新策略、镜像更新策略、网卡、元数据（metadata）、管理员密码(I版暂没有实现)

> 对虚拟机的更新中，没有更新管理员密码。目前(20140519)，Nova中只有Xen实现了修改密码的功能。在创建虚拟机时，密码注入是通过修改虚拟机文件系统实现的。

#### 更新镜像
更新 Amazon EC2 实例上的 AMI时，不能只通过启动和停止实例来修改 AMI；AWS CloudFormation 将其视为对资源不可变属性的更改。为了对不可变属性进行更改（再比如为虚拟机添加密钥对），AWS CloudFormation 必须启动替换资源，即：运行新 AMI 的新 Amazon EC2 实例。在新实例运行之后，AWS CloudFormation 会更新堆栈中的弹性 IP 地址等其他资源，以指向新资源。所有新资源被创建且旧资源被删除的过程被称为 UPDATE_CLEANUP。此时，堆栈中的实例的 ID 已随着更新更改。“Event”表中的事件包含描述“所请求的更新包含对不可变属性的更改，因此创建新的物理资源”，以指示资源已被替代。

在Heat中，与CloudFormation一样，镜像属于不可变属性。可以在模板中配置虚拟机的`image_update_policy`属性，允许的值是'REBUILD', 'REPLACE', 'REBUILD-PRESERVE-EPHEMERAL'，默认值是REPLACE。那么默认的行为会抛出一个内部异常，UpdateReplace，意即：创建一个新的虚拟机。如果是REBUILD，那么就对虚拟机执行rebuild操作（REBUILD-PRESERVE-EPHEMERAL就是在rebuild的时候多了一个参数而已）

#### 更新网卡
Heat中，更新网卡比较简单，就是把旧的网卡卸载（detach），将新的网卡添加（attach），如果更新时，没有填任何网卡信息，则卸载原有网卡后，会任意添加一个网卡。

### 更新autoscaling组  
OpenStack中没有针对autoscaling的资源操作，仅仅是在Heat中支持该资源类型，采用嵌套stack的方式实现。目前也没有实现虚拟机的健康检查功能。

更新 Auto Scaling 组的 Amazon EC2 启动配置，不会影响 Auto Scaling 组中任何正在运行的 EC2 实例。更新后的启动配置只适用于更新之后创建的新实例。

如果更新 stack 中包含的 AutoScaling 资源，在 Auto Scaling 组中的所有 EC2 实例上都不会提供任何同步或序列化行为。每个主机上的 cfn-hup 后台程序都将独立运行且会按其自己的计划更新应用程序。当您使用 cfn-hup 更新实例上的配置时，每个实例都将按其自己的计划运行 cfn-hup 程序；堆栈中的实例之间不协调。在使用时要考虑如下因素：

* 如果 Auto Scaling 组中所有 EC2 实例的 cfn-hup 更改都同时运行，更新期间您的服务可能不可用。
* 如果 cfn-hup 更改在不同时间运行，则新旧版软件可能会同时运行。

Heat中，autoscaling资源允许更新UpdatePolicy，autoscaling资源需要引用一个LaunchConfiguration资源，其属性有：  
'ImageId'：  镜像ID，必选  
'InstanceType'：虚拟机类型，必选  
'KeyName'：密钥名称，可选  
'UserData'：用户自定义数据，可选  
'SecurityGroups'：安全组们，可选  
'KernelId', 'RamDiskId'：兼容AWS的磁盘标识，目前不支持  
'BlockDeviceMappings'：创建虚拟机时的用户盘信息，目前不支持  
'NovaSchedulerHints'：调度信息  

autoscaling的资源属性有：  
'AvailabilityZones'：你懂的  
'LaunchConfigurationName'： 你懂的，必选，而且允许更新  
'MaxSize'：最大实例个数，必选，允许更新  
'MinSize'：最小实例个数，必选，允许更新  
'Cooldown'：冷却时间，允许更新  
'DesiredCapacity'：默认实例个数，允许更新  
'HealthCheckGracePeriod'：健康检查的时间间隔，未实现      
'HealthCheckType'：健康检查类型，未实现  
'LoadBalancerNames'：LB名称  
'VPCZoneIdentifier'：VPC信息  
'Tags'：标识，Heat与Ceilometer的交互，就仰仗这个Tags信息  

Heat允许更新LaunchConfiguration，需要在模板中定义UpdatePolicy属性中的AutoScalingRollingUpdate，是一个map。autoscaling资源中的metadata中保存了上次调整实例个数的时间（以确定冷却时间）。同时，实例个数发生变化的开始和结束时，会有事件通知。

### 可用性和影响
不同的属性会对堆栈中的资源造成不同的影响。您可以使用 AWS CloudFormation 更新任何属性；但是您应该在进行任何更改之前考虑以下问题：

* 更新会如何影响资源本身？例如，更新警报阈值会使警报在更新期间处于非活动状态。再比如，更改实例类型时需要停止和重启实例。
* 更改可变属性还是不可变属性？对资源属性的某些更改，如更改 Amazon EC2 实例上的 AMI，不受基础服务的支持。如果更改可变属性，AWS CloudFormation 将使用适用于基础资源的“更新”或“修改”类型 API。对于不可变的属性更改，AWS CloudFormation 将用更新后的属性创建新资源，然后再删除旧资源之前将此资源链接至stack。虽然 AWS CloudFormation 尝试减少堆栈资源的停机时间，但替代资源是一个多步骤过程，需要时间。重新配置堆栈期间，您的应用程序不能全面运行。例如，它可能不能为请求提供服务或访问数据库。

根据修改堆栈中的更新资源所采用的方法，您可以明智地决定修改资源的最佳时间，以降低此类更改对您的应用程序产生的影响。具体来说，您应该仔细计划好更新过程中必须替换资源的时间。例如，如果您更新了 AWS::RDS::DBInstance 资源的 Port 属性，AWS CloudFormation 将使用更改后的端口设置和新的物理名称创建一个新的数据库实例。为作计划，您应该为当前数据库创建快照、针对数据库实例替换时，使用该数据库实例的应用程序如何处理中断制定一个策略，确保使用该数据库实例的应用程序将新端口设置以及您进行的任何其他更改考虑在内，然后使用数据库快照在新数据库实例上恢复数据库。此示例并不详尽：它旨在让您了解当资源的更新方法为替换时，需要计划的事项。