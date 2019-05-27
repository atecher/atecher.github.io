---
title: hexo+next+GitHub搭建静态博客(四)-添加站内搜索
date: 2016-09-27 11:33:10
categories: "Hexo教程"
tags: 
     - Hexo
---

博客写多了（汗！我写的并不多），查找是个问题，所以我想添加个站内搜索功能。NexT主题支持集成 Swiftype、 微搜索、Local Search 和 Algolia,但Swiftype和Algolia都只有一段时间的试用期。所以我采用了Hexo提供的Local Search,原理是通过hexo-generator-search插件在本地生成一个search.xml/json文件，通过这个文件实现搜索功能。

<!-- more -->
### 为hexo和next增加站内搜索功能
#### 安装插件
```
  npm install hexo-generator-search
  npm install hexo-generator-searchdb
```
#### 修改hexo配置
在你的hexo目录下的_config.yml中增加如下配置：
```
search:
  path: search.xml
  field: post
  format: html
  limit: 10000
```

配置上之后，其实搜索已经配置完成了，但现在我们还看不到搜索的入口，接下来我们需要在next的主体上进行配置

#### 配置next中的搜索入口
打开themes\next\_config.yml，打开local search:
```
# Local search
local_search:
  enable: true
  # if auto, trigger search by changing input
  # if manual, trigger search by pressing enter key or search button
  trigger: auto
  # show top n results per article, show all results by setting to -1
  top_n_per_article: 1

```

接下来就可以运行：
```
$ hexo s
```
在本地打开[http://localhost:4000/](http://localhost:4000/)进行查看了。

#### travis-ci构建搜索模块
如果你的博客是使用travis-ci自动进行构建的话，需要将上面提到的两个插件在.travis.yml中进行配置：
```
# S: Build Lifecycle
install:
  - npm install
  - npm install hexo-generator-search
  - npm install hexo-generator-searchdb
```

效果图

[![Image-1][]][Image-1]

[Image-1]: http://qn.mintools.net/mts/20180418/3853306637468672



