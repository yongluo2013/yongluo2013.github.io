---
layout: post
title: Heat中的AutoScaling(2)
description: Heat中的AutoScaling(2)
category: blog
---

声明：  
本博客欢迎转发，但请保留原作者信息!  
新浪微博：[@孔令贤HW](http://weibo.com/lingxiankong)；   
博客地址：<http://lingxiankong.github.io/>  
内容系本人学习、研究和总结，如有雷同，实属荣幸！

---

之前的一篇[博客](http://lingxiankong.github.io/blog/2014/04/03/heat-autoscaling-1/)讲了AWS和Heat中AutoScaling的机制和大概的实现，本篇是上一篇的姊妹篇，主要讲autoscaling在heat中的简单使用。

## 环境准备
我们还是以一个模板为例，模板内容如下：

    {
      "HeatTemplateFormatVersion": "2012-12-12",
      "Description": "Template which create a base autoscaling for launch base.",
      "Parameters": {
        "InstanceType": {
          "Type": "String"
        },
        "ImageId": {
          "Type": "String"
        },
        "VPCZoneIdentifier": {
          "Type": "CommaDelimitedList"
        },
        "MinSize": {
          "Type": "String"
        },
        "MaxSize": {
          "Type": "String"
        },
        "DesiredCapacity": {
          "Type": "Number"
        },
        "Cooldown": {
          "Type": "String",
          "Default": "0"
        }
      },
      "Resources": {
        "WebServerGroup": {
          "Type": "AWS::AutoScaling::AutoScalingGroup",
          "Properties": {
            "AvailabilityZones": {
              "Fn::GetAZs": ""
            },
            "VPCZoneIdentifier": {
              "Ref": "VPCZoneIdentifier"
            },
            "LaunchConfigurationName": {
              "Ref": "LaunchConfig"
            },
            "MinSize": {
              "Ref": "MinSize"
            },
            "MaxSize": {
              "Ref": "MaxSize"
            },
            "DesiredCapacity": {
              "Ref": "DesiredCapacity"
            },
            "Cooldown": {
              "Ref": "Cooldown"
            }
          }
        },
        "WebServerScaleUpPolicy": {
          "Type": "AWS::AutoScaling::ScalingPolicy",
          "Properties": {
            "AdjustmentType": "ChangeInCapacity",
            "AutoScalingGroupName": {
              "Ref": "WebServerGroup"
            },
            "ScalingAdjustment": "1"
          }
        },
        "CPUAlarmHigh": {
          "Type": "OS::Ceilometer::Alarm",
          "Properties": {
            "meter_name": "instance",
            "statistic": "avg",
            "period": "60",
            "evaluation_periods": "1",
            "threshold": "1",
            "repeat_actions": true,
            "matching_metadata": {
              "metadata.user_metadata.groupname": {
                "Ref": "WebServerGroup"
              }
            },
            "alarm_actions": [
              {
                "Fn::GetAtt": [
                  "WebServerScaleUpPolicy",
                  "AlarmUrl"
                ]
              }
            ],
            "comparison_operator": "eq"
          }
        },
        "LaunchConfig": {
          "Type": "AWS::AutoScaling::LaunchConfiguration",
          "Properties": {
            "ImageId": {
              "Ref": "ImageId"
            },
            "InstanceType": {
              "Ref": "InstanceType"
            }
          }
        }
      }
    }

大致的意思我就不多说了，可以参考AWS的文档。只需要注意那个告警的定义，意思是，如果每60s检测内系统中的instance资源的统计平均值等于1，就触发告警。

> 这个条件总是成立，这里只是为了试验，这个告警在实际使用中没有任何意义

另外，我环境上的一些资源信息如下，创建stack时需要在参数中指定，注意环境中已有3个虚拟机：

    UVP:~ # nova list
    +--------------------------------------+--------------------------------------------+--------+------------+-------------+--------------------+
    | ID                                   | Name                                       | Status | Task State | Power State | Networks           |
    +--------------------------------------+--------------------------------------------+--------+------------+-------------+--------------------+
    | 735f0e66-19f9-40b1-9c13-4588051d7270 | cirros_vm                                  | ACTIVE | -          | Running     | demo_net1=10.1.1.4 |
    | 3d2a4042-7859-4d6d-8fdf-719006faffe8 | waitcondition_test-MyServer-uscavjnqqbna   | ACTIVE | -          | Running     | demo_net1=10.1.1.5 |
    | 57702f03-859e-4d4d-81b5-d6d61dfd1a24 | waitcondition_test_1-MyServer-e5q5bf4g4qrz | ACTIVE | -          | Running     | demo_net1=10.1.1.6 |
    +--------------------------------------+--------------------------------------------+--------+------------+-------------+--------------------+
    UVP:/home/kong # nova flavor-list
    +----+-----------+-----------+------+-----------+------+-------+-------------+-----------+
    | ID | Name      | Memory_MB | Disk | Ephemeral | Swap | VCPUs | RXTX_Factor | Is_Public |
    +----+-----------+-----------+------+-----------+------+-------+-------------+-----------+
    | 1  | m1.tiny   | 512       | 1    | 0         |      | 1     | 1.0         | True      |
    | 2  | m1.small  | 2048      | 20   | 0         |      | 1     | 1.0         | True      |
    | 3  | m1.medium | 4096      | 40   | 0         |      | 2     | 1.0         | True      |
    | 4  | m1.large  | 8192      | 80   | 0         |      | 4     | 1.0         | True      |
    | 5  | m1.xlarge | 16384     | 160  | 0         |      | 8     | 1.0         | True      |
    +----+-----------+-----------+------+-----------+------+-------+-------------+-----------+
    nUVP:/home/kong # glance image-list
    +--------------------------------------+----------------------------+-------------+------------------+-----------+--------+
    | ID                                   | Name                       | Disk Format | Container Format | Size      | Status |
    +--------------------------------------+----------------------------+-------------+------------------+-----------+--------+
    | 414bcb4d-8e68-4dfe-8d21-31e9bb3d26fd | cirros                     | qcow2       | bare             | 9159168   | active |
    | 72fb9278-0957-4f3e-bc9d-9a842c6f3788 | F17-x86_64-cfntools        | qcow2       | ovf              | 476704768 | active |
    | 252a1c83-7b7d-4776-a586-667d0b7311f4 | Ubuntu 12.04 cloudimg i386 | qcow2       | ovf              | 213123072 | active |
    +--------------------------------------+----------------------------+-------------+------------------+-----------+--------+
    UVP:/home/kong # neutron subnet-list
    +--------------------------------------+------------------+----------------+--------------------------------------------------------+
    | id                                   | name             | cidr           | allocation_pools                                       |
    +--------------------------------------+------------------+----------------+--------------------------------------------------------+
    | 5e011976-d15b-45e3-8a63-62f98f8d4a42 | external_subnet1 | 200.200.0.0/16 | {"start": "200.200.200.100", "end": "200.200.200.120"} |
    | a03be99b-b78f-46d0-9311-e45fd8bcf92d | demo_subnet1     | 10.1.1.0/24    | {"start": "10.1.1.2", "end": "10.1.1.254"}             |
    +--------------------------------------+------------------+----------------+--------------------------------------------------------+

## 创建stack

    UVP:/home/kong # heat stack-create autoscaling_test -f autoscaling.template -P "InstanceType=m1.tiny;ImageId=414bcb4d-8e68-4dfe-8d21-31e9bb3d26fd;VPCZoneIdentifier=a03be99b-b78f-46d0-9311-e45fd8bcf92d;MinSize=1;MaxSize=2;DesiredCapacity=1"
    +--------------------------------------+----------------------+--------------------+----------------------+
    | id                                   | stack_name           | stack_status       | creation_time        |
    +--------------------------------------+----------------------+--------------------+----------------------+
    | b4fcc73b-83f6-42fc-8847-75c5dd829251 | waitcondition_test   | CREATE_COMPLETE    | 2014-05-13T15:07:27Z |
    | f51e784d-0fbf-40ba-a580-423a2419e819 | waitcondition_test_1 | CREATE_COMPLETE    | 2014-05-13T15:49:29Z |
    | 0251d96f-af3a-4823-9aca-1c5fdea6475d | autoscaling_test     | CREATE_IN_PROGRESS | 2014-05-20T13:50:17Z |
    +--------------------------------------+----------------------+--------------------+----------------------+
    UVP:/home/kong # heat stack-list
    +--------------------------------------+----------------------+-----------------+----------------------+
    | id                                   | stack_name           | stack_status    | creation_time        |
    +--------------------------------------+----------------------+-----------------+----------------------+
    | b4fcc73b-83f6-42fc-8847-75c5dd829251 | waitcondition_test   | CREATE_COMPLETE | 2014-05-13T15:07:27Z |
    | f51e784d-0fbf-40ba-a580-423a2419e819 | waitcondition_test_1 | CREATE_COMPLETE | 2014-05-13T15:49:29Z |
    | 0251d96f-af3a-4823-9aca-1c5fdea6475d | autoscaling_test     | CREATE_COMPLETE | 2014-05-20T13:50:17Z |
    +--------------------------------------+----------------------+-----------------+----------------------+
    
查询虚拟机，看到环境中新增了一个虚拟机，该虚拟机属于stack中的autoscaling组：    
    
    UVP:/home/kong # nova list
    +--------------------------------------+-------------------------------------------------------+--------+------------+-------------+--------------------+
    | ID                                   | Name                                                  | Status | Task State | Power State | Networks           |
    +--------------------------------------+-------------------------------------------------------+--------+------------+-------------+--------------------+
    | 37f6bcd2-6964-49f0-97f9-f5c3a7e3f979 | au-ServerGroup-f35skwv4ty7k-slap352x6bpb-fv5dnk46nqim | ACTIVE | -          | Running     | demo_net1=10.1.1.8 |
    | 735f0e66-19f9-40b1-9c13-4588051d7270 | cirros_vm                                             | ACTIVE | -          | Running     | demo_net1=10.1.1.4 |
    | 3d2a4042-7859-4d6d-8fdf-719006faffe8 | waitcondition_test-MyServer-uscavjnqqbna              | ACTIVE | -          | Running     | demo_net1=10.1.1.5 |
    | 57702f03-859e-4d4d-81b5-d6d61dfd1a24 | waitcondition_test_1-MyServer-e5q5bf4g4qrz            | ACTIVE | -          | Running     | demo_net1=10.1.1.6 |
    +--------------------------------------+-------------------------------------------------------+--------+------------+-------------+--------------------+
    
大概过上60秒，通过Ceilometer查询statistic，发现满足告警条件，会自动增加一个虚拟机：    
    
    UVP:/home/kong # nova list
    +--------------------------------------+-------------------------------------------------------+--------+------------+-------------+--------------------+
    | ID                                   | Name                                                  | Status | Task State | Power State | Networks           |
    +--------------------------------------+-------------------------------------------------------+--------+------------+-------------+--------------------+
    | 6245912b-1711-4efa-8baf-f0ef2adebaed | au-ServerGroup-f35skwv4ty7k-65khnyblzmr5-rtnas4aj4pah | ACTIVE | -          | Running     | demo_net1=10.1.1.9 |
    | 37f6bcd2-6964-49f0-97f9-f5c3a7e3f979 | au-ServerGroup-f35skwv4ty7k-slap352x6bpb-fv5dnk46nqim | ACTIVE | -          | Running     | demo_net1=10.1.1.8 |
    | 735f0e66-19f9-40b1-9c13-4588051d7270 | cirros_vm                                             | ACTIVE | -          | Running     | demo_net1=10.1.1.4 |
    | 3d2a4042-7859-4d6d-8fdf-719006faffe8 | waitcondition_test-MyServer-uscavjnqqbna              | ACTIVE | -          | Running     | demo_net1=10.1.1.5 |
    | 57702f03-859e-4d4d-81b5-d6d61dfd1a24 | waitcondition_test_1-MyServer-e5q5bf4g4qrz            | ACTIVE | -          | Running     | demo_net1=10.1.1.6 |
    +--------------------------------------+-------------------------------------------------------+--------+------------+-------------+--------------------+
    
## 其他观察点
### Ceilometer
通过Ceilometer看一下告警的信息：

    UVP:/home/kong # ceilometer alarm-show -a 4a394c06-0653-43be-8be6-9ac33d2515c5
    +---------------------------+--------------------------------------------------------------------------+
    | Property                  | Value                                                                    |
    +---------------------------+--------------------------------------------------------------------------+
    | alarm_actions             | [u'http://172.25.150.8:8000/v1/signal/arn%3Aopenstack%3Aheat%3A%3A57314a |
    |                           | 8980b34c6ba3ece5cd19ff6214%3Astacks%2Fautoscaling_test%2F0251d96f-       |
    |                           | af3a-4823-9aca-1c5fdea6475d%2Fresources%2FWebServerScaleUpPolicy?Timesta |
    |                           | mp=2014-05-20T13%3A50%3A26Z&SignatureMethod=HmacSHA256&AWSAccessKeyId=48 |
    |                           | c2804055b04be988f3b5b9adeac486&SignatureVersion=2&Signature=6nWhUK8FTO63 |
    |                           | qDLRp0xkWx68lQGlf%2BvZgB44Dm%2BujP8%3D']                                 |
    | alarm_id                  | 4a394c06-0653-43be-8be6-9ac33d2515c5                                     |
    | comparison_operator       | eq                                                                       |
    | description               | Alarm when instance is eq a avg of 1.0 over 60 seconds                   |
    | enabled                   | True                                                                     |
    | evaluation_periods        | 1                                                                        |
    | exclude_outliers          | False                                                                    |
    | insufficient_data_actions | []                                                                       |
    | meter_name                | instance                                                                 |
    | name                      | autoscaling_test-CPUAlarmHigh-wrly4nxupktb                               |
    | ok_actions                | []                                                                       |
    | period                    | 60                                                                       |
    | project_id                | 57314a8980b34c6ba3ece5cd19ff6214                                         |
    | query                     | metadata.user_metadata.groupname == autoscaling_test-WebServerGroup-     |
    |                           | f35skwv4ty7k                                                             |
    | repeat_actions            | True                                                                     |
    | state                     | alarm                                                                    |
    | statistic                 | avg                                                                      |
    | threshold                 | 1.0                                                                      |
    | type                      | threshold                                                                |
    | user_id                   | 6114fcc32cd342859d1462d932144faa                                         |
    +---------------------------+--------------------------------------------------------------------------+
    
注意`alarm_actions`字段，其实就是一个URL，还记得[Heat服务简介](http://lingxiankong.github.io/blog/2014/05/07/heat-service/)么？错过的童鞋请恶补一下吧。 

### Nova
看一下通过Heat中autoscaling创建的虚拟机有什么不同之处：

    UVP:/home/kong # nova show 37f6bcd2-6964-49f0-97f9-f5c3a7e3f979
    +--------------------------------------+------------------------------------------------------------------------------------------------------------------------------------------------+
    | Property                             | Value                                                                                                                                          |
    +--------------------------------------+------------------------------------------------------------------------------------------------------------------------------------------------+
    | OS-DCF:diskConfig                    | MANUAL                                                                                                                                         |
    | OS-EXT-AZ:availability_zone          | nova                                                                                                                                           |
    | OS-EXT-SRV-ATTR:host                 | UVP                                                                                                                                            |
    | OS-EXT-SRV-ATTR:hypervisor_hostname  | UVP                                                                                                                                            |
    | OS-EXT-SRV-ATTR:instance_name        | instance-00000006                                                                                                                              |
    | OS-EXT-STS:power_state               | 1                                                                                                                                              |
    | OS-EXT-STS:task_state                | -                                                                                                                                              |
    | OS-EXT-STS:vm_state                  | active                                                                                                                                         |
    | OS-SRV-USG:launched_at               | 2014-05-20T13:50:25.000000                                                                                                                     |
    | OS-SRV-USG:terminated_at             | -                                                                                                                                              |
    | accessIPv4                           |                                                                                                                                                |
    | accessIPv6                           |                                                                                                                                                |
    | config_drive                         |                                                                                                                                                |
    | created                              | 2014-05-20T13:50:19Z                                                                                                                           |
    | demo_net1 network                    | 10.1.1.8                                                                                                                                       |
    | flavor                               | m1.tiny (1)                                                                                                                                    |
    | hostId                               | c058bf4191ba5b03d9410d850e050be5ab884df3fd0d804801d39d6f                                                                                       |
    | id                                   | 37f6bcd2-6964-49f0-97f9-f5c3a7e3f979                                                                                                           |
    | image                                | cirros (414bcb4d-8e68-4dfe-8d21-31e9bb3d26fd)                                                                                                  |
    | key_name                             | -                                                                                                                                              |
    | metadata                             | {"metering.groupname": "autoscaling_test-WebServerGroup-f35skwv4ty7k", "AutoScalingGroupName": "autoscaling_test-WebServerGroup-f35skwv4ty7k"} |
    | name                                 | au-ServerGroup-f35skwv4ty7k-slap352x6bpb-fv5dnk46nqim                                                                                          |
    | os-extended-volumes:volumes_attached | []                                                                                                                                             |
    | progress                             | 0                                                                                                                                              |
    | security_groups                      | default                                                                                                                                        |
    | status                               | ACTIVE                                                                                                                                         |
    | tenant_id                            | 57314a8980b34c6ba3ece5cd19ff6214                                                                                                               |
    | updated                              | 2014-05-20T13:50:25Z                                                                                                                           |
    | user_id                              | 6114fcc32cd342859d1462d932144faa                                                                                                               |
    +--------------------------------------+------------------------------------------------------------------------------------------------------------------------------------------------+
    
请注意其中的`metadata`字段的内容。Ceilometer会根据这个字段匹配监控的虚拟机指标。    