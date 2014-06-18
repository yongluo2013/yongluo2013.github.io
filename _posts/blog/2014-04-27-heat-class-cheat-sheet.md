---
layout: post
title: Heat Class Cheat Sheet
description: Heat Class Cheat Sheet
category: blog
---

声明：  
本博客欢迎转发，但请保留原作者信息!  
新浪微博：[@孔令贤HW](http://weibo.com/lingxiankong)；   
博客地址：<http://lingxiankong.github.io/>  
内容系本人学习、研究和总结，如有雷同，实属荣幸！

从F版本就开始熟悉OpenStack的代码，到现在几大核心模块的代码基本都精读过，但不得不说，目前Heat的代码编写，应该是用到Python高级语法最多的模块，各种对象建模也比其他几个模块复杂一些，初学时比较吃力。这里记录下来，给自己和感兴趣的人参考。

版本：Icehouse

## Dependencies
`__iadd__`：接受一个 (requirer, required) 元组，进行加法操作  

`required_by(self, last)`：返回依赖该资源的所有资源  

`__getitem__(self, last)`：返回一个Dependencies对象。以last作为叶子节点。  

`graph(self, reverse=False)`：返回原来的Graph对象，或逆序的Graph对象，即：将Graph中每个资源的依赖和被依赖关系倒置。  

`__iter__(self)`：从叶子节点开始依次返回各个资源  

## Resource
表示一个资源

`update_template_diff(self, after, before)`：根据两个字典，找到增加/修改/删除的键值对（如果删除，则值是None），键必须在`update_allowed_keys`范围内。  

`update_template_diff_properties(self, after, before)`：根据两个字典，找到增加/修改/删除的属性（如果删除，则值是None），属性必须在`update_allowed_properties`范围内。  

`add_dependencies(self, deps)`：根据资源的属性依赖，构造Dependencies对象的依赖关系。
 
`required_by(self)`： 返回所有依赖该资源的资源名称列表  

`create(self)`：创建资源。创建之前，先进行properties的校验，然后调用handle_create方法。

`get_abandon_data(self)`：返回abandon时的信息

`update(self, after, before=None, prev_resource=None)`：更新资源。如果前后的json字典相同，直接返回；如果资源正在create/update/adopt，则抛失败异常；更新资源的状态(action和status)，若状态变更，则增加event记录；对更新的资源属性进行校验（properties.validate()）；调用`handle_update`

`validate(self)`：校验资源。校验`DeletionPolicy`(若有)，校验资源属性(调用`properties.validate()`)

`delete(self)`：删除资源。如果资源的action属性是init（即还没有创建），直接返回；根据DeletionPolicy（“Delete”或“Snapshot”）分别调用不同的函数；目前貌似还不支持“Retain”

`destroy(self)`：资源的销毁。除了会调用delete，也会删除db中的资源；

`signal(self, details=None)`：在db中增加event表记录，然后调用handle_signal函数。

`resource_to_template(cls, resource_type)`：类方法。根据资源的属性，返回资源模板内容。

`FnGetRefId(self)`：返回资源的resource_id或者资源名称。对于与虚拟机关联的资源，resource_id一般是指虚拟机id

## Properties
表示一个资源的属性集。

`schema_from_params(params_snippet)`：静态方法。根据参数定义字典，返回Schema对象的字典。

`validate(self, with_value=True)`：对实际的属性值进行校验。

`__getitem__(self, key)`：根据key值，找到schema并在实际内容中解析，用schema验证。如果内容中没有，返回default值；若没有default，且该属性必选（prop.required()），抛异常；

`__len__(self)`：属性值的个数

`__contains__(self, key)`：属性中是否包含key

`schema_to_parameters_and_properties(cls, schema)`：类方法。根据入参中的schema，返回参数定义字典和属性字典。被Resource对象的`resource_to_template`方法调用，返回资源的模板。

## ResourceInfo
表示资源名称和资源实现类的关联。有几种不同的实现：ClassResourceInfo，TemplateResourceInfo，GlobResourceInfo，MapResourceInfo，实现了`__lt__`、`__gt__`等方法，可以对ResourceInfo对象进行排序操作。

ClassResourceInfo：很简单，从资源名称就可找到资源类（get_class(self)）  
TemplateResourceInfo：资源对应的类（一个模板）是TemplateResource（没看的太深入）  
GlobResourceInfo：主要用来关联批量映射的情况：`OS::Networking::* -> OS::Neutron::*`，该类继承自MapResourceInfo。  
MapResourceInfo：关联单个映射，比如`OS::Networking::FloatingIp -> OS::Neutron::FloatingIp`

后面两个在查找实现类时，需要依赖ResourceRegistry对象。

## ResourceRegistry
资源映射的管理类。

`load(self, json_snippet)`：根据字典信息注册资源（ResourceInfo）

`register_class(self, resource_type, resource_class)`：同上。

`iterable_by(self, resource_type, resource_name=None)`：根据条件迭代返回资源ResourceInfo对象

`get_resource_info(self, resource_type, resource_name=None, registry_type=None)`：从global注册表和自身的注册表中查找满足条件的ResourceInfo对象（会对满足条件的对象从小到大排序）。

`get_class(self, resource_type, resource_name=None)`：上述函数返回ResourceInfo对象，而这个函数返回对应的实现类。

`get_types(self, support_status)`：获取有效资源的名称列表

> 根据上述两个类的设计，在使用Heat时，可以自定义资源类型和资源实现，可以增加类型，删除系统已有类型，修改系统已有类型的实现等。感兴趣的话可以在程序中打印一下ResourceRegistry对象，看里面都保存了什么信息。

## Environment
维护一些heat在处理时用到的环境变量，有global env和user env。这个类中的方法，大部分都是直接调用ResourceRegistry对象的方法

## PluginManager
初始化时，从入参表示的包以及`CONF.plugin_dirs`下的heat.engine包中加载模块

`map_to_modules(self, function)`：对每一个模块进行函数调用

## PluginMapping
一般与PluginManager共同使用

`load_from_module(self, module)`：调用模块中的`%s_mapping`方法并返回

`load_all(self, plugin_manager)`：对plugin_manager中的模块们调用`load_from_module`方法

## Template
类中有一个描述符_plugins，访问它时，会获取到一个PluginManager对象，初始化它的入参是Template子类所在的包。

`load(cls, context, template_id)`：类方法，从db中获取数据，构造对象

`store(self, context=None)`：保存信息至数据库（如果还没有id）

`__iter__(self)`：返回所有可访问的section

`__len__(self)`：返回上述section的长度

`param_schemata(self)`：需要子类实现的方法。返回参数和参数模式的字典

`parameters(self, stack_identifier, user_params, validate_value=True, context=None)`：需要子类实现的方法。返回Parameters对象

`functions(self)`：返回与子类配套的函数映射

`parse(self, stack, snippet)`：利用上述函数返回的映射，解析入参中的字典

`validate(self)`：简单校验。看有无不存在的section，模板中包含最基本的resource资源，每个resource资源包含Type定义。

> Heat目前实现的有HeatTemplateFormatVersion、AWSTemplateFormatVersion和heat_template_version（HOT）三种模板类型

## Stack
Stack对象是Heat中最重要的对象。Heat基本就是围绕Stack在开展业务。  
Stack初始化时，会确保global env的初始化。

`load(cls, context, stack_id=None, stack=None, resolve_data=True, parent_resource=None, show_deleted=True)`：从db中获取数据，构造stack对象

`store(self, backup=False)`：保存stack信息到db，如果是创建备份，stack名称后加星号。根据`CONF.deferred_auth_method`配置项，创建user_creds表，stack中的`user_creds_id`就表示这个信息。stack中的parameters中的`AWS::StackId`指的是stack的arn

`__iter__(self)`：对stack迭代时，依次返回资源（无顺序），同理`__len__`返回资源总数（这个跟`total_resources`的不同之处在于，后者返回包含嵌套资源的总数）

`resource_by_refid(self, refid)`：根据refid查找stack中资源。会调用各个资源的`FnGetRefId`方法。

`register_access_allowed_handler(self, credential_id, handler)`：在stack中为某一个用户注册一个回调函数。

`access_allowed(self, credential_id, resource_name)`：获取回调并调用，判断对资源是否有操作权限

`validate(self)`：会用到模板中parameter_groups段（干啥的？）。校验模板中是否有不支持的section；校验是否有参数和资源名称冲突；校验每一个资源（validate）

`requires_deferred_auth(self)`：stack中是否有需要延迟鉴权的资源操作

`state_set(self, action, status, reason)`：将最新的stack状态更新db，同时，**会发送通知**，需要配置`notification_driver`。

`preview_resources(self)`：调用每个resource的preview方法，返回一个list

`create(self)`：创建stack。从叶子节点开始（所有的叶子节点同时创建），创建成功后，再创建依赖它的资源节点，这些动作发生在DependencyTaskGroup对象中。调用每个Resource对象的create方法。

`adopt(self)`：从已有资源创建stack，需要在创建stack时提供`adopt_stack_data`参数

`update(self, newstack)`：更新stack。

* 更新db的信息(action and status), 发送开始更新的通知, 将旧stack在db中备份（名称以星号结尾，ownerid就是stack的id），返回备份的stack对象，stack中的resource对象初始状态是(init, complete)
* 更新stack对象的(内存)属性
* 更新的动作发生在engine/update.py文件中。更新时，会构造一个Dependencies对象，包含新stack资源的顺序、旧stack资源的逆序（因为涉及删除），以及相同名称的旧资源对新资源的依赖（保证新资源创建后才能删除旧资源）。分情况处理：  
1、新stack中的资源。如果资源在旧stack中有，更新旧资源，调用resource.update()，若在更新过程中跑出了UpdateReplace异常，则创建新资源；否则，直接创建新资源。  
2、旧stack中的资源。如果资源在新stack中有，返回；否则，删除旧stack中的实际资源和资源对象。

`delete(self, action=DELETE, backup=False)`：删除stack。删除backup，逆序依次删除资源，删除user_creds表记录，去keystone删除trust，删除domain_project，最后删除db记录。

`suspend(self)`和`resume(self)`：没啥好说的。

`restart_resource(self, resource_name)`：先destroy该资源以及对其的所有依赖，然后创建

`get_availability_zones(self)`：调用Nova接口获取AZ名称列表