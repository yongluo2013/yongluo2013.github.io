---
layout: post
title: how to run tempest in devstack within vmware workstation
description: how to run tempest in devstack within vmware workstation
category: blog
---

声明：  
本博客欢迎转发，但请保留原作者信息!  
新浪微博：[@孔令贤HW](http://weibo.com/lingxiankong)；   
博客地址：<http://lingxiankong.github.io/>  
内容系本人学习、研究和总结，如有雷同，实属荣幸！

----------------------

## 安装vmware workstation

## 创建ubuntu虚拟机
下载ubuntu iso，网络模式nat（前提是本机能联网），安装过程不需要人工干预。

## 预配置虚拟机
用创建虚拟机时指定的用户登录，修改root登录密码：

> sudo passwd

切换到root用户。修改apt源：

	cp /etc/apt/sources.list /etc/apt/sources.list.bak
	vi /etc/apt/sources.list 
	:%s/us.archive/cn.archive/g

更新软件：`apt-get update`  
安装ssh：`apt-get install openssh-server`  
安装vim: `apt-get install vim`  
安装git: `apt-get install git`  
查看虚拟机IP，然后在本机通过ssh登录，方便后续操作。  
配置pip国内源，新建`~/.pip/pip.conf`文件，输入如下内容：

>[global]  
index-url=http://mirrors.tuna.tsinghua.edu.cn/pypi/simple

重新以root身份登录，以使pip源生效。

## 安装devstack
假设后续都是在/openstack目录下操作。执行：

>git clone git://github.com/openstack-dev/devstack.git   
cd devstack  
chmod +x tools/create-stack-user.sh  
./tools/create-stack-user.sh #创建stack用户  
chown -R stack:stack /openstack/devstack  
su - stack  
vi /openstack/devstack/localrc #新建localrc文件

输入如下内容：

	# Misc
	HOST_IP=192.168.70.132
	DATABASE_PASSWORD=Galax8800
	ADMIN_PASSWORD=Galax8800
	SERVICE_PASSWORD=Galax8800
	SERVICE_TOKEN=Galax8800
	RABBIT_PASSWORD=Galax8800
	
	# Enable Logging
	
	LOGFILE=/opt/stack/logs/stack.sh.log
	VERBOSE=True
	SCREEN_LOGDIR=/opt/stack/logs
	
	# Pre-requisite
	ENABLED_SERVICES=rabbit,mysql,key
    KEYSTONE_TOKEN_FORMAT=UUID
	
	# 使用csdn的代码仓库，也可以不使用
	# GIT_BASE=https://code.csdn.net
	
	# Nova
	ENABLED_SERVICES+=,n-api,n-crt,n-obj,n-cpu,n-cond,n-sch
	#IMAGE_URLS+=",https://launchpad.net/cirros/trunk/0.3.0/+download/cirros-0.3.0-x86_64-disk.img"
	
	#Horizon
	ENABLED_SERVICES+=,horizon
	
	# Glance
	ENABLED_SERVICES+=,g-api,g-reg
	
	# Neutron
	ENABLED_SERVICES+=,q-svc,q-agt,q-dhcp,q-l3,q-meta,neutron
	
	# Cinder
	ENABLED_SERVICES+=,cinder,c-api,c-vol,c-sch
	
	# Heat - Orchestration Service
	ENABLED_SERVICES+=,heat,h-api,h-api-cfn,h-api-cw,h-eng
	#IMAGE_URLS+=",http://fedorapeople.org/groups/heat/prebuilt-jeos-images/F17-x86_64-cfntools.qcow2"
	
	# Ceilometer - Metering Service (metering + alarming)
    CEILOMETER_BACKEND=mongo
	ENABLED_SERVICES+=,ceilometer-acompute,ceilometer-acentral,ceilometer-collector,ceilometer-api
	ENABLED_SERVICES+=,ceilometer-alarm-notifier,ceilometer-alarm-evaluator

保存文件，然后切换到stack用户执行`./stack.sh`，根据本机网速，自动安装all-in-one的devstack环境.  
看到下面这句话时，证明安装成功：  
>2013-10-01 06:15:12 stack.sh completed in 676 seconds.  

## screen
使用screen -r XXX时有时会出现：`Cannot open your terminal '/dev/pts/2' - please check.`的提示，不要急，使用如下方法解决：  
使用root用户执行：

    chown stack:stack `readlink /proc/self/fd/0`
    
如果不想使用screen，则可以修改stackrc文件中的`USE_SCREEN=False`    

## 验证安装
按照下述步骤，看功能是否OK：  
![](/images/2014-05-10-vmware-workstation-devstack/1.png)  

## 配置tempest
现在已经有了一个可运行的OpenStack环境，可将本机修改过的tempest工程通过winscp工具复制到ubuntu虚拟机上。假设是这个目录`/openstack/code/tempest`，我们需要一个tempest配置文件。执行：  
>cp /openstack/code/tempest/etc/tempest.conf.sample /openstack/code/tempest/etc/tempest.conf  

根据你的devstack环境，主要修改其中的以下配置：  

	admin_password、image_ref

因为是在虚拟机中安装devstack，所以对于规格尽量占的资源少（主要是内存）因此修改数据库中的flavor：  

	mysql -uroot -pGalax8800
	mysql> use nova;
	mysql> update instance_types set memory_mb=100 where id=2;

## 一切就绪，启动tempest用例

	root@ubuntu:/openstack/code/tempest# nosetests -v tempest.api.compute.admin.test_aggregates:AggregatesAdminTestJSON.test_aggregate_create_invalid_aggregate_name

刚执行时可能会出现有些python库没有安装，pip安装即可：  
>pip install testtools  
pip install fixtures  
pip install testresources  



