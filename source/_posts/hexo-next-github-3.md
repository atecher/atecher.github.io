---
title: hexo+next+GitHub搭建静态博客(三)-添加tags和categories
date: 2016-04-23 11:33:10
categories: "Hexo教程"
tags: 
     - Hexo
---

之前我们已经写了怎么安装设置Hexo和怎么部署到gitHub上，下面我们说一下怎么完善我们的blog，今天先介绍下怎么添加分类和标签页面

<!-- more -->

### 创建Tag页面
```
$ hexo new page "tags"

```

编辑刚新建的页面，将页面的类型设置为tags，主题将自动为这个页面显示标签云。页面内容如下：
```
---
title: Tagcloud
date: 2017-06-22 12:39:04
type: "tags"
---
```

注意：如果有启用多说 或者Disqus评论，默认页面也会带有评论。需要关闭的话，请添加字段 comments 并将值设置为 false，如：
```
---
title: Tagcloud
date: 2017-06-22 12:39:04
type: "tags"
comments: false
---
```
### 创建Categories页面

创建分类页和Tag页是差不多的步骤。

```
$ hexo new page "categories"

```

```
---
title: categories
date: 2017-06-22 12:39:04
type: "categories"
comments: false
---
```

### 添加菜单

在菜单中添加链接。编辑 主题配置文件 ，添加 tags 到 menu 中，如下:
```
menu:
  home: /
  categories: /categories/
  #about: /about/
  archives: /archives/
  tags: /tags/
  #sitemap: /sitemap.xml
  #commonweal: /404.html

```
