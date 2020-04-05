---
title: 利用frp实现内网穿透
date: 2019-03-17 17:35:30
categories:
- 网络
---
## 简介
　　人在学校外，想要查学分？想要查成绩？想要上知网？很多时候不想麻烦别人，这个时候就需要自己折腾折腾了。。。
　　之前有了解过内网穿透的方式，可以使用ngrok、natapp、autossh，但是我想要的是外网机器最终达到如在内网一样的环境，最终选定了frp。

## 关于frp
　　引一段官网的话：
> frp是一个高性能的反向代理应用，可以帮助您轻松地进行内网穿透，对外网提供服务，支持 tcp, http, https 等协议类型，并且 web 服务支持根据域名进行路由转发。

　　我的理解如图：
![frp](https://s2.ax1x.com/2019/03/17/Aed5ex.jpg)

## 准备
- 内网机器一台(10.8.*.*)
- 外网机器一台(47.*.*.*)
　　
　　首先在内网机器(10.8.*.*)上搭建好softether服务，可以参照[这里](https://blog.csdn.net/qq_35422558/article/details/78018089)，搭建完成，创建登陆用户、开启虚拟路由器和虚拟DHCP、启用Openvpn克隆server功能，这里我选择UDP类型的2333端口。
![openvpn](https://s2.ax1x.com/2019/03/17/AeNjO0.jpg)
　　点击‘为Openvpn Client生成配置样本文件’，保存到本地，修改remote地址为公网服务器地址：
![config](https://s2.ax1x.com/2019/03/17/Aea8C4.jpg)


## 开始使用
### 外网机器(47.*.*.*)
　　外网机器作为frp server端，点击[这里](http://diannaobos.iok.la:81/frp/frp-v0.20.0/frp_0.20.0_linux_amd64.tar.gz)下载tar包。wget下来，tar解压，编辑配置文件，如我的frps.ini：
```
[common]
bind_addr = 0.0.0.0
bind_port = 7000
auto_token = 123456

dashboard_port = 7500
dashboard_user = admin
dashboard_pwd = admin
```

[common]部分是必须有的配置，其中bind_port是自己设定的frp服务端端口，auto_token必须与frpc端配置一致。后三行启用dashboard，可以选择配置。
最终启动： `nohup ./frps -c ./frps.ini &`

### 内网机器(10.8.*.*)
　　内网机器作为frp client端，tar包同上。编辑配置文件，如我的frpc.ini：
```
[common]
server_addr = 47.*.*.*
server_port = 7000
auto_token = 123456

[ssh]
type = tcp
local_ip = 127.0.0.1 # ssh服务的地址
local_port = 22  # ssh服务的端口
remote_port = 2277 # 最终公网对外开放的ssh端口

[p2p]
type = tcp
local_ip = 127.0.0.1
local_port = 5555
remote_port = 5555

[openvp]
type = udp
local_ip = 127.0.0.1
local_port = 2333
remote_port = 2333
```
　　保存配置，运行frp客户端：`nohup ./frpc -c ./frpc.ini &`

## 完成
　　Done！
　　之前试用softether客户端，可以点击连接，但是在分配ip地址时出现问题，而openvpn客户端可以正常连接。
