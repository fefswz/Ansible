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
