---
layout:     post
title:      使用GoAgent代理访问GitHub
category: blog
description: 在大陆活着是那么的艰难
tags: github, goagent
---

声明：  
本博客欢迎转发，但请保留原作者信息!  
博客地址：<http://lingxiankong.github.io/>  
内容系本人及本人团队学习、研究和总结，如有雷同，实属荣幸！  
Author：华为云计算工程师 [孔令贤](http://weibo.com/lingxiankong)  

预置环境：  
- 操作系统：Win7

步骤：  
1. 到goagent的主页可以找到下载和配置goagent的教程  
2. 修改git https协议的代理为goagent:  
进入主目录：C:\Users\用户名\  
修改.gitconfig文件，添加两行设置：

    [http]  
    proxy = http://127.0.0.1:8087  
    sslVerify = false
    
这里的proxy就是goagent的监听的本地路径。  
sslVerify为不进行SSL证书的验证，不设置会出错，提示：ssl certificate problem verify that the ca cert is ok. details错误

Linux下的配置教程[点击这里](http://weibo.com/1791166224/zfvJvnG39?type=repost)