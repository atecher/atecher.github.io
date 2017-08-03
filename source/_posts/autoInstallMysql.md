---
title: MySQL-5.5.25 centos一键安装脚本 autoInstallMysql
date: 2014-12-11 19:00:10
categories: "Mysql"
tags: 
     - Linux
     - Mysql
---

```bash
#!/bin/bash
if [ `uname -m` == "x86_64" ];then
machine=x86_64
else
machine=i686
fi

mysqlBasedir=/storage/server/mysql
mysqlDatadir=${mysqlBasedir}/data/
mysqlLogdir=/storage/log/mysql
mysqlUser=mysql
mysqlGroup=mysql

mkdir -p $mysqlBasedir
mkdir -p $mysqlDatadir
mkdir -p $mysqlLogdir

#如果mysql已安装,删除原有mysql
if [ $machine == "x86_64" ];then
  rm -rf mysql-5.6.30-linux-glibc2.5-x86_64
  if [ ! -f mysql-5.6.30-linux-glibc2.5-x86_64.tar.gz ];then
     wget http://mirrors.sohu.com/mysql/MySQL-5.6/mysql-5.6.30-linux-glibc2.5-x86_64.tar.gz
  fi
  tar -xzvf mysql-5.6.30-linux-glibc2.5-x86_64.tar.gz
  mv mysql-5.6.30-linux-glibc2.5-x86_64/* $mysqlBasedir
else
  rm -rf mysql-5.6.30-linux-glibc2.5-i686
  if [ ! -f mysql-5.6.30-linux-glibc2.5-i686.tar.gz ];then
  wget http://mirrors.sohu.com/mysql/MySQL-5.6/mysql-5.6.30-linux-glibc2.5-i686.tar.gz
  fi
  tar -xzvf mysql-5.6.30-linux-glibc2.5-i686.tar.gz
  mv mysql-5.6.30-linux-glibc2.5-i686/* $mysqlBasedir

fi

#添加mysql用户组
groupadd $mysqlGroup
#添加mysql用户 ,并制定组为mysql /sbin/nologin意思是用户不允许登录
useradd -g $mysqlGroup -s /sbin/nologin $mysqlUser
#安装服务
${mysqlBasedir}/scripts/mysql_install_db --datadir=$mysqlDatadir --basedir=$mysqlBasedir --user=$mysqlUser 

#设置权限
chown -R ${mysqlUser}:${mysqlGroup} $mysqlBasedir
chown -R ${mysqlUser}:${mysqlGroup} $mysqlDatadir
chown -R ${mysqlUser}:${mysqlGroup} $mysqlLogdir

#把mysql.server放到/etc/init.d 目录下方便使用
\cp -f ${mysqlBasedir}/support-files/mysql.server /etc/init.d/mysqld
#脚本里面的这两行在mysql启动文件指定mysql数据库的安装目录和数据目录存放目录
sed -i 's#^basedir=$#basedir='${mysqlBasedir}'#' /etc/init.d/mysqld
sed -i 's#^datadir=$#datadir='${mysqlDatadir}'#' /etc/init.d/mysqld

#配置文件
cat > /etc/my.cnf <<END
[client]
port            = 3306
socket          = /tmp/mysql.sock
[mysqld]
port            = 3306
socket          = /tmp/mysql.sock
skip-external-locking
key_buffer_size = 16M
max_allowed_packet = 1M
table_open_cache = 64
sort_buffer_size = 512K
net_buffer_length = 8K
read_buffer_size = 256K
read_rnd_buffer_size = 512K
myisam_sort_buffer_size = 8M

log-bin=mysql-bin
binlog_format=mixed
server-id       = 1

sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES

[mysqldump]
quick
max_allowed_packet = 16M

[mysql]
no-auto-rehash

[myisamchk]
key_buffer_size = 20M
sort_buffer_size = 20M
read_buffer = 2M
write_buffer = 2M

[mysqlhotcopy]
interactive-timeout
END

chmod 755 /etc/init.d/mysqld
/etc/init.d/mysqld start
#没测试过
$mysqlBasedir/bin/mysqladmin -uroot password 'hhw840129'

```