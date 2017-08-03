---
title: debian安装配置笔记
date: 2015-03-21 12:33:10
categories: "Linux"
tags: 
     - Debian
     - Linux
---

#### 将新建的用户增加到sudoers中
```bash
vim /etc/sudoers
#修改如下
# User privilege specification
root    ALL=(ALL:ALL) ALL
mark    ALL=(ALL:ALL) ALL
# Allow members of group sudo to execute any command
%sudo   ALL=(ALL:ALL) ALL
```

<!-- more -->

#### Debian安装vim并设置代码高亮
##### 安装
```bash
apt-get install vim

```
##### 设置
```
vim /etc/vim/vimrc

```
去掉syntax on的注释
```bash
" Vim5 and later versions support syntax highlighting. Uncommenting the next
" line enables syntax highlighting by default.
syntax on

" If using a dark background within the editing area and syntax highlighting
" turn on this option as well

```

#### 切换登录管理器
```bash
dpkg-reconfigure

```

#### VirtualBox下Debian安装增强功能
打开一个root终端：
```
apt-get install build-essential
```
然后 设备 安装增强功能 提示不能自动运行 不必担心 执行下一步：
```
 sh /media/cdrom0/VBoxLinuxAdditions.run
```
按提示操作即可。
终于可以想拖久拖，想黏贴就黏贴了~~

#### 为debian8 安装Arc-theme主题
```
url：http://software.opensuse.org/download.html?project=home%3AHorst3180&package=arc-theme
echo 'deb http://download.opensuse.org/repositories/home:/Horst3180/Debian_8.0/ /' >> /etc/apt/sources.list.d/arc-theme.list
wget http://download.opensuse.org/repositories/home:Horst3180/Debian_8.0/Release.key
apt-key add - < Release.key  
apt-get update
apt-get install arc-theme
```
#### 安装图标

下载url:https://github.com/captiva-project/captiva-icon-theme

解压后放到/usr/share/icons/中

#### 安装Mysql
```bash
sudo apt-get install mysql-server
###安装完成后
vim /etc/mysql/my.cnf
##注释掉
#bind-address           = 127.0.0.1

sudo systemctl restart mysql.service

更新mysql中root用户可以被远程访问

update user set host='%' where host='atecher' and user='root';
flush privileges;
```