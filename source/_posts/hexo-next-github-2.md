---
title: hexo+next+GitHub搭建静态博客(二)-部署到github
date: 2016-02-27 12:33:10
categories: "Hexo教程"
tags: 
     - Hexo
     - Next
---
上一篇文章我们说了怎么在我们的电脑上安装和使用Hexo，这次我想介绍下怎么将hexo部署到github上，这篇文章中我将介绍两种方法。
<!-- more -->
介绍这两种方法之前，我们先将准备工作做一下。

### github准备工作

#### 注册账号
如果你已经有了github账号，这一步可以忽略，注册的细节我就不多说了。
 
#### 配置

新建Repository

<img src="/img/T146hvB4WQ1RCvBVdK.jpg"  />

创建yourname.github.io，打勾表示名称可用

<img src="/img/T146hvB4WQ1RCvBVdK4.jpg" />

### 第一种方法

#### 生成网站


```
$ hexo generate

```
此时会将/source的.md文件生成到/public中，形成网站的静态文件。

#### 本地服务器

```
$ hexo server

```

输入http://localhost:4000 即可查看网站。

可修改：

```
$ hexo server -p 3000

```

此时，输入http://localhost:3000 查看网站。


#### 部署网站

第一步需要在_config.yml中配置你所要部署的站点：

```
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repository: https://github.com/mamadown/mamadown.github.io.git
  branch: master
  
```

部署命令

```
$ hexo deploy

```


部署网站之前需要生成静态文件，也可以用$ hexo generate -d直接生成并部署。


到此为止完成网站的雏形。输入yourname.github.io可访问博客主页。例如：[http://atecher.github.io/](http://atecher.github.io/) 。

部署的时候有可能会出错，别急，加这一步操作就ok了。
```
 $ npm install hexo-deployer-git --save
 
```

PS:这种方式，如果需要多台电脑之间操作blog，会很麻烦。个人建议使用第二种方式。

### 第二种方法

第一种方法，我们需要每次写完文章后自己进行编译静态文件并部署到github上。那么能不能借助免费的开源持续集成构建项目来完成自动编译部署呢？

能，那就是Travis CI。

#### 什么是Travis CI

Travis CI 是目前新兴的开源持续集成构建项目，它与jenkins，Go的很明显的特别在于采用yaml格式，同时他是在在线的服务，不像jenkins需要你本地打架服务器，简洁清新独树一帜。目前大多数的github项目都已经移入到Travis CI的构建队列中，据说Travis CI每天运行超过4000次完整构建。对于做开源项目或者github的使用者，如果你的项目还没有加入Travis CI构建队列，那么我真的想对你说out了。
 
#### github代码位置

使用Travis CI需要在github上创建两个分支，一个是默认的master，还有一个是blog-source分支。

 
master：博客的静态文件，也就是hexo生成后的HTML文件，因为要使用Github Pages服务，所以他规定的网页文件必须是在master分支。
 
<img src="/img/20170717153834.png" />
 
blog-source：是博客的源代码，我们需要将hexo的代码上传到这个分支。

<img src="/img/20170717153923.png" />

我们只需要将你的blog clone到本地，在blog-source分支写blog，写完之后it push到github，然后Travis自动构建，构建完成后自动推送到Github上的master分支下。

#### 启用要构建的项目

首先如果你要使用Travis CI，你必须要GIthub账号（好像Travis CI只支持构建github的项目）和一个项目。

使用Github账号登录[Travis CI](https://travis-ci.org/)官网，如下图:

<img src="/img/20170717154918.png" />


