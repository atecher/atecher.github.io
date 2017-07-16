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

### 1 安装Git和NodeJS环境
因为hexo需要依赖Git和NodeJs，所以需要先安装环境。
```
Git下载地址：https://git-scm.com/download/win
NodeJS下载地址：https://nodejs.org/download/

```
至于安装过程我就不多写了。 

### 2 安装hexo

```
#安装hexo服务
$ npm install -g hexo-cli
#初始化hexo
$ hexo init <your-hexo-site>
$ cd <your-hexo-site>
$ npm install
 
```

### 3 使用Next主题
下面来使用Next主题让站点更酷炫，当然hexo有很多主题，操作方法都类似。

#### 3.1 安装
```
$ cd <your-hexo-site>
$ git clone https://github.com/iissnan/hexo-theme-next themes/next

```
#### 3.2 配置

修改<your-hexo-site>/_config.yml中的blog的主题:

```

# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: next

```

修改<your-hexo-site>/themes/next/_config.yml中的scheme，我使用的是Muse。

```
# ---------------------------------------------------------------
# Scheme Settings
# ---------------------------------------------------------------

# Schemes
scheme: Muse
#scheme: Mist
#scheme: Pisces

```

 
 
 
 
 ### 4 写文章
 
 ```
 $ hexo new "Hello World"
 
 $ hexo s --debug
 
 ```
 
 访问[http://localhost:4000](http://localhost:4000)，确保站点正确运行。
 
 
 
 
 