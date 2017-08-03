---
title: centos6.x在线搭建RabbitMQ集群
date: 2015-07-12 12:33:10
categories: "服务"
tags: 
     - RabbitMQ
     - Linux
---

#### 1 环境
两个节点

```
172.17.164.17 rabbitmq-node1 #centos6.4
172.17.164.18 rabbitmq-node2 #centos6.4
```
##### 1.1 修改hostname
修改3个地方
###### /etc/hosts

```
172.17.164.17 rabbitmq-node1
172.17.164.18 rabbitmq-node2
```

###### /etc/sysconfig/network

```
HOSTNAME=rabbitmq-node1
HOSTNAME=rabbitmq-node2
```
###### 如果不想重启，直接修改hostname，重新登陆即可

```
[root@localhost ~]# hostname rabbitmq-node1
```

<!-- more -->

#### 2 安装erlang和mq
##### 2.1 安装erlang依赖的基本环境

```
yum -y install make gcc gcc-c++ kernel-devel m4 ncurses-devel openssl-devel
```

##### 2.2 导入erlang源，并安装erlang
```
[root@localhost ~]# rpm --import http://binaries.erlang-solutions.com/debian/erlang_solutions.asc
[root@localhost ~]# wget -O /etc/yum.repos.d/erlang_solutions.repo http://binaries.erlang-solutions.com/rpm/centos/erlang_solutions.repo
 [root@localhost ~]# wget http://apt.sw.be/redhat/el6/en/x86_64/rpmforge/RPMS/rpmforge-release-0.5.2-2.el6.rf.x86_64.rpm
[root@localhost ~]# rpm --import http://apt.sw.be/RPM-GPG-KEY.dag.txt
[root@localhost ~]# rpm -i rpmforge-release-0.5.2-2.el6.rf.*.rpm
[root@localhost ~]# yum update
[root@localhost ~]# yum update --skip-broken
[root@localhost ~]# yum install erlang
```
##### 2.3 安装rabbitmq

```bash
[root@localhost ~]# wget -c http://www.rabbitmq.com/releases/rabbitmq-server/v3.3.0/rabbitmq-server-3.3.0-1.noarch.rpm
[root@localhost ~]# yum -y install rabbitmq-server-3.3.0-1.noarch.rpm
#启动
[root@rabbitmq-node1 ~]# /etc/init.d/rabbitmq-server start
Starting rabbitmq-server: SUCCESS
rabbitmq-server.
[root@rabbitmq-node1 ~]# rabbitmqctl status
Status of node 'rabbit@rabbitmq-node1' ...
[{pid,23358},
 {running_applications,[{rabbit,"RabbitMQ","3.3.0"},
                        {mnesia,"MNESIA  CXC 138 12","4.13.2"},
                        {os_mon,"CPO  CXC 138 46","2.4"},
                        {xmerl,"XML parser","1.3.9"},
                        {sasl,"SASL  CXC 138 11","2.6.1"},
                        {stdlib,"ERTS  CXC 138 10","2.7"},
                        {kernel,"ERTS  CXC 138 10","4.1.1"}]},
 {os,{unix,linux}},
 {erlang_version,"Erlang/OTP 18 [erts-7.2] [source-e6dd627] [64-bit] [smp:4:4] [async-threads:30] [hipe] [kernel-poll:true]\n"},
 {memory,[{total,37743168},
          {connection_procs,2808},
          {queue_procs,5616},
          {plugins,0},
          {other_proc,13735712},
          {mnesia,61312},
          {mgmt_db,0},
          {msg_index,23064},
          {other_ets,792160},
          {binary,13024},
          {code,16616638},
          {atom,654217},
          {other_system,5838617}]},
 {alarms,[]},
 {listeners,[{clustering,25672,"::"},{amqp,5672,"::"}]},
 {vm_memory_high_watermark,0.4},
 {vm_memory_limit,3300463411},
 {disk_free_limit,50000000},
 {disk_free,12553482240},
 {file_descriptors,[{total_limit,924},
                    {total_used,3},
                    {sockets_limit,829},
                    {sockets_used,1}]},
 {processes,[{limit,1048576},{used,127}]},
 {run_queue,0},
 {uptime,39}]
...done.
```
##### 2.4 安装插件管理界面
```
[root@rabbitmq-node1 ~]# mkdir -m 777 /etc/rabbitmq/ 
#如果目录已经存在直接执行  
[root@rabbitmq-node1 ~]# chmod 777 /etc/rabbitmq/

[root@rabbitmq-node1 ~]# rabbitmq-plugins enable rabbitmq_management
The following plugins have been enabled:
  mochiweb
  webmachine
  rabbitmq_web_dispatch
  amqp_client
  rabbitmq_management_agent
  rabbitmq_management
Plugin configuration has changed. Restart RabbitMQ for changes to take effect.

#重启rabbitmq-server
[root@rabbitmq-node1 ~]# rabbitmqctl stop
Stopping and halting node 'rabbit@rabbitmq-node1' ...
...done.
[root@rabbitmq-node1 ~]# /etc/init.d/rabbitmq-server start
Starting rabbitmq-server: SUCCESS
rabbitmq-server.

#查看管理端口有没有启动：
[root@rabbitmq-node1 ~]# netstat -tnlp|grep 15672
```

浏览器打开http://IP:15672 账号密码都是guest。 
注意：rabbitmq从3.3.0开始禁止使用guest/guest权限通过除localhost外的访问。
如果想使用guest/guest通过远程机器访问，需要在rabbitmq配置文件中(/etc/rabbitmq/rabbitmq.config)中设置loopback_users为[]。
#/etc/rabbitmq/rabbitmq.config文件完整内容如下（注意后面的半角句号）：
```
[{rabbit, [{loopback_users, []}]}].
```
##### 2.5 添加管理用户

我们不设置guest远程访问了。
使用命令添加一个新的管理账号(每个节点都要配置)：
```
 [root@rabbitmq-node1 ~]# rabbitmqctl add_user admin htxx51fpmq
Creating user "admin" ...
...done.
#赋予管理员权限：
[root@rabbitmq-node1 ~]# rabbitmqctl set_user_tags admin administrator
Setting tags for user "admin" to [administrator] ...
...done.
#重启服务
[root@rabbitmq-node1 ~]# /etc/init.d/rabbitmq-server restart
Restarting rabbitmq-server: SUCCESS
rabbitmq-server.
```
##### 2.6 开放端口
```
[root@rabbitmq-node2 ~]# /sbin/iptables -I INPUT -p tcp --dport 5672 -j ACCEPT
[root@rabbitmq-node2 ~]# /sbin/iptables -I INPUT -p tcp --dport 15672 -j ACCEPT
[root@rabbitmq-node2 ~]# /etc/rc.d/init.d/iptables save
iptables: Saving firewall rules to /etc/sysconfig/iptables:[  OK  ]
[root@rabbitmq-node2 ~]# /etc/init.d/iptables restart
iptables: Setting chains to policy ACCEPT: filter          [  OK  ]
iptables: Flushing firewall rules:                         [  OK  ]
iptables: Unloading modules:                               [  OK  ]
iptables: Applying firewall rules:                         [  OK  ]
[root@rabbitmq-node2 ~]# /etc/init.d/iptables status
```
#### 3 配置集群
##### 3.1 设置 Erlang Cookie

Erlang Cookie 文件：/var/lib/rabbitmq/.erlang.cookie。这里将 node1 的该文件复制到 node2。由于这个文件权限是 400，所以需要先修改 node2中的该文件权限为 777：
```
[root@rabbitmq-node2 ~]# chmod 777 /var/lib/rabbitmq/.erlang.cookie

[root@rabbitmq-node1 ~]# scp -P 26622 /var/lib/rabbitmq/.erlang.cookie root@172.17.164.18:/var/lib/rabbitmq/.erlang.cookie
```
然后将 node1 中的该文件拷贝到 node2，最后将权限和所属用户/组修改回来：
```
[root@rabbitmq-node2 ~]# chmod 400 /var/lib/rabbitmq/.erlang.cookie
 ```
##### 3.2 使用 -detached 参数运行各节点
```
[root@rabbitmq-node1 ~]#rabbitmqctl stop
[root@rabbitmq-node1 ~]# rabbitmq-server -detached
 ```
##### 3.3 组成集群

将 node2 与 node1 组成集群：
```
[root@rabbitmq-node2 ~]# rabbitmqctl stop_app
[root@rabbitmq-node2 ~]# rabbitmqctl join_cluster rabbit@node1
[root@rabbitmq-node2 ~]# rabbitmqctl start_app
 ```
如果要使用内存节点，则可以使用
```
node2 # rabbitmqctl join_cluster --ram rabbit@rabbitmq-node1 加入集群。
 ```
集群配置好后，可以在 RabbitMQ 任意节点上执行 rabbitmqctl cluster_status 来查看是否集群配置成功。
3.4 将rabbitmq设为开机自启动
```
[root@rabbitmq-node1 ~]# chkconfig --level 35 rabbitmq-server on
 ```