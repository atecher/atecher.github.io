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

[![Image-1][]][Image-1]

创建yourname.github.io，打勾表示名称可用

[![Image-2][]][Image-2]

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

输入[http://localhost:4000][Link-3] 即可查看网站。

可修改：

```
$ hexo server -p 3000

```

此时，输入[http://localhost:3000][Link-4] 查看网站。


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


到此为止完成网站的雏形。输入yourname.github.io可访问博客主页。例如：[http://atecher.github.io/][Link-5] 。

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
 
[![Image-6][]][Image-6]
 
blog-source：是博客的源代码，我们需要将hexo的代码上传到这个分支。

[![Image-7][]][Image-7]

我们只需要将你的blog clone到本地，在blog-source分支写blog，写完之后it push到github，然后Travis自动构建，构建完成后自动推送到Github上的master分支下。

#### 启用要构建的项目

首先如果你要使用Travis CI，你必须要GIthub账号（好像Travis CI只支持构建github的项目）和一个项目。

使用Github账号登录[Travis CI][Link-8]官网，如下图:

[![Image-9][]][Image-9]


登录完后会进入如下界面 
 
[![Image-10][]][Image-10]

当然如果你以前没用使用过，所以你登录完是没有上图红框内的内容的，这里是因为我在写这篇博客前已经使用了，所以会有这些内容。

接下来我们点击My Repositories旁边的+，意思是添加一个要自动构建的仓库，点击后就会来到如下界面：

[![Image-11][]][Image-11]

可以看到这个界面会显示当前github账号的所以项目，如果没有显示，点击右上角的“Sync account”按钮，就可以同步过来了。

下一步肯定是要开启你需要构建的仓库，大家可以看到红框圈起来的部分，就是我开启了我的博客。

开启后我们还需要进行一些配置，操作如下：

[![Image-12][]][Image-12]

点击红框的那个菜单按钮，就会出现这样的下拉菜单，我们选择设置，来到这个界面，我们按照如下勾选：

[![Image-13][]][Image-13]

Build only if .travis.yml is present：是只有在.travis.yml文件中配置的分支改变了才构建；
Build branch updates：当推送完这个分支后开始构建；

到这一步， 我们已经开启了要构建的仓库，但是还有个问题就是，构建完后，我们怎么将生成的文件推送到github上呢，如果不能推送那我们就不需要倒腾一番来使用Travis CI服务了，我们要的结果就是，我们只要想github一push，他就自动构建并push静态文件到github pages呢，那么下面要解决的就是Travis CI怎么访问github了。

#### 在Travis CI配置Github的Access Token

标题已经说得很明白了吧，我们需要在Travis上配置Access Token，这样我们就可以在他构建完后自动push到github pages了。

##### 在github上生成Access Token

首先我们来到github的设置界面，点击到Personal access tokens页面，点击右上角的Generate new token按钮会重新生成一个，点击后他会叫你输入密码，然后来到如下界面，给他去一个名字，下面是勾选一些权限。

[![Image-14][]][Image-14]

生成完后，你需要拷贝下来，只有这时候他才显示，下载进来为了安全他就不会显示了，如果忘了只能重新生成一个了，拷贝完以后我们需要到Travis CI网站配置下

###### 在Travis CI配置

配置界面还是在项目的setting里面，如下图：

[![Image-15][]][Image-15]

至于为什么我们要在这里配置，我想大家肯定应该明白了，写在程序里不安全，配置到这里相当于一个环境变量，我们在构建的时候就可以引用他。 
到这里我已经配置了要构建的仓库和要访问的Token，但是问题来了，他知道怎么构建，怎么生成静态文件吗，怎么push的github pages，又push到那个仓库吗，所以这里我们还需要在源代码的仓库里创建一个.travis.yml配置文件，放到源代码的根目录，如下图：

[![Image-16][]][Image-16]

文件内容：

```
language: node_js
node_js: stable

# S: Build Lifecycle
install:
  - npm install


#before_script:
 # - npm install -g gulp

script:
  - hexo g

after_script:
  - cd ./public
  - git init
  - git config user.name "your nickname"
  - git config user.email "your email"
  - git add .
  - git commit -m "Update docs"
  - git push --force --quiet "https://${GH_TOKEN}@${GH_REF}" master:master
# E: Build LifeCycle

branches:
  only:
    - blog-source
env:
 global:
   - GH_REF: github.com/atecher/atecher.github.io.git

```

记得配置一下你的昵称、邮箱和你的git地址(GH_REF)。

到这一步，我们可以写一篇文章，添加到你的博客的_posts目录下，然后commit并push到你的Github上。

如果不出意外，我们可以就可以在Travis CI网站看到他已经在构建了，如下图：

[![Image-17][]][Image-17]

构建完成后，我们去blog上就能看到这篇文章了。

[Image-1]: //qn.atecher.com/mts/20180418/3853306650002432
[Image-2]: //qn.atecher.com/mts/20180418/3853306656179200
[Link-3]: http://localhost:4000
[Link-4]: http://localhost:3000
[Link-5]: http://atecher.github.io/
[Image-6]: //qn.atecher.com/mts/20180418/3853306661143552
[Image-7]: //qn.atecher.com/mts/20180418/3853306661618688
[Link-8]: https://travis-ci.org/
[Image-9]: //qn.atecher.com/mts/20180418/3853306665387008
[Image-10]: //qn.atecher.com/mts/20180418/3853306665468928
[Image-11]: //qn.atecher.com/mts/20180418/3853306667451392
[Image-12]: //qn.atecher.com/mts/20180418/3853306667877377
[Image-13]: //qn.atecher.com/mts/20180418/3853306667877376
[Image-14]: //qn.atecher.com/mts/20180418/3853306664092672
[Image-15]: //qn.atecher.com/mts/20180418/3853306634912768
[Image-16]: //qn.atecher.com/mts/20180418/3853306634945536
[Image-17]: //qn.atecher.com/mts/20180418/3853306635273216