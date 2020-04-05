---
title: Wi-FireMan的总结与传承
date: 2018-10-06 16:06:57
categories:
- 网络
---
### 作品简介
　　该作品主体是运行在无线网络环境下用于侦测自身WIFI并分析安全状况的一套系统。
　　它可以监测收集周围无线环境内无线路由设备的SSID、MAC地址、信道、频率、信号强度等信息。根据收集到的这些信息进行统计和分析，判断所在Wi-Fi的安全程度。对于有威胁的事件配置警告措施若检测到某种攻击时，能发出滴滴警报声，并同时发送攻击项目数据报告邮件给用户。配合日志管理系统，通过WEB端，利用图形化界面方便直观地查看收集到的数据内容。

### 相关名词
#### 站点STA（Station）
　　所谓的站点，是指具有WIFI通信功能的，并且连接到无线网络中的终端设备，如手机、平板电脑、笔记本电脑等。

#### 接入点AP（Access Point）
　　就是我们平常所说的WIFI热点，更通俗一点，就是我们家里的无线路由器。那么它的作用是什么呢？当我们需要从互联网上获取数据到手机上显示时，那么接入点就相当于一个转发器，将互联网上其他服务器上的数据转发给我们的手机上，当然这只是一个粗略的说法。同时，接入点也属于站点的一种。

#### 服务集识别码SSID（Service Set IDentifier）
　　当我们去到一个新地方的时候，开口第一句就是：“请问WIFI账号和密码是多少？”，这里的WIFI账号就是SSID。SSID是通过接入点广播出来了。同时，我们在设置无线路由器时，可修改SSID的名称。

#### 身份认证（Authentication）
　　实体安全防护在有线局域网络安全解决方案中是不可或缺的一部分。网络和连接点（attachment point）受到限制，通常只有位于外围访问控制设备（perimeter access control device）之后的办公区才能加以访问。网络设备可以通过加锁的配线柜（locked wiring closet）加以保护，而办公室与隔间的网络插座只在必要时才连接至网络。无线网络无法提供相同层次的实体保护，因此必须依赖额外的身份认证程序，以保证访问网络的用户已获得授权。身份认证是关联的必要前提，唯有经过身份认证的用户才允许使用网络。     　　站点与无线网络连接的过程中，可能必须经过多次身份认证。关联之前，站点会先以本身的MAC地址来跟基站进行基本的身份认证。此时的身份认证，通常称为802.11身份认证，有别于后续所进行、牢靠而经过加密的用户身份认证。

#### 解除认证（Deauthentication）
　　解除认证用来终结一段认证关系。因为获准使用网络之前必须经过身份认证，解除认证的副作用就是终止目前的关联。在强健安全网络中，解除认证也会清除密钥信息。

### WiFi接入原理
　　现在常见的WiFi基本上都带有一个锁头，即它本身是经过了加密，需要经过认证后才能连接上它。一次连接大致需要三个步骤：扫描（Scanning）、认证（Authentication）、关联（Association）。
![WiFi接入过程](http://imglf3.nosdn0.126.net/img/c09lVS9TR3YrUGFMZGVTbGw4ZE9HV2dwRGJzWnM3aE9kOENkRHEvN1hDaXRyZWtqMUk2SVJnPT0.jpg?imageView&thumbnail=500x0&quality=96&stripmeta=0&type=jpg)
#### 扫描
　　要加入一个无线网络，首先我们需要找到它的网络名称，即SSID。这个SSID其实是接入点（Access Point）回应工作站扫描时所带的参数，还有其它的网络参数，包括BSSID（可理解为接入点的MAC地址）、信号强度、加密和认证方式等。
　　扫描类型分两种，一种是主动扫描（active　scanning），另一种是被动扫描（passivescanning）。

##### 主动扫描（activescanning）
　　即我们的手机（工作站STA）以主动的方式，在每个信道上发出Probe Request帧，请求某个特定无线网络予以回应。主动扫描是主动寻找网络，而不是静候无线网络声明本身的存在。
![主动扫描](http://imglf6.nosdn0.126.net/img/c09lVS9TR3YrUGFMZGVTbGw4ZE9HZVlFYSt4eThubXhIOFlhTDRrVjZPSmNkcllzbHdRenRnPT0.jpg?imageView&thumbnail=500x0&quality=96&stripmeta=0&type=jpg)
##### 被动扫描（passivescanning）
　　现在大部分移动电子产品都是采用被动扫描（passive　scanning）的方式，原因是扫描过程中不需要传送任何信号，可以省电。在被动扫描中，工作站会在信道列表（channel　list）所列的各个信道之间不断切换，并静候Beacon帧的到来。所收到的任何帧都会被暂存起来，以便取出传送这些帧的BSS 的相关数据。
![被动扫描](http://imglf4.nosdn0.126.net/img/c09lVS9TR3YrUGFMZGVTbGw4ZE9HVzI2anJvMTVDVmtaTTkwTTNLTk1RMkRTNUhYbVRxVU9nPT0.jpg?imageView&thumbnail=500x0&quality=96&stripmeta=0&type=jpg)

#### 认证
　　由于无线网络的最大缺陷就是安全性，进行身份认证是必须的，同时认证的连接工作也必须予以加密，以防未经授权的使用者访问。
　　扫描完成后，我们会选择想要加入的WIFI热点。此时，加密后的网络会弹出一个输入密码的提示框。这个过程叫做：认证（Authentication）。
早期的IEEE802.11定义了两种认证方式：
* 开放系统认证（OpenSystem authentication），
* 共享密钥认证（SharedKey authentication）。

　　开放系统认证是IEEE802.11默认的认证方式，实质上并没有做认证。连接无线网络时，基站并没有验证工作站的真实身份。认证过程由以下两个步骤组成：第一，工作站发送身份声明和认证请求；第二，基站应答认证结果，如果返回的结果是“successful”，表示两者已相互认证成功。共享密钥认证依赖于WEP（Wired Equivalent Privacy，有线等效加密）机制，这种加密机制非常不安全，可以使用Kali上集成的Aircrack-ng轻松破解出密码，因此WEP渐渐被淘汰。
　　WPA（Wi-Fi Protected Access）是WIFI联盟制定的安全性标准，WPA2是第二个版本。PSK（PreShared Key）叫做预共享密钥。经过这种加密方式后，无线网络中的数据包几乎是不可能解密的了。

#### 关联
　　认证完成后，下一步就是关联（Association）。工作站与基站进行关联，以便获得网络的完全访问权。一旦移动式工作站与基站完成认证，便可送出关联请求（Association Request）帧。尚未经过身份认证的工作站，会在基站的答复中收到一个解除关联（Deauthenticaton）帧。一旦关联请求获准，基站就会以代表成功的状态代码0及关联识别码（Association ID，简称 AID）来回应。关联请求如果失败，就只会返回状态码，并且中止整个过程。
　　最后，基站开始为移动式工作站处理帧。在常见的产品中，所使用的分布式系统媒介通常是Ethernet。在交换式Ethernet 里，该工作站的MAC地址得以跟某个特定的交换端口（Switch Port）形成关联。
![关联过程](http://imglf5.nosdn0.126.net/img/c09lVS9TR3YrUGFMZGVTbGw4ZE9HZFJGTTBPT3ZPWnFZQ0NYUnhpTmtYMUJUSEV5RUlnTVd3PT0.jpg?imageView&thumbnail=500x0&quality=96&stripmeta=0&type=jpg)

　　以上，就是一次WiFi连接的整个过程，在这一整套过程都通过以后，用户即获得该无线网络的完全访问权限。

### 系统架构
在了解以上知识后，介绍一下我们的作品所需要的设备以及整体的架构：
　　作为微型计算机的树莓派，提供多个接口，外接保持监控模式的无线网卡是最适合的。此外，树莓派成本低，功耗低，适合7*24工作以持续地在自己的WiFi中巡逻。我们使用云服务器是考虑到终端用户达到一定数量，它们收集的日志都能直接通过云服务器来处理，以简化流程。开源的日志管理系统Graylog，部署维护简单、内置简单的告警、提供简单的聚合统计功能、提供友好的UI，我们在此基础上汉化了相关内容，优化了界面，提升了它的易用性。
![arch](http://imglf3.nosdn0.126.net/img/c09lVS9TR3YrUFlGTTVtbnVobUpvUm5mV0RUUDdRMzNST2hrZ2F0MG1XRXNtZXhGbmtuR1d3PT0.jpg?imageView&thumbnail=500x0&quality=96&stripmeta=0&type=jpg)

### 安全问题和解决思路
　　由于WIFI网络具有移动性，同时WIFI以无线电波作为传输媒介，这种媒介本质上是开放的，且容易被拦截，因此它是整个网络系统中最为薄弱的一环，常常受到各种攻击。这里以MDK3为例，展示常见的WiFi标准下的攻击以及所带来的影响。
#### Beacon Flood
　　这个模式可以产生大量死亡SSID来充斥无线客户端的无线列表，从而扰乱无线使用者；我们甚至还可以自定义发送死亡SSID的BSSID和ESSID、加密方式（如wep/wpa2）等。 
常用命令：
```
mdk3 mon0 b 
      -n <ssid>        #自定义ESSID
      -f <filename>            #读取ESSID列表文件
      -v <filename>           #自定义ESSID和BSSID对应列表文件
      -d         #自定义为Ad-Hoc模式
      -w         #自定义为wep模式
      -g           #54Mbit模式
      -t            # WPA TKIP encryption
      -a           #WPA AES encryption
      -m          #读取数据库的mac地址
       -c <chan>                   #自定义信道
       -s <pps>                #发包速率mdk3 --help b  #查看详细内容
```
检测效果：
![beacon flood](http://imglf5.nosdn0.126.net/img/c09lVS9TR3YrUGFMZGVTbGw4ZE9HV1NDbGtmekFqdGx0UFJncmFONW8xenFLRkRRNWFlemNRPT0.jpg?imageView&thumbnail=500x0&quality=96&stripmeta=0&type=jpg)

#### Authentication DoS
　　这是一种验证请求攻击模式：在这个模式里，软件自动模拟随机产生的mac向目标AP发起大量验证请求，可以导致AP忙于处理过多的请求而停止对正常连接客户端的响应；这个模式常见的使用是在reaver穷据路由PIN码，当遇到AP被“pin死”时，可以用这个模式来直接让AP停止正常响应，迫使AP主人重启路由！ 
常用命令：
```
mdk3 mon0 a
      -a <ap_mac>              #测试指定BSSID
      -m              #使用有效数据库中的客户端mac地址
      -c          #对应 -a ，不检查是否测试成功
      -i  <ap_mac>           #对指定BSSID进行智能攻击
      -s <pps>               #速率，默认50
```
检测效果：
![auth dos](http://imglf4.nosdn0.126.net/img/c09lVS9TR3YrUGFMZGVTbGw4ZE9HZnJtOEE4UWlhbVdlSittaG9GTTZTcXhYdkRONy9LSFpBPT0.jpg?imageView&thumbnail=500x0&quality=96&stripmeta=0&type=jpg)

#### Deauthentication/Disassociation Amok
　　强制解除验证解除连接。在这个模式下，软件会向周围所有可见AP发起循环攻击......可以造成一定范围内的无线网络瘫痪（当然有白名单，黑名单模式），直到手动停止攻击！ 
常用命令：
```
mdk3 mon0 d
      -w <filename>             #白名单mac地址列表文件
      -b <filename>              #黑名单mac地址列表文件
      -s <pps>                        #速率，这个模式下默认无限制
      -c [chan,chan,chan,...]                  #信道，可以多填，如 2,4,5,1
```
检测效果：
![deauth amok](http://imglf5.nosdn0.126.net/img/c09lVS9TR3YrUGFMZGVTbGw4ZE9HYmRxWTMwTWFuY2pyZEpjOEFESWNuS01qdmJRUll5cUNBPT0.jpg?imageView&thumbnail=500x0&quality=96&stripmeta=0&type=jpg)

#### Brute-forcing access
　　其实以上攻击方式的共性就是干扰合法用户与接入点的通信，而在用户与接入点重新认证与关联的过程中，攻击者可以抓取这个过程中的握手包。握手包一般在10KB-1000KB不等，但这个数据包中包含了接入点的正确秘钥。攻击者在抓取到握手包以后，可以安安心心地回家使用EWSA工具暴力破解这个数据包的密钥，当得到密钥后，即得到了原接入点的访问权。
　　这种攻击方式一般没有特别好的防御方式，通常来说就是在察觉到有干扰通信的时候，定期修改一次密码，且保障密码的复杂性。

#### Rogue access points
　　钓鱼热点是WiFi环境下最常用且最容易获取用户凭证的一种攻击方式，类似WiFiPhsher、Fluxion等工具就可以开启高度定制化的钓鱼热点。在基于以上干扰用户连接的前提下，让用户错连接到攻击者开启的钓鱼热点，以此窃取用户凭证。
![rogue](http://imglf3.nosdn0.126.net/img/c09lVS9TR3YrUFlGTTVtbnVobUpvWWZLSGNCRE9hY29CTmFXSGFSdHhjWUhlYi9mdS9YRlh3PT0.jpg?imageView&thumbnail=500x0&quality=96&stripmeta=0&type=jpg)
　　虽然用户可能很难发现钓鱼热点与真实热点的细微差别，但我们的作品就能放大这其中的巨大变化。
##### SSID白名单
　　每一个WiFi接入点都有自己的MAC地址，而MAC地址也是它会发送的数据的其中一部分。一种检测流氓热点的方法就是设置一个可信接入点白名单，然后用MAC地址做标识来进行热点匹配。某些攻击者可能通过简单地伪装与原接入点相同的SSID名称来骗取用户的信任。

##### 变化的Beacon帧数值
　　前面说到的被动扫描中有说到Beacon帧，接入点向外发送的Beacon帧数值应当是规律且稳定的。若一个流氓接入点同时开启，那么，这个数量值就会成倍地增长。

##### 错误的信道
　　一般来说，一个接入点同一时刻只会使用一条信道，且不会经常去切换。因此，可以设置一个列表来存储所有受信任接入点的信道，如果信道不同，则说明该接入点有问题。

##### 信号强度异常
　　同上，接入点的物理位置通常都不会变化，所以它的相对信号质量会非常稳定。若一个流氓接入点在另一个地点开启，那么信号质量会有较大的波动。下面的图中，因为是实验环境，我的攻击网卡设备和接入点离得很近，因此这个图表没有较大的变化。
![rogue detection](http://imglf3.nosdn0.126.net/img/c09lVS9TR3YrUGFMZGVTbGw4ZE9HZDNNSjVQbktHMmF5U3lwbkdZMTVEb3hYWTkvSGN6ai9RPT0.jpg?imageView&thumbnail=500x0&quality=96&stripmeta=0&type=jpg)


### 参考
[无线攻击神器--MDK3 使用方法](http://ju.outofmemory.cn/entry/148457)
[WiFi接入原理-CSDN-小韩同学的博客](https://blog.csdn.net/superhcq/article/category/6704440)
[无线局域网安全-博客园-七夜的故事](https://www.cnblogs.com/qiyeboy/category/901758.html)
[common-wifi-attacks](https://wtf.horse/2017/09/19/common-wifi-attacks-explained/)