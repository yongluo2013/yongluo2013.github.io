---
layout: post
title: linux里install命令和cp命令
description: 在看Heat代码目录时看到了install命令，不是很理解，google后，记下以便后续查看
category: blog
---

install和cp类似，都可以将文件/目录拷贝到指定的地点。但是，install允许你控制目标文件的属性。install通常用于程序的makefile（在RPM的spec里面也经常用到），使用它来将程序拷贝到目标（安装）目录。

install主要用法如下：  
install [OPTION]... SOURCE... DIRECTORY  
此时，DIRECTORY必须存在，否则被当成新的文件  
install [OPTION]... -t DIRECTORY SOURCE...  
install [OPTION]... -d DIRECTORY...  
如果目录不存在则创建  
-b：为每个已存在的目的地文件进行备份；  
-D：创建目的地前的所有目录，然后将来源复制到目的地  
-g：自行设置所属的组；  
-m：自行设置权限，而不是默认的rwxr-xr-x  
-o：自行设置所有者  
-p：以来源文件的修改时间作为相应的目的地的文件属性

例如：  

	@install -d /usr/bin
	@install -p -D -m 0755 targets /usr/bin
	相当于
	@mkdir -p /usr/bin
	@cp targets /usr/bin
	@chmod 755 /usr/bin/targets
	@touch /usr/bin/tagets       <---- 更新文件时间戳
	<----@前缀的意思是不在控制台输出结果。

install和cp完成同样的任务--拷贝文件，它们之间的区别主要如下：  
1、最重要的一点，如果目标文件存在，cp会先清空文件后往里写入新文件，而install则会先删除掉原先的文件然后写入新文件。这是因为往正在使用的文件中写入内容可能会导致一些问题，比如说写入正在执行的文件可能会失败，再比如说往已经在持续写入的文件句柄中写入新文件会产生错误的文件。而使用install先删除后写入（会生成新的文件句柄）的方式去安装就能避免这些问题了；

2、install命令会恰当地处理文件权限的问题。比如说，install -c会把目标文件的权限设置为rwxr-xr-x；

3、install命令可以打印出更多更合适的debug信息，还会自动处理SElinux上下文的问题。




