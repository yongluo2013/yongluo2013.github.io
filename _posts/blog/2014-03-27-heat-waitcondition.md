---
layout: post
title: 从Heat中的WaitCondition说起
description: 从Heat中的WaitCondition说起
category: blog
---

声明：  
本博客欢迎转发，但请保留原作者信息!  
新浪微博：[@孔令贤HW](http://weibo.com/lingxiankong)；   
博客地址：<http://lingxiankong.github.io/>  
内容系本人学习、研究和总结，如有雷同，实属荣幸！

*写在前面*  
对一个东西不懂的时候，在网上到处搜教程，搜资料，如果资料少了，还会谩骂，会沮丧；等到好不容易弄懂，到自己总结的时候，却又嫌麻烦，不愿意从基本概念开始提起，觉得太简单，不值一写。 

人总是这样，想获取帮助，却又不想提供帮助。当然了，写博客也不全是为了帮助别人，更是为了帮助自己。记得曾经读过一段话，具体怎么说忘记了，但大致意思是，自认为掌握的知识，不一定能准确无误的给别人讲出来，更不用说有体系、有章法的讲出来。我在工作中也发现很多员工，看了几天代码，处理了几个问题，就觉得自己已经炉火纯青了，但当我让他从头给我讲的时候，又没有任何头绪。所以，适时的总结和回顾，也是升华自我、构筑知识体系的过程。

好了，废话不说了，今天总结一下OpenStack中业务编排服务Heat中的WaitCondition以及相关的知识（与其说是Heat，还不如说是CloudFormation）。

## 基本概念
既然说到WaitCondition，就先提一下`condition`。虽然Heat类比于AWS的CloudFormation，但目前并不支持condition（但不排除以后会支持），为了扫盲，在这里一并把这个概念提一下。可以把condition看成一个判断结果，创建stack时可以根据某个condition的结果决定是否执行某个动作。说的抽象了，看个例子吧：

    "Parameters": {
      "EnvType": {
        "Description": "Environment type.",
        "Default": "test",
        "Type": "String",
        "AllowedValues": [
          "prod",
          "test"
        ]
      }
    },
    "Conditions": {
      "CreateProdInstance": {
        "Fn::Equals": [
          {
            "Ref": "EnvType"
          },
          "prod"
        ]
      }
    }

上述定义了一个参数和一个条件（Condition），如果参数 EnvType 等于 prod，则 CreateProdInstance 条件的计算结果为 true。参数 EnvType 是在创建或更新stack时指定的输入参数。定义了condition后如何使用呢？再看一个例子：  

    "ProductionInstance": {
      "Type": "AWS::EC2::Instance",
      "Condition": "CreateProdInstance",
      "Properties": {
        "InstanceType": "c1.xlarge",
        "SecurityGroups": [
          {
            "Ref": "ProdSecurityGroup"
          }
        ],
        "KeyName": {
          "Ref": "ProdKeyName"
        },
        "ImageId": {
          "Fn::FindInMap": [
            "RegionMap",
            {
              "Ref": "AWS::Region"
            },
            "AMI"
          ]
        }
      }
    }

仅当 CreateProdInstance 条件的计算结果为 true 时，才会创建 ProductionInstance 资源。

好了，言归正传。知道了Condition，那WaitCondition又是什么呢？通俗的讲，可以在模板中放置一个等待条件，以便 Heat 可将stack的创建暂停，并在继续创建stack前等待一个信号。

## WaitCondition资源详解
资源定义如下：

    {
      "Type": "AWS::CloudFormation::WaitCondition",
      "Properties": {
        "Count": String,
        "Handle": String,
        "Timeout": String
      }
    }

*Count*  
继续堆栈创建过程之前必须接收的成功信号的数目。当等待条件接收必需数目的成功信号后，会恢复堆栈的创建。如果等待条件在超时期结束前未接收到指定数目的成功信号，则会认为该等待条件已失败并将该堆栈回滚。  
Required: No.  
Type: String.

*Handle*  
对用于发送此等待条件信号的句柄的引用。使用 Ref 内部函数来指定一个 AWS::CloudFormation::WaitConditionHandle 资源。  
Required: Yes.  
Type: String.  

*Timeout*  
等待 Count 属性所指定的信号数目的时间长度（以秒为单位）。  
Required: Yes.  
Type: String.

AWS::CloudFormation::WaitCondition通常和AWS::CloudFormation::WaitConditionHandle一起使用，WaitCondition需要WaitConditionHandle来设置用作信号机制的预签名URL，通过预签名URL发送信号。

典型的场景是：在虚拟机创建后，应用程序部署完毕，才能对外提供服务，此时绑定浮动IP才有意义，所以，浮动IP的绑定要等到应用部署完毕后开始，而不是当虚拟机状态running时（实际上，虚拟机启动系统时，Nova就会认为虚拟机状态为running，此时，服务显然是不可用的），那么如何表明应用程序已经部署完毕了呢？如果读过我之前的一篇[博客](http://lingxiankong.github.io/blog/2013/12/02/heat/)，那么应该知道，CloudFormation提供了一系列的帮助脚本来辅助stack的创建和应用的部署。这里就用到了cfn-signal脚本，通常在应用部署完后调用。下面是一个简单的template例子（为了后面的演示方便，很多参数我都是写死的，需要根据实际情况做修改）：

    Resources:
      MyWaitHandle:
        Type: AWS::CloudFormation::WaitConditionHandle
    
      MyServer:
        Type: AWS::EC2::Instance
        Properties:
          InstanceType: m1.small
          ImageId: F17-x86_64-cfntools
          KeyName: heatkey
          SubnetId: 7cb2cb6f-91d1-4649-a416-9c4c867a0d26
          AvailabilityZone: nova:n0819a69aa40a
          UserData:
            Fn::Base64:
              Fn::Join:
              - "\n"
              - - "#!/bin/bash -v"
                - "PATH=${PATH}:/opt/aws/bin"
                - Fn::Join:
                  - ""
                  - - "cfn-signal -e 0 -r Done '"
                    - {"Ref" : "MyWaitHandle"}
                    - "'"
    
      MyWaitCondition:
        Type: AWS::CloudFormation::WaitCondition
        DependsOn: MyServer
        Properties:
          Handle: {"Ref": "MyWaitHandle"}
          Timeout: 300

## 信号机制
对于CloudFormation来说，cfn-signal调用了curl命令向CloudFormation发送信号，如下：

    curl -T /tmp/a "https://cloudformation-waitcondition-test.s3.amazonaws.com/arn%3Aaws%3Acloudformation%3Aus-east-1%3A034017226601%3Astack%2Fstack-gosar-20110427004224-test-stack-with-WaitCondition--VEYW%2Fe498ce60-70a1-11e0-81a7-5081d0136786%2FmyWaitConditionHandle?Expires=1303976584&AWSAccessKeyId=AKIAIOSFODNN7EXAMPLE&Signature=ik1twT6hpS4cgNAw7wyOoRejVoo%3D"

其中，/tmp/a中的内容为：

    {
      "Status": "SUCCESS",
      "Reason": "Configuration Complete",
      "UniqueId": "ID1234",
      "Data": "Application has completed configuration."
    }

或者，直接一步到位：

    curl -X PUT -H 'Content-Type:' --data-binary '{"Status" : "SUCCESS","Reason" : "Configuration Complete","UniqueId" : "ID1234","Data" : "Application has completed configuration."}' https://cloudformation-waitcondition-test.s3.amazonaws.com/arn%3Aaws%3Acloudformation%3Aus-east-1%3A034017226601%3Astack%2Fstack-gosar-20110427004224-test-stack-with-WaitCondition--VEYW%2Fe498ce60-70a1-11e0-81a7-5081d0136786%2FmyWaitConditionHandle?Expires=1303976584&AWSAccessKeyId=AKIAIOSFODNN7EXAMPLE&Signature=ik1twT6hpS4cgNAw7wyOoRejVoo%3D

其中，发送回的内容遵循如下格式：

    {
      "Status": "StatusValue",
      "UniqueId": "Some UniqueId",
      "Data": "Some Data",
      "Reason": "Some Reason"
    }

**StatusValue** 是SUCCESS或FAILURE；  
**UniqueId** 表示发送至Heat的信号，如果等待条件的 Count 属性大于 1，那么在针对特定等待条件发送的所有信号中，UniqueId 值必须唯一；  
**Data** 是您想要通过信号发送回的任何信息；  
**Reason** 为字符串，除了要符合 JSON 格式外，对其内容无任何其他限制。

对于Heat来说，从虚拟机内部访问的URL如下(已经做了%3A和%2F的转换)：

    http://172.30.101.207:8000/v1/waitcondition/arn:openstack:heat::df816333ee04421d96aea3470b36dd51:stacks/waitcondition_test/16b1d980-8fad-44b7-aac7-34f74d38ae22/resources/MyWaitHandle?Timestamp=2014-03-27T03:41:17Z&SignatureMethod=HmacSHA256&AWSAccessKeyId=1599e1ff129a4615ae25ba44099ff677&SignatureVersion=2&Signature=MMS4fBdg8jLZozRd9TYcJVYhMjODY%2B1lzFJTmQq74Yg%3D

## 配置
要使用WaitCondition，需要关注heat配置文件中如下几个配置项：  

    [DEFAULT]
    heat_metadata_server_url=http://<heat metadata server ip>:8000
    heat_waitcondition_server_url=http://<heat waitcondition server ip>:8000/v1/waitcondition

## 实战

### 准备活动

* Havana环境，功能正常(特别是网络)
* 镜像注册到Glance，Heat经典的[Fedora镜像](http://fedorapeople.org/groups/heat/prebuilt-jeos-images/F17-x86_64-cfntools.qcow2)，注意镜像名称使用"F17-x86_64-cfntools"
* 模板，直接使用上面那个简单的模板内容，根据实际情况更新模板中使用的SubnetId和AvailabilityZone，保存为"waitcondition_test.template"
* 为方便登录，创建keypair，名称为"heatkey"

相关的命令：

    nova keypair-add heathey > heatkey.pem
    glance image-create --name="F17-x86_64-cfntools" --public --container-format=ovf --disk-format=qcow2 < /home/kong/F17-x86_64-cfntools.qcow2

### 创建stack

    heat stack-create waitcondition_test --template-file=/home/kong/waitcondition_test.template

等待stack创建成功，可以使用`heat event-list waitcondition_test`查看stack创建过程中的事件。

    n548998f4a206:/home/kong # heat event-list waitcondition_test
    +-----------------+----+------------------------+--------------------+----------------------+
    | resource_name   | id | resource_status_reason | resource_status    | event_time           |
    +-----------------+----+------------------------+--------------------+----------------------+
    | MyWaitHandle    | 61 | state changed          | CREATE_IN_PROGRESS | 2014-03-27T03:41:17Z |
    | MyWaitHandle    | 62 | state changed          | CREATE_COMPLETE    | 2014-03-27T03:41:17Z |
    | MyServer        | 63 | state changed          | CREATE_IN_PROGRESS | 2014-03-27T03:41:17Z |
    | MyServer        | 64 | state changed          | CREATE_COMPLETE    | 2014-03-27T03:41:23Z |
    | MyWaitCondition | 65 | state changed          | CREATE_IN_PROGRESS | 2014-03-27T03:41:23Z |
    | MyWaitCondition | 66 | state changed          | CREATE_COMPLETE    | 2014-03-27T03:43:48Z |
    +-----------------+----+------------------------+--------------------+----------------------+

查看虚拟机启动过程，看到如下输出：  
![](/images/2014-03-27-heat-waitcondition/1.png)

## 代码讲解
虽然是代码讲解，但我还是少贴点代码，毕竟是容易变动的东西，主要说一说代码背后的思路。还是以上面的template内容为例，模板中有三个资源，依赖关系是：  
![](/images/2014-03-27-heat-waitcondition/2.png)

在资源映射中，'AWS::CloudFormation::WaitConditionHandle'对应于WaitConditionHandle对象，在创建该资源时，会先创建一个用户：

    def _create_user(self):
        # Check for stack user project, create if not yet set
        if not self.stack.stack_user_project_id:
            project_id = self.keystone().create_stack_domain_project(
                stack_name=self.stack.name)
            self.stack.set_stack_user_project_id(project_id)

        # Create a keystone user in the stack domain project
        user_id = self.keystone().create_stack_domain_user(
            username=self.physical_resource_name(),
            password=self.password,
            project_id=self.stack.stack_user_project_id)

        # Store the ID in resource data, for compatibility with SignalResponder
        db_api.resource_data_set(self, 'user_id', user_id)

以及用户的EC2密钥：

    def _create_keypair(self):
        # Subclasses may optionally call this in handle_create to create
        # an ec2 keypair associated with the user, the resulting keys are
        # stored in resource_data
        user_id = self._get_user_id()
        kp = self.keystone().create_stack_domain_user_keypair(
            user_id=user_id, project_id=self.stack.stack_user_project_id)
        if not kp:
            raise exception.Error(_("Error creating ec2 keypair for user %s") %
                                  user_id)
        else:
            db_api.resource_data_set(self, 'credential_id', kp.id,
                                     redact=True)
            db_api.resource_data_set(self, 'access_key', kp.access,
                                     redact=True)
            db_api.resource_data_set(self, 'secret_key', kp.secret,
                                     redact=True)
        return kp

从模板中可以看到，虚拟机和WaitCondition都依赖于WaitConditionHandle，在资源解析时，`{"Ref" : "MyWaitHandle"}`元素需要调用WaitConditionHandle的FnGetRefId方法获取一个请求URL（`ec2_signed_url`），并且将上述的密钥加入到请求中。创建虚拟机时，作为user-data的内容注入到虚拟机内，由cfn-init在虚拟机内部进行调用。

而在WaitCondition资源的创建中，会在超时时间内循环调用:

    handle_status = handle.get_status()
其实是在获取handle资源的metadata的内容，那么metadata的内容从何而来呢？

在Heat的服务中，有一个cfn服务，在heat的paste.ini中定义如下：

    [app:apicfnv1app]
    paste.app_factory = heat.common.wsgi:app_factory
    heat.app_factory = heat.api.cfn.v1:API

在该服务中，定义了stack和signal资源，在signal资源中就对外提供了`update_waitcondition`API接口（在虚拟机内部会调用这个接口），在业务的处理中，其实就调用了handle的`metadata_update`方法根据请求的参数来更新metadata内容。这样就跟WaitCondition完成了交互，WaitCondition的循环任务就可以结束，该资源就算是创建成功了。
