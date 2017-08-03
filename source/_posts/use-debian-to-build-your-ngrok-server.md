---
title: 搭建自己的ngrok服务器-debian版
date: 2016-04-02 12:13:10
categories: "服务"
tags:
     - Debian
     - Ngrok
---


作为一个Web开发者，我们有时候会需要临时地将一个本地的Web网站部署到外网，以供他人体验评价或协助调试等等，通常我们会这么做：

1. 找到一台运行于外网的Web服务器。
2. 服务器上有网站所需要的环境，否则自行搭建
3. 将网站部署到服务器上
4. 调试结束后，再将网站从服务器上删除

只不过是想向朋友展示一下网站而已，要不要这么麻烦，累感不爱。

<!-- more -->

### 安装go lang环境

```bash
wget http://www.golangtc.com/static/go/1.7.3/go1.7.3.linux-amd64.tar.gz

```

常见的不同版本根据下方来匹配(可以到这里下载)：

```
mac： darwin-amd64
ubuntu： linux-amd64
centos： linux-386

```

或者使用命令安装：

```
apt-get install golang-go

```

### 安装git

```
apt-get install git

```
### git clone ngrok源码，编译

ngrok源码：https://github.com/inconshreveable/ngrok.git

#### 进入/usr/local目录

```
git clone https://github.com/inconshreveable/ngrok.git

```
#### 引入临时的全局环境变量，此次登录有效

```
# 这个等会编译的时候要用
export GOPATH=/usr/local/ngrok/
# 这个是你自己的域名，可以是二级或三级域名
# 注意，这边ngrok.gabin.top和它的所有子域名都必须指向中转服务器，我最开始就是没有注意这点，导致各种没报错，但是就是不能用
export NGROK_DOMAIN="atecher.net"
```
#### 替换域名证书，注意到了吗，NGROK_DOMAIN这个环境变量是我们刚刚设置的。
```
#生成证书
cd /usr/local/ngrok
openssl genrsa -out rootCA.key 2048
openssl req -x509 -new -nodes -key rootCA.key -subj "/CN=$NGROK_DOMAIN" -days 5000 -out rootCA.pem
openssl genrsa -out server.key 2048
openssl req -new -key server.key -subj "/CN=$NGROK_DOMAIN" -out server.csr
openssl x509 -req -in server.csr -CA rootCA.pem -CAkey rootCA.key -CAcreateserial -out server.crt -days 5000

#替换证书
cp rootCA.pem assets/client/tls/ngrokroot.crt
cp server.crt assets/server/tls/snakeoil.crt
cp server.key assets/server/tls/snakeoil.key

```
#### 开始生成服务端执行文件
```
# 自己注意下，不同操作系统“GOARCH”是不一样的参数，上面也有写到了
GOOS=linux GOARCH=amd64 make release-server
```
成功之后在/usr/local/ngrok/bin目录下会生成一个ngrokd的文件，这就是服务端的启动执行文件了

#### 生成客户端可执行文件
```
#--mac
cd /usr/local/ngrok
GOOS=darwin GOARCH=amd64 make release-client
#--window
cd /usr/local/ngrok
GOOS=windows GOARCH=amd64 make release-client
#成功之后在/usr/local/ngrok/bin目录下会生成对应的目录，一般是darwin_amd64和window_64，前一个是mac的，后一个是window的
```
#### 替换掉引用（国内被墙了，没法用）
```
vim /usr/local/ngrok/src/ngrok/log/logger.go
# 替换掉import中log的引用，记得删除旧的，别注释了，会报错哈哈
log "github.com/keepeye/log4go"
```
#### 调试
--启动服务端，这边使用的是80端口。一般都需要用这个，原本想用反向代理，发现好像是不行。如果有发现可以的朋友，可以分享一下。
如果需要在后台执行的话，参考nohup命令
```
# 由于NGROK_DOMAIN是临时的环境变量，所以如果要重复使用的话，这个变量最好保存起来，否则下次登录就失效了。
/usr/local/ngrok/bin/ngrokd -domain="$NGROK_DOMAIN" -httpAddr=":80"
```
--启动客户端
先设置好配置文件：同目录下创建一个ngrok.cfg
```
server_addr: "blog.atecher.net:4443"
trust_host_root_certs: false
# 通过配置文件启动，这边的端口代表的是自己本地调试程序启用的端口，一般是8080
./ngrok -config=./ngrok.cfg -subdomain=blog 80
```
好了，可以用了。访问以下 blog.atecher.net

PS：

1. 其实主要就是装好go环境，如果想学习新的程序语言，可以考虑下这个最近正火的语言
2. 需要知晓基础的一些知识：环境变量、证书、make（虽然我也不是很懂，总之做多了会有点感觉，就感觉这么做是对的...）
3. 如果没有测试环境可以用的话，可以购买特价的国际域名，一般一年不要十几二十块的，然后申请个像是华为企业云的服务器（本人就申请了一个1块钱15天的试用服务），就可以自己动手尝试了
4. 其实自己没有服务器资源的可以使用国人分享出来的，百度搜索一下，最近看着还蛮多的