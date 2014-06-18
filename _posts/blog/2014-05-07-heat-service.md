---
layout: post
title: Heat服务简介
description: Heat服务简介
category: blog
---

声明：  
本博客欢迎转发，但请保留原作者信息!  
新浪微博：[@孔令贤HW](http://weibo.com/lingxiankong)；   
博客地址：<http://lingxiankong.github.io/>  
内容系本人学习、研究和总结，如有雷同，实属荣幸！

Heat中对外提供服务的进程有：heat-api, heat-api-cfn, heat-api-cloudwatch，内部进程只有一个heat-engine。这些进程都支持灵活部署。

访问heat-api时，在keystone鉴权时可以提供用户token，或者用户名和密码（此时不需要在系统中提供管理员token），需要配置：  

    [paste_deploy]
    flavor = standalone

或者，使用第三方的云服务鉴权（不使用keystone），配置：

    flavor = custombackend
此时，通过调用heat-engine中的`authenticated_to_backend`方法进行鉴权。

而heat-api-cfn和heat-api-cloudwatch在配置standalone时仅使用EC2鉴权。

## heat-api-cfn
heat-api-cfn代码中，对于stack资源，函数和对外提供API的对应关系及说明：  

    _actions = {
        'list': 'ListStacks', #查询所有stack信息
        'create': 'CreateStack', #创建stack
        'describe': 'DescribeStacks', #查询stack详情
        'delete': 'DeleteStack', #删除stack
        'update': 'UpdateStack', #更新stack
        'events_list': 'DescribeStackEvents', #查询stack事件
        'validate_template': 'ValidateTemplate', #校验模板
        'get_template': 'GetTemplate', #获取stack的模板
        'estimate_template_cost': 'EstimateTemplateCost', #评估模板的计费，未实现
        'describe_stack_resource': 'DescribeStackResource', #查询模板中的指定资源
        'describe_stack_resources': 'DescribeStackResources', #查询模板中的所有资源详情
        'list_stack_resources': 'ListStackResources', #查询模板中的所有资源
    }

另外，heat-api-cfn还对外提供了signal资源的操作，URL分别是`/waitcondition/{arn:.*}`和`/signal/{arn:.*}`，这两个函数之前的博客中有提到过，基本上都是接受来自虚拟机中的请求。

waitcondition：主要用来处理模板中的waitcondition资源，该资源创建后会等待从虚拟机中发来信号，以便条件满足后继续进行stack的创建。虚拟机发信号其实就是调用heat-api-cfn对外提供的接口，底层会调用heat-engine的metadata-update方法。虚拟机发送消息的URL类似于如下格式：

    http://172.30.101.207:8000/v1/waitcondition/arn:openstack:heat::df816333ee04421d96aea3470b36dd51:stacks/waitcondition_test/16b1d980-8fad-44b7-aac7-34f74d38ae22/resources/MyWaitHandle?Timestamp=2014-03-27T03:41:17Z&SignatureMethod=HmacSHA256&AWSAccessKeyId=1599e1ff129a4615ae25ba44099ff677&SignatureVersion=2&Signature=MMS4fBdg8jLZozRd9TYcJVYhMjODY%2B1lzFJTmQq74Yg=

signal：提供给告警服务作为alarm-action调用，向Heat中发送告警消息，更新某些资源的状态。底层调用heat-engine的resource-signal方法，最终调用入参中对应资源的signal方法（如果有的话），会在db中记录对应的事件。目前，实现handle-signal方法的资源有`OS::Heat::HARestarter`、`AWS::AutoScaling::ScalingPolicy`，以及`OS::Heat::SoftwareDeployment`，HARestarter作为告警动作会重新创建资源中的虚拟机资源；ScalingPolicy收到告警时会根据policy的属性调整scalinggroup中的虚拟机个数。

> 关于Heat中的HA，可以参考我同事的这篇[博客](http://kiwik.github.io/openstack/2014/03/22/Heat-HA/)，文章内容有需要改进的地方。

## heat-api-cloudwatch
heat-api-cloudwatch代码中，对于watch资源，函数和对外提供API的对应关系及说明：

    _actions = {
        'delete_alarms': 'DeleteAlarms', #未实现
        'describe_alarm_history': 'DescribeAlarmHistory', #未实现
        'describe_alarms': 'DescribeAlarms', #根据参数查询alarm信息(watch-rule表)
        'describe_alarms_for_metric': 'DescribeAlarmsForMetric', #未实现
        'disable_alarm_actions': 'DisableAlarmActions', #未实现
        'enable_alarm_actions': 'EnableAlarmActions', #未实现
        'get_metric_statistics': 'GetMetricStatistics', #未实现
        'list_metrics': 'ListMetrics', #查询alarm相关的数据(watch-data表)
        'put_metric_alarm': 'PutMetricAlarm', #未实现
        'put_metric_data': 'PutMetricData', #新增watch-data记录
        'set_alarm_state': 'SetAlarmState', #修改watch-rule的状态
    }

可以看到heat-api-cloudwatch中实现的接口并不多，是因为该功能会被陆续转到Ceilometer实现。

在Heat中，与heat-api-cloudwatch服务相关联的资源有`OS::Heat::CWLiteAlarm`、`OS::Ceilometer::Alarm`，前者在创建时会增加watch-rule记录，在Heat的循环任务中对watch-rule进行判断和处理，而后者除了增加记录外(为了兼容)，还会向Ceilometer请求创建Alarm。虚拟机内部调用PutMetricData接口(cfn-push-stats)，会增加watch-rule相关联的watch-data数据，对于前者就是增加db记录，对于后者还是调用Ceilometer接口。

## heat-api
这个服务是Heat中的主服务，其对外提供的API和内部函数的对应关系可以参考`heat.api.openstack.v1.__init__.py`的实现。