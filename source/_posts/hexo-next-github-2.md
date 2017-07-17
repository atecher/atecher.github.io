---
title: 使用hexo+next在GitHub上搭建自己的blog（二）--部署到github
date: 2016-02-27 12:33:10
categories: "Hexo教程"
tags: 
     - Hexo
     - Next
---
上一篇文章我们说了怎么在我们的电脑上安装和使用Hexo，这次我想介绍下怎么将hexo部署到github上，这篇文章中我将介绍两种方法。
<!-- more -->
介绍这两种方法之前，我们先将准备工作做一下。

### 1 github准备工作

#### 1.1 注册账号
如果你已经有了github账号，这一步可以忽略，注册的细节我就不多说了。
 
#### 1.2 配置

新建Repository

<img src="/img/T146hvB4WQ1RCvBVdK.jpg"  />

创建yourname.github.io，打勾表示名称可用

<img src="/img/T146hvB4WQ1RCvBVdK4.jpg" />

### 2 第一种方法

#### 2.1 生成网站


```
$ hexo generate

```
此时会将/source的.md文件生成到/public中，形成网站的静态文件。
#### 2.2 本地服务器
```
$ hexo server

```
输入http://localhost:4000 即可查看网站。

可修改：
```
$ hexo server -p 3000

```  
此时，输入http://localhost:3000 查看网站。


#### 2.3 部署网站

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


到此为止完成网站的雏形。输入yourname.github.io可访问博客主页。例如：http://atecher.github.io/ 。

部署的时候有可能会出错，别急，加这一步操作就ok了。
```
 $ npm install hexo-deployer-git --save
 
```

PS:这种方式，如果需要多台电脑之间操作blog，会很麻烦。个人建议使用第二种方式。

### 3 第二种方法

第一种方法，我们需要每次写完文章后自己进行编译静态文件并部署到github上。那么能不能借助免费的开源持续集成构建项目来完成自动编译部署呢？

能，那就是Travis CI。

#### 3.1 

 
 
 
 
 