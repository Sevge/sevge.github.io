---
title: 阿里云轻量应用服务器下部署Graylog2
date: 2018-04-21 21:00:27
categories:
- 阿里云
---
## 前言
前面有折腾过在树莓派上部署Graylog，受限于树莓派的性能，最终还是放弃用树莓派来干这么重的活了。
正好同学买了一台轻量级应用服务器(Ubuntu 16.04 1核2G)闲置着，本着不折腾不舒服的原则，我又借过来玩一玩。

## 步骤
前人有写过[教程](https://www.aliyun.com/jiaocheng/118555.html?spm=5176.100033.2.5.hkYTKX)
安装JAVA JDK、Elasticsearch、Mongodb，配置好Graylog，一路顺畅，Graylog顺利启动。
服务器本地测试没有问题，在我的浏览器中可以打开页面
本地测试
```
curl -X GET http://localhost:9000 
```
浏览器测试
```
http://公网IP:9000
```
一切顺畅，真是简单啊......
但是，在我的浏览器Graylog显示加载完成后，报出服务不可用的错误。

## 踩的坑
本以为之前在树莓派上踩过的坑够多，在这云服务器上搭建只是三五分钟的事情。
但是这个问题很是匪夷所思，服务器本地测试通过，按道理服务应该是成功启动了。
先是检查服务器防火墙问题，然后看阿里云控制台那边，试着将所有端口打开。
```
getenforce
iptables -L -n
```
无果......
又试着修改Graylog中`rest_listen_uri`和`web_listen_uri`的端口和地址。
无果......
自己不能解决，最后谷歌在官方论坛找到[答案](https://community.graylog.org/t/server-currently-unavailable-error-is-coming-while-trying-to-access-from-browser/503/3)

> okay I seem to have it working.
Set the rest_listen_uri to the private ip on the host that the public ip NATs to.
set the web_listen_uri to the private ip on the host that the public ip NATs to
set the web_endpoint_uri to http://public 26 IP :9000/api/ ( this is basically the same as the rest_listen_uri but with the public ip instead of the private IP )


在Graylog配置文件中，将` rest_listen_uri `和` web_listen_uri ` 这两项设置为内网IP地址，将注释掉的内容` web_endpoint_uri `设置为公网IP地址，完成后重启服务。

## 解决
完美解决！
果然网上的老哥个个都是人才，说的话好听又管用！