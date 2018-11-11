---
title: Java_Excption_URISyntaxException的解决办法
date: 2017-08-14 11:00:00
categories: Java
tags:
    - Java
    - Exception
---


近日在用HttpClient访问抓取汇率时，为了省力，直接采用
String url = "http://api.liqwei.com/currency/?exchange=usd|cny&count=1";
HttpClient client    = new DefaultHttpClient();
HttpGet httpget = new HttpGet(url);
HttpResponse response = client.execute(httpget);
 
以前用这种方法都没有问题，但这次却报如下错误：
java.net.URISyntaxException: Illegal character in query at index 44
 
查找了一些网上资料，说地址中涉及了特殊字符，如‘｜’‘&’等。所以不能直接用String代替URI来访问。必须采用%0xXX方式来替代特殊字符。但这种办法不直观。所以只能先把String转成URL，再能过URL生成URI的方法来解决问题。代码如下
URL url = new URL(strUrl);
URI uri = new URI(url.getProtocol(), url.getHost(), url.getPath(), url.getQuery(), null);
HttpClient client    = new DefaultHttpClient();
HttpGet httpget = new HttpGet(uri);

ref:
[http://qsfwy.iteye.com/blog/1926302](http://qsfwy.iteye.com/blog/1926302)

<!-- more -->

