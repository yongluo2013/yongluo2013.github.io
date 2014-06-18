---
layout: post
title: DevOps--在本地执行远程命令
description: 为了赶时髦，我也使用了DevOps这个词儿
category: blog
---

## ssh
在本地使用 `ssh $RemoteNode <cmd>` 可以在执行远程机器上的命令，例如 `ssh user@node ls /local` 会执行远程机器上的 ls /local 命令，如果想在远程机器上连续执行多条命令，可以用单引号或者双引号将这些命令括起来。
 
如果想在本地启动远程机器上的命令后就返回，可以这样：  
 `ssh user@node "/local/x.sh  >/dev/null  2>&1"`

## pssh
> PSSH provides parallel versions of OpenSSH and related tools. Included are pssh, pscp, prsync, pnuke, and pslurp. The project includes psshlib which can be used within custom applications. The source code is written in Python.

[PSSH][]下载地址：<https://parallel-ssh.googlecode.com/files/pssh-2.3.1.tar.gz>  
使用python编写，使用setup.py正常安装。

[PSSH][]总是通过清单文件指定主机，其中的每行采用 [user@]host[:port] 形式。其主要功能是在多个主机上并行地运行命令。  
`# pssh -P -h /home/server.txt hostname`  
在默认情况下，每个命令实例的输出出现在 stdout 中，输出划分为每个主机一段。但是，可以指定一个目录来捕捉每个实例的输出。例如，如果运行前面的命令并添加 `--outdir=/opt/output/`，那么会把每个主机的命令输出捕捉到/opt/output/ 中单独的文件中

pssh可以生成最多 32 个进程，并行地连接各个节点。如果远程命令在 60 秒内没有完成，连接会终止。如果命令需要更多处理时间，可以使用 -t 设置更长的到期时间。（parallel-scp 和 parallel-rsync 没有默认的到期时间，但是可以用 -t 指定到期时间。）

pssh的参数解释：  
-h 执行命令的远程主机列表  
-l 远程机器的用户名  
-P 执行时输出执行信息  
-p 一次最大允许多少连接  
-o 输出内容重定向到一个文件  
-e 执行错误重定向到一个文件  
-t 设置命令执行的超时时间  
-A 提示输入密码并且把密码传递给ssh  
-O 设置ssh参数的具体配置，参照ssh_config配置文件  
-x 传递多个SSH 命令，多个命令用空格分开，用引号括起来  
-X 同-x 但是一次只能传递一个命令  
-i 显示标准输出和标准错误在每台host执行完毕后  

顺带说一下其他tools：  
pscp--把文件或者目录并行地复制到多个主机上  
`# pscp -h /home/server.txt /home/old_file /opt/new_file`  
`# pscp --recursive -h /home/server.txt /srv/old_dir /opt`  

pslurp--把文件或者目录并行地从多个远程主机复制到中心主机上  
`# pslurp --recursive -h /home/server.txt -L /srv/test/ /srv llll`  
`-L`指定在本地创建子目录的位置  
`llll`为拷贝到本地后的目录名

pnuke--并行地在多个远程主机上杀死进程  

prsync--使用rsync协议从本地计算机同步到远程主机

## Fabric
[fabric][]是一个python命令行工具，通过ssh来部署应用或者完成常规运维任务，安装后，通过fab命令指定fabfile.py文件（当然可以指定为其他文件）中的任务函数。[fabric][]的牛逼之处在于**它能很灵活的为不同的任务配置不同的执行机**，这是ssh或pssh很难自动做到的。网上有很多现成的[fabric][]的教程，我这里就不细讲。列一些我认为比较重要或比较有特色的特性。

任务可以串行(@serial)或并行(@parallel(pool_size=5))执行，执行时每个任务都有各自的hosts列表，命令行中为单个任务指定主机的优先级最高，会覆盖其他地方的配置.  
`fab mytask:hosts="host1;host2"`；  
也可以在代码中通过env.hosts配置，可以在一个task中修改该变量，会影响后面所有的任务；  
或者使用fabric.api.hosts或roles装饰器，如果同时指定会合并，会覆盖env的配置；  
或者直接通过命令行指定，`fab -H host1,host2 mytask`，最早被解析，会被覆盖，或被添加: env.hosts.extend(['host3', 'host4'])

env环境变量：<http://docs.fabfile.org/en/1.8/usage/env.html>  
env.dedupe_hosts置为False不会过滤重复的主机  
可以排除主机：--exclude-hosts/-x  
可以将主机划分角色组

使用execute可以不必在命令行中指定多个task，如下，migrate和update是两个task

    from fabric.api import run, roles, execute
    def deploy():
        execute(migrate)
        execute(update)

同时，execute可以接受hosts参数，这样可以**动态确定hosts**，这对自动化意义重大，一个例子：

    from fabric.api import run, execute, task
    # For example, code talking to an HTTP API, or a database, or ...
    from mylib import external_datastore
    # This is the actual algorithm involved. It does not care about host
    # lists at all.
    def do_work():
        run("something interesting on a host")
    # This is the user-facing task invoked on the command line.
    @task
    def deploy(lookup_param):
        # This is the magic you don't get with @hosts or @roles.
        # Even lazy-loading roles require you to declare available roles
        # beforehand. Here, the sky is the limit.
        host_list = external_datastore.query(lookup_param)
        # Put this dynamically generated host list together with the work to be
        # done.
        execute(do_work, hosts=host_list)

命令行中执行：fab deploy:db

命令行的使用 ：  
<http://docs.fabfile.org/en/1.8/usage/fab.html>

[fabric]: http://docs.fabfile.org/
[PSSH]: https://code.google.com/p/parallel-ssh/