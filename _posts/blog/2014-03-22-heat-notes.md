---
layout: post
title: Heat笔记
description: 
category: blog
---

声明：  
本博客欢迎转发，但请保留原作者信息!  
新浪微博：[@孔令贤HW](http://weibo.com/lingxiankong)；   
博客地址：<http://lingxiankong.github.io/>  
内容系本人学习、研究和总结，如有雷同，实属荣幸！
  
## Heat OpenStack API  
  
    path_prefix="/{tenant_id}"
    validate_template /validate
    list_resource_types /resource_types
    resource_schema  /resource_types/{type_name}
    generate_template /resource_types/{type_name}/template
    index /stacks
    create /stacks
    preview /stacks/preview
    detail /stacks/detail
    lookup /stacks/{stack_name}
    lookup /stacks/{stack_name}/{path:resources|events|templates|actions}
    show /stacks/{stack_name}/{stack_id}
    template /stacks/{stack_name}/{stack_id}/template
    update /stacks/{stack_name}/{stack_id}
    delete /stacks/{stack_name}/{stack_id}
    abandon /stacks/{stack_name}/{stack_id}/abandon

    path_prefix=/{tenant_id}/stacks/{stack_name}/{stack_id}
    index /resources
    show /resources/{resource_name}
    metadata /resources/{resource_name}/metadata
    signal /resources/{resource_name}/signal

    path_prefix=/{tenant_id}/stacks/{stack_name}/{stack_id}
    index /events
    index /resources/{resource_name}/events
    show /resources/{resource_name}/events/{event_id}

    path_prefix=/{tenant_id}/stacks/{stack_name}/{stack_id}
    action /actions(suspend/resume)

    path_prefix="/{tenant_id}"
    build_info /build_info

    path_prefix="/{tenant_id}/software_configs"
    create 
    show /{config_id}
    delete /{config_id}

    path_prefix='/{tenant_id}/software_deployments'
    index
    metadata /metadata/{server_id}
    create
    show /{deployment_id}
    update /{deployment_id}
    delete /{deployment_id}
  
## heat-engine initialization  
initialize global \_environment variable  
  
    chain('ABC', 'DEF') --> A B C D E F
    chain.from_iterable(['ABC', 'DEF']) --> A B C D E F 
  
load modules in heat.engine.resources and cfg.CONF.plugin\_dirs, register with global env, and then load user-env files(cfg.CONF.environment\_dir)  
start listener(consume threads)  
get all the stacks in db, start watch(timer) for each which has watch rule  

## stack
### create stack  
the request params from heat-api to heat-engine：  
stack\_name, template\_content, user\_env(params + resouce\_registry), file, args(timeout\_mins/disable\_rollback/adopt\_stack\_data)  
  
### heat-engine create stack  
create a new Template Object, and check some constraints:  
  
* stackname cannot be duplicated  
* number of tenant's stacks cannot exceed `cfg.CONF.max_stacks_per_tenant`  
* number of resources belonging to one stack cannot exceed `cfg.CONF.max_resources_per_stack`  
  
create a user env based on global env;  
verify template params and provided params;  
resolve static data for outputs;  
resolve resources;  
resolve dependencies;  
validate template:  
  
* param name cannot duplicate with resource name  
* circular dependency should not exist  
* validate each resource  
  
create template in db;  
create trusts if `cfg.CONF.deferred_auth_method` is 'trusts';  
create stack in db;  
acquire a lock, then create stack in a thread, start the thread;  
if you adopt a stack(create stack with all the existing resources) or create a stack with new resources, 'stack\_task' will be running;  
when the creation is finished, schedule a periodic watcher task for this stack, just as what heat-engine initialization does.  

### Dependencies

* `Ref `
* `Fn::GetAtt`, e.g. {"Fn::GetAtt": ["DBServer", "PrivateIp"]}
* `DependsOn`

Heat will create resources in parallel. Each resource will be created as soon as all of its dependencies are complete.
  
### watcher task  
fetch rules for a stack in 'watch\_rule'(name, rule, state, last\_evaluated, stack\_id, watch\_data, ) table

### preview stack
related bp: <https://blueprints.launchpad.net/heat/+spec/preview-stack>  
related AWS feature: <http://docs.aws.amazon.com/AWSCloudFormation/latest/APIReference/API_EstimateTemplateCost.html>

### update stack
the params of update-stack is almost the same with create-stack. In heat-engine:

* get the existing stack from the db, validate its status(can not be suspend or in-progress)
* get new stack object, validate
* get a thread from ThreadGroupManager, related to the existing stack
* call update method of existing stack object
* in the 'taskrunner',  change the db state(in_progress), then backup stack(original name is suffixed with \* ), then update.
* delete the resources not in the new stack, update existed resources, add new resources. 

debug log: "Deleting backup resource"

## Software Deployment
There are several bootstrap methods for software deployment:   
1. Create image with application ready to go  
2. Use `cloud-init` to run a startup script passed as userdata to nova  
3. Use the CloudFormation instance helper scripts

scripts in terms of choice 3:  
cfn-init - Reads the `AWS::CloudFormation::Init` for the instance resource, installs packages, and starts services;  
cfn-signal - Waits for an application to be ready before continuing, ie: supporting the WaitCondition feature;  
cfn-hup - Handle updates from the `UpdateStack` CloudFormation API call
  
## Heat Extention  
### Template   
all the TemplateClass are in the heat.engine package, 'template\_mapping' in the file  
>假设T是一个类，t是它的一个实例，d是T的一个descriptor属性，value是一个有效值：  
>读取属性时，如T.d,返回的是d.\_\_get\_\_(None, T)的结果，t.d返回的是d.\_\_get\_\_(t, T)的结果。  
>设置属性时，t.d = value，实际上调用d.\_\_set\_\_(t, value)，T.d = value，这是真正的赋值，T.d的值从此变成value  
  
### Environment
except for heat.engine.resources package, you can specify `cfg.CONF.plugin_dirs` as heat.engine.plugins system module path. The resource\_mapping function is defined in each resource definition file(e.g. heat.engine.resources.instance). There are a few ResourceInfo classes: TemplateResourceInfo, ClassResourceInfo, GlobResourceInfo, MapResourceInfo. 

you can define your own global environment files in `cfg.CONF.environment_dir`, they will take precedence over default global env.

Or, you specify your own env in the request body for your special need.

### Clients  
`cfg.CONF.cloud_backend`, default value is heat.engine.clients.OpenStackClients 

### Notification
when stack's state changed in the database, a notification will be sent if `CONF.notification_driver` is configured properly.