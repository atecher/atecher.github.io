---
title: 搭建HAProxy+Keepalived 高可用负载均衡
date: 2016-01-11 14:33:10
categories: "高可用"
tags: 
     - HAProxy
     - Keepalived
---

### HAProxy

HAProxy是一款提供高可用性、负载均衡以及基于TCP（第四层）和HTTP（第七层）应用的代理软件（PS:nginx最新版也可以基于第四层和第七层的负载均衡）。
HAProxy和Keepalived 都采用源码方式安装，如果没有gcc编译器，需要先安装gcc编译工具。
<!-- more -->
#### 下载解压安装
下载haproxy：http://www.haproxy.org/download/1.4/src/

```
tar zxvf haproxy-1.4.24.tar.gz
cd haproxy-1.4.24
make install
mkdir -p /usr/local/haproxy/etc
mkdir -p /usr/local/haproxy/sbin
cp examples/haproxy.cfg /usr/local/haproxy/etc
ln -s /usr/local/sbin/haproxy /usr/local/haproxy/sbin/haproxy
```
#### 修改配置文件
`vim /usr/local/haproxy/etc/haproxy.cfg`

```powershell
global 
    maxconn 51200 #最大连接数
    chroot /usr/local/haproxy #改变当前工作目录
    uid 99
    gid 99
    daemon #后台方式运行
    #quiet
    nbproc 1  #并发进程数
    pidfile /usr/local/haproxy/logs/haproxy.pid     #定义haproxy的pid


defaults  #默认部分的定义
        mode http #mode {http|tcp|health} 。
			      #http是七层模式，tcp是四层模式，health是健康检测返回OK
        #retries 2
        option redispatch
        option abortonclose
        timeout connect 5000ms   #连接超时
        timeout client 30000ms   #客户端超时
        timeout server 30000ms   #服务器超时
        #timeout check 2000      #心跳检测超时
        log 127.0.0.1 local0 err #[err warning info debug]
			#使用本机syslog服务的local3设备记录错误信息
        balance roundrobin

	# option httplog
	# option httpclose
	# option dontlognull
	# option forwardfor

 
listen admin_stats
		#定义一个名为status的部分，可以在listen指令指定的区域中定义匹配规则和后端服务器ip，
        bind 0.0.0.0:8888   # 定义监听的套接字
        option httplog
        stats refresh 30s   #统计页面的刷新间隔为30s
        stats uri /stats     #登陆统计页面是的uri
        stats realm Haproxy Manager
        stats auth admin:admin #登陆统计页面是的用户名和密码
        #stats hide-version  # 隐藏统计页面上的haproxy版本信息

listen tcp_test
        bind :12345
        mode tcp
        server t1 127.0.0.1:9000
        server t2 192.168.15.13:9000
        
listen zzs_dzfp_proxy:90
		mode http
		balance roundrobin   #轮询
        cookie LBN insert indirect nocache   
        option httpclose   
        server web01 192.168.15.12:9000 check inter 2000 fall 3 weight 20  
        server web02 192.168.15.13:9000 check inter 2000 fall 3 weight 20
```
#### 启停haproxy
启动Haproxy
```
/usr/local/haproxy/sbin/haproxy -f /usr/local/haproxy/etc/haproxy.cfg
```
停止Haproxy：
```
killall haproxy 
```
访问统计页面：
```
http://10.10.3.163:1080/stats
```

### Keepalived

keepalived是集群管理中保证集群高可用的一个服务软件，其功能类似于heartbeat，用来防止单点故障。

keepalived是以VRRP协议为实现基础的，VRRP全称Virtual Router Redundancy Protocol，即虚拟路由冗余协议。

虚拟路由冗余协议，可以认为是实现路由器高可用的协议，即将N台提供相同功能的路由器组成一个路由器组，这个组里面有一个master和多个backup，master上面有一个对外提供服务的vip（该路由器所在局域网内其他机器的默认路由为该vip），master会发组播，当backup收不到vrrp包时就认为master宕掉了，这时就需要根据VRRP的优先级来选举一个backup当master。这样的话就可以保证路由器的高可用了。


#### 下载解压编译安装
下载地址：http://www.keepalived.org/download.html
 或者 ：
```
wget http://www.keepalived.org/software/keepalived-1.2.8.tar.gz
```
安装：
```
tar zxvf keepalived-1.2.8.tar.gz
cd keepalived-1.2.8
./configure --prefix=/usr/local/keepalived
make
make install
```
#### 安装成功后做成服务模式。
```
cp /usr/local/keepalived/sbin/keepalived /usr/sbin/
cp /usr/local/keepalived/etc/sysconfig/keepalived /etc/sysconfig/
cp /usr/local/keepalived/etc/rc.d/init.d/keepalived /etc/init.d/
```

```
mkdir -p /etc/keepalived/
cp /usr/local/keepalived/etc/keepalived/keepalived.conf /etc/keepalived/keepalived.conf
chmod +x /etc/init.d/keepalived
vi /etc/keepalived/keepalived.conf
```
#### 修改文件配置`keepalived.conf`
```powershell
global_defs {
    router_id LVS_DEVEL
}
#监测haproxy进程状态，健康检查，每2秒执行一次 
vrrp_script chk_haproxy {
    script "/etc/keepalived/chk_haproxy.sh" #监控脚本
    interval 2   #每两秒进行一次
    weight -10    #如果script中的指令执行失败，vrrp_instance的优先级会减少10个点
}
vrrp_instance VI_1 {
    state MASTER          #主服务器MASTER，从服务器为BACKUP
    interface eth0        #服务器固有IP（非VIP）的网卡
    virtual_router_id 51  #取值在0-255之间，用来区分多个instance的VRRP组播，同一网段中virtual_router_id的值不能重复，否则会出错。
    priority 100          #用来选举master的，要成为master，那么这个选项的值最好高于其他机器50个点。此时，从服务器要低于100；
    advert_int 1          #健康查检时间间隔
    mcast_src_ip 192.168.15.12    #MASTER服务器IP,从服务器写从服务器的IP
    authentication { #认证区域
        auth_type PASS  #推荐使用PASS（密码只识别前8位）
        auth_pass 12345678
    }
    track_script {
        chk_haproxy    #监测haproxy进程状态
    }
    virtual_ipaddress {
        192.168.15.235   #虚拟IP，作IP漂移使用
    }
}

```
#### 监控脚本配置

`vi /etc/keepalived/chk_haproxy.sh`

```
tatus=$(ps aux|grep haproxy | grep -v grep | grep -v bash | wc -l)
if [ "${status}" = "0" ]; then
    /usr/local/haproxy/sbin/haproxy -f /usr/local/haproxy/etc/haproxy.cfg

    status2=$(ps aux|grep haproxy | grep -v grep | grep -v bash |wc -l)

    if [ "${status2}" = "0"  ]; then
            /etc/init.d/keepalived stop
    fi
fi
```
这个配置文件意思：检查haproxy是否挂掉，如果挂掉启动haproxy；若启动之后还是没有检测到启动状态，则关闭keepalived，让IP飘移到备机上。

#### 启动停止keepalived命令

```
service keepalived start #启动
service keepalived stop	#关闭服务
```