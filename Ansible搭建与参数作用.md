## Ansible简介：

### ansible是什么？

ansible是自动化运维工具

自动化运维工具那么多，比如（statstack，puppet，cfengine、chef、func、fabric）为什么，偏偏要使用ansible呢？

### 他有哪些好处，那他到底能做些什么呢？

好了，接下来一一说明：

首先ansible是新出现的自动化运维工具，基于Python开发，集合了众多运维工具（puppet、cfengine、chef、func、fabric）的优点，实现了批量系统配置、批量程序部署、批量运行命令等功能，ansible是基于模块工作的，本身没有批量部署的能力。真正具有批量部署的是ansible所运行的模块，ansible只是提供一种框架。

重点：他不像saltstack那种自动化运维工具，还点需要安装客户端才可以配合完成需求，在这方面ansible没有客户端，而且被管理的客户端也无需安装任何插件，也没有服务器端，只需要直接运行把ansible安装上后运行命令即可，是不是很简便呢，ansible只需要将自己的key值分享给客户端节点主机即可实现批量化运维管理。

### ansible大体分为以下几个部分：

(1)、连接插件connection plugins：负责和被监控端实现通信；

(2)、host inventory：指定操作的主机，是一个配置文件里面定义监控的主机；

(3)、各种模块核心模块、command模块、自定义模块；

(4)、借助于插件完成记录日志邮件等功能；

(5)、playbook：剧本执行多个任务时，非必需可以让节点一次性运行多个任务。

![ansible1](/pics/ansible1.jpg)

### ansible的具体优点：

(1)、轻量级，无需在客户端安装agent，更新时，只需在操作机上进行一次更新即可；  
(2)、批量任务执行可以写成脚本，而且不用分发到远程就可以执行；  
(3)、使用python编写，维护更简单，ruby语法过于复杂；  
(4)、支持sudo。

### ansible的运行过程：

![ansible的运行过程](/pics/20180904142014762.jpg)

### 接下来就开始正式安装ansible了

### 实验环境：

### Centos7.2系统为例

### 在安装之前我们要，首先安装python环境，为什么呢？因为之前有提到过ansible是基于python语言开发的，所以ansible的所有功能都是通过python模块来实现的。

### 1.安装python2.7

官网安装包网址：(https://www.python.org/ftp/python/2.7.8/Python-2.7.8.tgz)

下载完后开始解压，配置

```bash
# tar xvzf Python-2.7.8.tgz
# cd Python-2.7.8
# ./configure --prefix=/usr/local
```

编译安装python

```bash
[root@192 Python-2.7.8]# make && make install
```

注意：将python头文件拷贝到标准目录，以避免编译ansible时，找不到所需的头文件

```bash
[root@192 Python-2.7.8]# cd /usr/local/include/python2.7/
[root@192 python2.7]# cp -a ./* /usr/local/include/
```

接下来对旧版本的python做个备份，并符号链接到新版本的python

查看旧版本：

可以看到旧版本为2.7.5

```bash

[root@192 ~]# python -V
Python 2.7.5

[root@192 ~]# cd /usr/bin/
[root@192 bin]# mv python python2.7.5
[root@192 bin]# ln -s /usr/local/bin/python
```

接下来修改yum脚本，使其指向旧版本的python2.7.5，已避免其无法运行

```bash
[root@192 ~]# vim /usr/bin/yum
```

yum脚本内容：

主要修改红颜色的部分

#！/usr/bin/python2.7.5 

```bash
#!/usr/bin/python2.7.5
import sys
try:    
    import yum
except ImportError:    
    print >> sys.stderr, """\
There was a problem importing one of the Python modules
required to run yum. The error leading to this problem was:
```

保存退出，检查一下yum源看是否能正常使用。

```bash
[root@192 ~]# yum list
```

发现执行yum list命令卡主不动了，报了一堆错误，那是因为还有一个配置文件和没有修改

就是这个配置文件：

```bash
vi /usr/libexec/urlgrabber-ext-down
```

进去修改一致即可：

```bash
#! /usr/bin/python2.7.5
#  A very simple external downloader
#  Copyright 2011-2012 Zdenek Pavlas 
#   This library is free software; you can redistribute it and/or
#   modify it under the terms of the GNU Lesser General Public
```

这次yum就可以正常使用了！！！

安装完了python环境，那么接下来就要安装ansible所需要的功能模块了。他们都是基于python提供的

### 2.安装setuptools模块

官网模块网址：(https://pypi.python.org/packages/source/s/setuptools/setuptools-7.0.tar.gz)
