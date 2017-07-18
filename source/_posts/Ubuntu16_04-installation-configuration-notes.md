---
title: Ubuntu16.04安装配置笔记
date: 2016-12-22 12:33:10
categories: "Linux"
tags: 
     - Ubuntu
---


折腾过很多Linux的发行版，最后还是决定回归Ubuntu。给自己的台式机和笔记本都装上了Ubuntu，开始了折腾之旅。

<!-- more -->


#### 更改左上角的Slogan

新建Slogan.po，内容如下：

```
msgid "Ubuntu Desktop"msgstr "每个人都是自己梦想王国的国王"

```

```
cd /usr/share/locale/zh_CN/LC_MESSAGES

sudo msgfmt -o unity.mo /home/mark/Slogan.po

```


#### 删除libreoffice

```
sudo apt-get remove libreoffice-common

```

#### 删除Amazon的链接


```
sudo apt-get remove unity-webapps-common

```

#### 安装WPS

```
wget http://kdl.cc.ksosoft.com/wps-community/download/a20/wps-office_10.1.0.5503~a20p2_amd64.deb

chmod +x wps-office_10.1.0.5503~a20p2_amd64.deb

sudo dpkg -i wps-office_10.1.0.5503~a20p2_amd64.deb

```

#### 解决字体缺少的问题


```
##解压安装即可
http://gd.7edown.com:808/green/symbol%A3%ADfonts_all.rar

```

#### 安装Unity tweak tool

<img src="/img/1472631243749014082.jpg">


```
sudo apt install unity-tweak-tool

```

#### 安装Arc-theme主题

Source:

```
http://software.opensuse.org/download.html?project=home%3AHorst3180&package=arc-theme

```

```
sudo sh -c "echo 'deb http://download.opensuse.org/repositories/home:/Horst3180/xUbuntu_16.04/ /' >> /etc/apt/sources.list.d/arc-theme.list"
wget http://download.opensuse.org/repositories/home:Horst3180/xUbuntu_16.04/Release.key
sudo apt-key add - < Release.key
sudo apt-get update
sudo apt-get install arc-theme

```

#### 安装图标

##### Moka

<img src="/img/1472631263972063628.jpg">

```
sudo add-apt-repository ppa:snwh/moka-icon-theme-daily

sudo apt-get update

sudo apt-get install moka-icon-theme moka-icon-theme-symbolic moka-icon-theme-extras

```

##### Numix

<img src="/img/1472632116730004374.jpg">

```
sudo add-apt-repository ppa:numix/ppa

sudo apt-get update

sudo apt-get install numix-icon-theme numix-icon-theme-circle

```
##### Uniform

<img src="/img/1472632088729006934.jpg">

```
http://0rax0.deviantart.com/art/Uniform-Icon-Theme-453054609

```

##### Plateau

<img src="/img/1472632128962088713.jpg">


```
http://malysss.deviantart.com/art/Plateau-0-2-391110900

```




                       