---
title: hexo+next+GitHub搭建静态博客(一)
date: 2016-02-22 12:33:10
categories: "Hexo教程"
tags: 
     - Hexo
     - Next
---


之前都是在公共的博客平台写东西，突然想自己搭建blog了。我也有自己的阿里云服务器，但一是搭载麻烦，二是数据保存是个问题。最后瞄上了Github Pages,网上也查了些资料，决定使用hexo+next来搭建自己的blog。

<!-- more -->

### 安装Git和NodeJS环境
因为hexo需要依赖Git和NodeJs，所以需要先安装环境。
```
Git下载地址：https://git-scm.com/download/win
NodeJS下载地址：https://nodejs.org/download/

```
至于安装过程我就不多写了。 

### 安装hexo

```
#安装hexo服务
$ npm install -g hexo-cli
#初始化hexo
$ hexo init <your-hexo-site>
$ cd <your-hexo-site>
$ npm install
 
```

### 使用Next主题
下面来使用Next主题让站点更酷炫，当然hexo有很多主题，操作方法都类似。

#### 安装
```
$ cd <your-hexo-site>
$ git clone https://github.com/iissnan/hexo-theme-next themes/next

```
#### 配置

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

 
 
 
 
 ### 写文章
 
 ```
 $ hexo new "Hello World"
 
 $ hexo s --debug
 
 ```
 
 访问[http://localhost:4000](http://localhost:4000)，确保站点正确运行。
 
 
 
 
 