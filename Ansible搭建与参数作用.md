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

官网安装包网址：https://www.python.org/ftp/python/2.7.8/Python-2.7.8.tgz

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

官网模块网址：https://pypi.python.org/packages/source/s/setuptools/setuptools-7.0.tar.gz

```bash
[root@192 ~]# tar zxf setuptools-7.0.tar.gz 
[root@192 ~]# cd setuptools-7.0/
[root@192 setuptools-7.0]# python setup.py install
```

运行时候，突然出现错误，如下图所示

```bash
creating 'dist/setuptools-7.0-py2.7.egg' and adding 'build/bdist.linux-x86_64/egg' to it
Traceback (most recent call last):  
  File "setup.py", line 219, in <module>    
    dist = setuptools.setup(**setup_params)  
  File "/usr/local/lib/python2.7/distutils/core.py", line 151, in setup    
    dist.run_commands()  
  File "/usr/local/lib/python2.7/distutils/dist.py", line 953, in run_commands    
    self.run_command(cmd)  
  File "/usr/local/lib/python2.7/distutils/dist.py", line 972, in run_command    
    cmd_obj.run()  
  File "/root/setuptools-7.0/setuptools/command/install.py", line 67, in run    
    self.do_egg_install()  
  File "/root/setuptools-7.0/setuptools/command/install.py", line 109, in do_egg_install    
    self.run_command('bdist_egg')  
  File "/usr/local/lib/python2.7/distutils/cmd.py", line 326, in run_command    
    self.distribution.run_command(command)  
  File "/usr/local/lib/python2.7/distutils/dist.py", line 972, in run_command    
    cmd_obj.run()  
  File "/root/setuptools-7.0/setuptools/command/bdist_egg.py", line 223, in run    
    dry_run=self.dry_run, mode=self.gen_header())  
  File "/root/setuptools-7.0/setuptools/command/bdist_egg.py", line 472, in make_zipfile    
    z = zipfile.ZipFile(zip_filename, mode, compression=compression)  
  File "/usr/local/lib/python2.7/zipfile.py", line 736, in __init__    
    "Compression requires the (missing) zlib module"
RuntimeError: Compression requires the (missing) zlib module
[root@192 setuptools-7.0]# 
```

出错的原因：

提示的很清楚，缺少 zlib模块导致安装失败

解决方式:

```bash
[root@192 ~]# yum -y install zlib zlib-devel
```

下载成功后，进入python2.7的目录，重新执行 

```bash
[root@192 ~]# cd Python-2.7.8/
[root@192 Python-2.7.8]# make && make install 
```

此时先前执行的 软连接仍旧生效 

然后进入 setuptool目录，重新安装

```bash
[root@192 setuptools-7.0]# python setup.py install
```

安装成功显示结果：

```bash
copying setuptools.egg-info/requires.txt -> build/bdist.linux-x86_64/egg/EGG-INFO
copying setuptools.egg-info/top_level.txt -> build/bdist.linux-x86_64/egg/EGG-INFO
creating 'dist/setuptools-7.0-py2.7.egg' and adding 'build/bdist.linux-x86_64/egg' to it
removing 'build/bdist.linux-x86_64/egg' (and everything under it)
Processing setuptools-7.0-py2.7.egg
Copying setuptools-7.0-py2.7.egg to /usr/local/lib/python2.7/site-packages
Adding setuptools 7.0 to easy-install.pth file
Installing easy_install script to /usr/local/bin
Installing easy_install-2.7 script to /usr/local/bin

Installed /usr/local/lib/python2.7/site-packages/setuptools-7.0-py2.7.egg
Processing dependencies for setuptools==7.0
Finished processing dependencies for setuptools==7.0
[root@192 setuptools-7.0]# 
```

### 3.安装pycrypto模块

官方模块网址：https://pypi.python.org/packages/source/p/pycrypto/pycrypto-2.6.1.tar.gz

```bash
[root@192 ~]# tar zxf pycrypto-2.6.1.tar.gz 
[root@192 ~]# cd pycrypto-2.6.1/
[root@192 pycrypto-2.6.1]# python setup.py install
```

### 4.安装PyYAML模块

官方模块网址：https://pypi.python.org/packages/source/P/PyYAML/PyYAML-3.11.tar.gz

```bash
[root@192 ~]# tar zxf PyYAML-3.11.tar.gz 
[root@192 ~]# cd PyYAML-3.11/
[root@192 PyYAML-3.11]# python setup.py install
```

### 5.安装Jinja2模块

官方模块网址：https://pypi.python.org/packages/source/M/MarkupSafe/MarkupSafe-0.9.3.tar.gz

```bash
[root@192 ~]# tar zxf MarkupSafe-0.9.3.tar.gz 
[root@192 ~]# cd MarkupSafe-0.9.3/
[root@192 MarkupSafe-0.9.3]# python setup.py install
```

官方模块网址：https://pypi.python.org/packages/source/J/Jinja2/Jinja2-2.7.3.tar.gz

```bash
[root@192 ~]# tar zxf Jinja2-2.7.3.tar.gz 
[root@192 ~]# cd Jinja2-2.7.3/
[root@192 Jinja2-2.7.3]# python setup.py install
```

### 6.安装paramiko模块

官方模块网址：https://pypi.python.org/packages/source/e/ecdsa/ecdsa-0.11.tar.gz

```bash
[root@192 ~]# tar zxf ecdsa-0.11.tar.gz 
[root@192 ~]# cd ecdsa-0.11/
[root@192 ecdsa-0.11]# python setup.py install
```

官方模块网址：https://pypi.python.org/packages/source/p/paramiko/paramiko-1.15.1.tar.gz

```bash
[root@192 ~]# tar zxf paramiko-1.15.1.tar.gz 
[root@192 ~]# cd paramiko-1.15.1/
[root@192 paramiko-1.15.1]# python setup.py install
```

### 7.安装simplejson模块

官方模块网址：https://pypi.python.org/packages/source/s/simplejson/simplejson-3.6.5.tar.gz

```bash
[root@192 ~]# tar zxf simplejson-3.6.5.tar.gz 
[root@192 ~]# cd simplejson-3.6.5/
[root@192 simplejson-3.6.5]# python setup.py install
```

接下来开始安装ansible以及进行ansible的相关配置

官方ansible安装包网址：https://codeload.github.com/ansible/ansible/tar.gz/v1.7.2

```bash
[root@192 ~]# tar zxf ansible-1.7.2.tar.gz 
[root@192 ~]# cd ansible-1.7.2/
[root@192 ansible-1.7.2]# python setup.py install
```

### 8.ansible的配置：

（1.）SSH免密钥登录设置

```bash
[root@192 ~]# ssh-keygen -t rsa -P ''
```

 key文件的名字可以随便起，但是一定要对应上。
 
 ```bash
 [root@192 ~]# ssh-keygen -t rsa -P ''
 Generating public/private rsa key pair.
 Enter file in which to save the key (/root/.ssh/id_rsa): /root/.ssh/id_rsa_storm1
 Created directory '/root/.ssh'.
 Your identification has been saved in /root/.ssh/id_rsa_storm1.
 Your public key has been saved in /root/.ssh/id_rsa_storm1.pub.
 The key fingerprint is:
 39:05:df:15:a3:af:4f:dc:30:c1:b8:8c:97:b3:82:bb root@192.168.1.13
 The key's randomart image is:
 +--[ RSA 2048]----+
 |        .     +. |
 |         o . = . |
 |          o + o  |
 |         o o + . |
 |        S . * +  |
 |         o . = + |
 |        . . o o .|
 |         . . o   |
 |        E.    .  |
 +-----------------+
 ```
 
 写入信任key值，受于authorized_keys文件权限：
 
 ```bash
[root@192 ~]# cd /root/.ssh/
[root@192 .ssh]# lsid_rsa_storm1  
id_rsa_storm1.pub
[root@192 .ssh]# cat /root/.ssh/id_rsa_storm1.pub >> /root/.ssh/authorized_keys
[root@192 .ssh]# chmod 600 /root/.ssh/authorized_keys 
```

接下来将authorized_keys文件分发拷贝到所有客户主机上：

```bash
[root@192 .ssh]# ls
authorized_keys  id_rsa_storm1  id_rsa_storm1.pub  known_hosts
[root@192 .ssh]# scp authorized_keys root@192.168.1.17:/root/.ssh/
root@192.168.1.17's password: 
authorized_keys                                                                    100%  399     0.4KB/s   00:00    
[root@192 .ssh]# scp authorized_keys root@192.168.1.18:/root/.ssh/
The authenticity of host '192.168.1.18 (192.168.1.18)' can't be established.
ECDSA key fingerprint is 37:8f:0c:17:94:47:51:eb:82:38:47:01:89:f0:ff:5c.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.1.18' (ECDSA) to the list of known hosts.
root@192.168.1.18's password: 
authorized_keys    
```

然后在添加相应的权限：

```bash
[root@192 .ssh]# ls
authorized_keys
[root@192 .ssh]# chmod 600 /root/.ssh/authorized_keys
```

这样免秘钥登录就配置完了，接下来开始配置ansible了

创建ansible主配置目录及配置文件

```bash
[root@192 ~]# mkdir -p /etc/ansible
[root@192 ~]# vim /etc/ansible/ansible.cfg
```

注意：端口和key的文件一定要配置对

```bash
[defaults]
inventory = /etc/ansible/hosts 
sudo_user=root
remote_port=22
host_key_checking=False
remote_user=rootlog_path=/var/log/ansible.log
module_name=command
private_key_file=/root/.ssh/id_rsa_storm1
no_log:True
```

主配置文件配好了，接下来开始配置要管理的节点客户端ip了

```bash
[root@192 ~]# cd /etc/ansible/
[root@192 ansible]# ls
ansible.cfg
[root@192 ansible]# vim hosts
```

```bash
[storm_cluster]
192.168.1.13
192.168.1.17
192.168.1.18
```

到此环境就部署配置好了，接下来进行个简单的测试

```bash
ansible storm_cluster -m command -a 'uptime'
```

如下图所示，免秘钥登录，探测到三台主机存活

```bash
[root@192 ansible]# ansible storm_cluster -m command -a 'uptime'
192.168.1.18 | success | rc=0 >> 
 16:49:19 up  2:07,  4 users,  load average: 0.00, 0.01, 0.13 
192.168.1.13 | success | rc=0 >> 
 16:49:20 up  2:15,  4 users,  load average: 0.08, 0.03, 0.05 
192.168.1.17 | success | rc=0 >> 16:49:22 up  2:12,  4 users,  load average: 0.34, 0.09, 0.07 
[root@192 ansible]#
```

### 9.ansible常用模块的使用

（1.）setup

用来查看远程主机的一些基本信息

```bash
 ansible storm_cluster -m setup
```

（2.）ping

用来测试远程主机的运行状态

```bash
ansible storm_cluster -m ping
```

```bash
[root@192 ansible]# ansible storm_cluster -m ping
192.168.1.13 | success >> {
    "changed": false,     
    "ping": "pong"
} 

192.168.1.18 | success >> { 
    "changed": false,     
    "ping": "pong"
} 

192.168.1.17 | success >> {    
    "changed": false,     
    "ping": "pong"
}
```

（3.）file

设置文件的属性，及相关选项说明

force：需要在两种情况下强制创建软链接，一种是源文件不存在，但之后会建立的情况下；另一种是目标软链接已存在，需要先取消之前的软链，然后创建新的软链，有两个选项：yes|no

group：定义文件/目录的属组

mode：定义文件/目录的权限

owner：定义文件/目录的属主

path：必选项，定义文件/目录的路径

recurse：递归设置文件的属性，只对目录有效

src：被链接的源文件路径，只应用于state=link的情况

dest：被链接到的路径，只应用于state=link的情况

state：

       directory：如果目录不存在，就创建目录

       file：即使文件不存在，也不会被创建

       link：创建软链接

       hard：创建硬链接

       touch：如果文件不存在，则会创建一个新的文件，如果文件或目录已存在，则更新其最后修改时间

       absent：删除目录、文件或者取消链接文件

### 实例：远程文件符号链接创建

```bash
ansible storm_cluster -m file -a "src=/etc/resolv.conf dest=/tmp/resolv.conf state=link"
```

远程文件符号链接删除

```bash
ansible storm_cluster -m file -a "path=/tmp/resolv.conf state=absent"
```

远程文件信息查看

```bash
ansible storm_cluster -m command -a "ls -al /tmp/resolv.conf"
```

上述结果是因为软连接，已被删除掉了

（4.）copy

复制文件到远程主机

相关选项如下：

backup：在覆盖之前，将源文件备份，备份文件包含时间信息。有两个选项：yes|no

content：用于替代“src”，可以直接设定指定文件的值

dest：必选项。要将源文件复制到的远程主机的绝对路径，如果源文件是一个目录，那么该路径也必须是个目录

directory_mode：递归设定目录的权限，默认为系统默认权限

force：如果目标主机包含该文件，但内容不同，如果设置为yes，则强制覆盖，如果为no，则只有当目标主机的目标位置不存在该文件时，才复制。默认为yes

others：所有的file模块里的选项都可以在这里使用

src：被复制到远程主机的本地文件，可以是绝对路径，也可以是相对路径。如果路径是一个目录，它将递归复制。在这种情况下，如果路径使用“/”来结尾，则只复制目录里的内容，如果没有使用“/”来结尾，则包含目录在内的整个内容全部复制，类似于rsync。

### 实例：将本地文件“/etc/ansible/ansible.cfg”复制到远程服务器

```bash
ansible storm_cluster -m copy -a "src=/etc/ansible/ansible.cfg dest=/tmp/ansible.cfg owner=root group=root mode=0644"
```

远程文件信息查看

```bash
ansible storm_cluster -m command -a "ls -al /tmp/ansible.cfg"
```

(5)、command

在远程主机上执行命令，相关选项如下：

creates：一个文件名，当该文件存在，则该命令不执行

free_form：要执行的linux指令

chdir：在执行指令之前，先切换到该目录

removes：一个文件名，当该文件不存在，则该选项不执行

executable：切换shell来执行指令，该执行路径必须是一个绝对路径

示例：

```bash
ansible storm_cluster -m command -a "uptime"
```

(6)、shell  

切换到某个shell执行指定的指令，参数与command相同。  

与command不同的是，此模块可以支持命令管道，同时还有另一个模块也具备此功能：raw  

实例：  

 先在本地创建一个SHELL脚本
 
 ```bash 
vim /tmp/rocketzhang_test.sh
#!/bin/sh
date +%F_%H:%M:%S
```

```bash
#chmod +x /tmp/rocketzhang_test.sh
```

将创建的脚本文件分发到远程

```bash
ansible storm_cluster -m copy -a "src=/tmp/rocketzhang_test.sh dest=/tmp/rocketzhang_test.sh owner=root group=root mode=0755"
```

远程执行 

```bash
 ansible storm_cluster -m shell -a "/tmp/rocketzhang_test.sh"
 ```
 
 (7)、更多模块
 
 相关网站：http://docs.ansible.com/modules_by_category.html
