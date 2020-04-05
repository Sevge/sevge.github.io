---
title: 无线密码攻击
date: 2017-12-11 20:38:09
categories:
- Kali
---
## 前言
当今时代，几乎每个人都离不开网络。随着网络的普及，无线网络逐渐扎根于人们的生活之中。然而，很多情况下，这些无线信号都需要身份验证后才能使用。

现在我要讲的就是破除这道身份验证，连接上内网。当然，这不仅仅是可以上网了，做其他事情也更加方便。

以下操作实验使用的都是自家无线路由器，使用的主要工具是Aircrack-ng。(Aircrack-ng是无线渗透测试的经典工具，它是一款基于破解无线802.11协议的WEP以及WPA-PSK加密的工具。)

## WEP加密的无线网络
### 简介
Wired equivalent privacy（WEP）协议是对在两台设备间无线传输的数据进行加密的方式，用来防止非法用户窃听或者侵入无线网络。不过密码分析学家已经找出WEP的好几个弱点，因此2003年被WI-FI protected access（WPA）淘汰，又在2004年由完整的IEEE 802.11i标准（WPA2）所取代。

WEP的破解为利用加密体制缺陷，通过收集足够的数据包，使用分析加密算法还原出密码。 
### 步骤
下面我以自家用路由器进行示例。
![WEP](http://imglf3.nosdn.127.net/img/c09lVS9TR3YrUFp0MEpaVmEzalN0WUw3U0kySXpXcnB3Y1NNM1dxM3puRUhrbzRQdk1RVk5nPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)
1. 启动KALI终端，输入`airmon-ng`命令查看当前系统中的无线网路接口：
![airmon-ng](http://imglf3.nosdn.127.net/img/c09lVS9TR3YrUFp0MEpaVmEzalN0YlQxRWNna3RMZ29UbmhxRzNwUnlPZDc1dUkrWXdySWdBPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)
从输出的信息可以看出，当前系统存在一个无线网络接口。从输出结果的Interface列，可以知道当前系统的无线接口为wlan0。

2. 开启监听模式：`airmon-ng start wlan0`
![airmon-ng start](http://imglf5.nosdn.127.net/img/c09lVS9TR3YrUFp0MEpaVmEzalN0UmNJckZYNjhVbEJzNWwzRE9oNVdhOUppQ1cwWVR2d2tnPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)
输出信息显示监听模式被启用，映射端口为wlan0mon。

3. 使用`airodum-ng wlan0mon`命令定位附近所有可用的无线网络。
![scan](http://imglf3.nosdn.127.net/img/c09lVS9TR3YrUFp0MEpaVmEzalN0Y1YyWkdQR0REWEJaZWFiR3ZMR2lUblJtb3F5MkF1T1FRPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)
以上输出信息显示了附近所有可用的无线网络。从输出信息可以看到很多参数：
> BSSID:无线的mac地址、
> PWR：网卡报告的信号水平（这个值越小信号越好）
> Beacons：无线发出的通告编号
> CH：AP使用的信道（从Beacons中获取）
> MB：无线所支持的最大速率
> ENC：加密方式

4. 使用`airodump-ng`捕获指定BSSID的文件。
常用命令：
> -c 指定选择的频道
> -w 指定一个文件名，用于保存捕获的数据
> --bssid 指定攻击的Bssid

下面将Bssid为EC:26:CA:C6:CB:1B的无线路由器作为攻击目标。
```
airodump-ng -c 10 -w catch --bssid EC:26:CA:C6:CB:1B wlan0mon
```
![airodump-ng](http://imglf4.nosdn.127.net/img/c09lVS9TR3YrUFp0MEpaVmEzalN0V1dsOEZObTFsMHo5SUFZbHVyZ2cyNU55ZHFXSWQ0alR3PT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)
从输出信息可以看到Essid为TP-LINK的无线路由器的beacons和#Data一直在变化，表示有客户端与AP发生数据交换。从以上命令执行完毕后，会生成一个名为catch-01系列的文件，为了方便后面破解时候的调用，所有保存的文件按顺序编了号，于是就多了-01这样的编号，后面再执行会有-02，-03，以此类推。

5. 打开一个新的终端窗口，执行aireplay命令。使用aireplay发送一些流量给无线路由器，以至于能够捕获到数据。其中，-b后接AP的mac，-h接我们自己网卡wlan0的mac地址
![aireplay](http://imglf3.nosdn.127.net/img/c09lVS9TR3YrUFp0MEpaVmEzalN0Ui8vWSs5UmwrempaUGlDeS9kcmxxRFpVcEc3VHM3c3RBPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)
输出信息就是使用ARP Requests的方式来读取ARP请求报文的过程，此时回到airodump-ng界面查看，可以看到TP-LNK的Frames栏的数字在飞速地增加，在抓取的无线数据报文达到一定数量后，就可以开始破解，若不能成功就等待数据报文继续抓取，然后多尝试几次。

6. 再新建一个终端，在新终端执行aircrack-ng catch-02.cap成功得到密码。其中第一次我抓了1W+ 的DATA没有出密码，第二次等得稍微久点，抓了2W+DATA出了密码
其中捕获文件用了大概半小时，破解密码仅仅用了四秒时间！
![wep_catch](http://imglf6.nosdn.127.net/img/c09lVS9TR3YrUFp0MEpaVmEzalN0YVdHR0I5V1BYR3R1VzZwODhEQWxianEvVythZkRBdWJBPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)
### 获得KEY
![found_wep_key](http://imglf6.nosdn.127.net/img/c09lVS9TR3YrUFp0MEpaVmEzalN0YVBnY3VQTTFyajdVelR6QXRMcDRjaXRITXNTek1Bd293PT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)

## WPA、WPA2加密的网络
### 简介
WPA全名为Wi-Fi Proteted Access，有WPA和WPA2两个标准。它是一种保护无线电脑网络安全的协议。对于启用WPA/WPA2加密的无线网络，其攻击和破解步骤及攻击时完全一样的。当使用aireplay-ng进行攻击后，同样获取到WPA握手数据包及提示；在破解时需要提供一个密码字典。

### 步骤
这里我仍然以我家的路由器为例：
![WPA](http://imglf3.nosdn.127.net/img/c09lVS9TR3YrUFp0MEpaVmEzalN0Y1VjTmYwYzdBZDhJdXFGWm4wemd1am1sWDhxNmNyQ3V3PT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)

1.	查看无线网络接口
![airmon-ng](http://imglf3.nosdn.127.net/img/c09lVS9TR3YrUFp0MEpaVmEzalN0YlQxRWNna3RMZ29UbmhxRzNwUnlPZDc1dUkrWXdySWdBPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)

2.	启用无线网络接口监听
![airmon-ng start](http://imglf5.nosdn.127.net/img/c09lVS9TR3YrUFp0MEpaVmEzalN0UmNJckZYNjhVbEJzNWwzRE9oNVdhOUppQ1cwWVR2d2tnPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)

3.	获取相关AP的信息，`airodump-ng wlan0`
![scan](http://imglf3.nosdn.127.net/img/c09lVS9TR3YrUFp0MEpaVmEzalN0Y1YyWkdQR0REWEJaZWFiR3ZMR2lUblJtb3F5MkF1T1FRPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)

4.	捕获数据包，执行
```
airodump-ng -c 4 -w wpa --bssid EC:26:CA:C6:CB:1B wlan0mon
```
常用命令：	  
> -c 指定选择的频道
> -w 指定一个文件名，用于保存捕获的数据
> --bssid 指定攻击的Bssid

![airodump-ng](http://imglf4.nosdn.127.net/img/c09lVS9TR3YrUFp0MEpaVmEzalN0V1dsOEZObTFsMHo5SUFZbHVyZ2cyNU55ZHFXSWQ0alR3PT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)

5.新建一个终端（之前打开的终端不要关闭！），对无线路由器进行Deauth攻击（取消验证攻击，迫使已经连接的客户端断开；当客户端自动连接的时候，即可抓取握手包）：
```
aireplay -0 3 –a EC:26:CA:C6:C:1B –c EC:9B:F3:E0:27:8F wlan0mon
```
> -0 ：指定为取消验证攻击 ，3 为攻击次数为3 
> -a ：指定AP的mac地址
> -c ：指定连接AP的客户端的mac地址

![aireplay](http://imglf3.nosdn.127.net/img/c09lVS9TR3YrUFp0MEpaVmEzalN0Ui8vWSs5UmwrempaUGlDeS9kcmxxRFpVcEc3VHM3c3RBPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)
执行完后可以看到airodump终端的右上角抓到了握手包:
![handshake](http://imglf3.nosdn.127.net/img/c09lVS9TR3YrUFp0MEpaVmEzalN0WkV5SnRSUGVDanZSN0pDd1BVRWR6SUh1MHprZ01QbDBRPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)

抓到握手包后，使用aircrack-ng进行暴力破解。
执行命令：`aircrack-ng -w pass.txt wpa-01.cap`，接下来就是无尽的跑字典过程了。

### 获得KEY
![dic_passwd](http://imglf3.nosdn.127.net/img/c09lVS9TR3YrUFp0MEpaVmEzalN0VnB2UFlmdkc3SERjVUVmcXoxNVlxMFBXUDJyNGdZTlBBPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)
这里我使用的是8位纯数字字典，其大小约858M。
Airacrack跑字典的速度取决于你的电脑的配置，找出密码的速度则取决于字典的质量还有运气了。

## WPS(Wi-Fi Protect Setup)
### 简介
WPS是由WIFI联盟推出的全新WIFI安全防护设定标准。该标准主要是为了解决无线网络加密认证过于繁杂的弊病。因为很多用户觉得设置步骤太麻烦，不做任何安全设定。所以很多人使用wps设置无线设备，可以通过个人识别码(PIN)或按钮（PBC）取代输入一个很长的密码。路由器开启wps功能后，会随机生成一个8位的pin码，通过暴力枚举pin码，达到破解的目的。

pin码是由8位纯数字组成的识别码，pin码破解是分三部分进行的，规律是这样的：pin码分为三部分，如图：
![pin_part](http://imglf4.nosdn.127.net/img/c09lVS9TR3YrUFp0MEpaVmEzalN0WGhCMGs1UFNsbFJSNStCcmJod0hxamhRWWNLUDE3SzFnPT0.jpg?imageView&thumbnail=500x0&quality=96&stripmeta=0&type=jpg)
前4位为第一部分，第5-7位为第二部分，最后1位为第三部分。第一部分的验证跟第二部分没关联，最后1位是根据第二部分计算得出的校验码。

破解一开始是先单独对第一部分进行pin码匹配，也就是说先破解前4位pin码。前4位是0000-9999总共10000个组合。

当前4位pin码确定后再对第二部分进行pin码匹配，也就是再对5-7位进行破解，5-7位是000-999总共1000个组合。

当前7位都确定后，最后1位也会自动得出，至此即可得出密码。

根据pin码破解的原理，可以看到只需要枚举11000种情况就会必然破解出pin码，从而通过pin码得到WIFI密码。
### 步骤
由于我家的路由器没有这个功能，找到我姑姑家的老式TP-LINK：	
![tp-link](http://imglf6.nosdn.127.net/img/c09lVS9TR3YrUFp0MEpaVmEzalN0Ym9xcUFMamN1ejVJRlYyZnlYdzIwMXJxWEhldW9NNkVBPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)

1.	打开终端执行`airmon-ng`检测网卡
![airmon-ng](http://imglf3.nosdn.127.net/img/c09lVS9TR3YrUFp0MEpaVmEzalN0YlQxRWNna3RMZ29UbmhxRzNwUnlPZDc1dUkrWXdySWdBPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)

2.	开启监听模式
![airmon-ng start](http://imglf5.nosdn.127.net/img/c09lVS9TR3YrUFp0MEpaVmEzalN0UmNJckZYNjhVbEJzNWwzRE9oNVdhOUppQ1cwWVR2d2tnPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)

3.	扫描开启WPS的设备，LCK为NO的都可以爆破试试
![wps_on](http://imglf3.nosdn.127.net/img/c09lVS9TR3YrUFp0MEpaVmEzalN0WGFhKzcxSThqRW05cG1oSUVTMGpQMjRqUkdFckpVc3lnPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)

4.	使用Reaver爆破
reaver 命令：            　　
`reaver -i mon0 -b mac -S -v`  
reaver命令参数:      
> -i 监听后接口名称     
> -b 目标mac地址       
> -S 使用最小的DH key（可以提高爆破速度）     
> -vv 显示更多的非严重警告     
> -d  即delay每穷举一次的闲置时间 预设为1秒         
> -c  指定频道可以方便找到信号，如-c　1 指定1频道  　
> -N 不发送NACK信息（如果一直pin不动，可以尝试这个参数）

终端执行：
```
reaver -i wlan0mon 20:DC:E6:D1:DE:E4 -N -vv -c 8 
```
### 获得KEY
![reaver](http://imglf6.nosdn.127.net/img/c09lVS9TR3YrUFp0MEpaVmEzalN0V2Jyc2Vzb0RjR2NmRTlwWnhSMnVaQlpGVEkwSlJPZWNRPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)
当Reaver暴力穷举出正确的pin码时，不管无线路由器的密码有多复杂，它都是手到擒来了。

## WIFI万能钥匙
### 简介
Wifi万能钥匙的使用十分简单，而且据我的个人经历，小区范围内私人使用的无线网络基本可以使用WIFI万能钥匙解开。要得到WIFI的密码，，只需要三步即可到位。
### 步骤
1.	打开WIFI万能钥匙，可以看到有钥匙图标的热点可以直接解开：
![WIFI_master](http://imglf5.nosdn.127.net/img/c09lVS9TR3YrUFp0MEpaVmEzalN0WXkyNVZ5TXRkRzZ2WElVVFc5WE9EWkxpM2dCZ25SUWRRPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)

2.	解开密码，连接上WIFI后，打开RE管理器（安卓手机下的文件管理器），进入如下路径（需要root权限）：`/data/misc/`
![RE](http://imglf3.nosdn.127.net/img/c09lVS9TR3YrUFp0MEpaVmEzalN0ZjlLblBVWkw1UVZLdjY2WWRxOFFWd3hLVVNxektXcjFRPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)

3.	打开wpa_supplicant.conf可以看到刚刚连接的WIFI的密码：
![cat_passwd](http://imglf3.nosdn.127.net/img/c09lVS9TR3YrUFp0MEpaVmEzalN0Y3FoMVQ1N00rZEZkWlBpL1FwMDBDTVVMd0hrSFlLVEVnPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)

## 总结
以上，是我平时取得WIFI密码的常用方法。

基本思路是：
先看万能钥匙能否解开，这个方式最简便
然后再看加密方式，WEP加密参考0x02，
若是WPA加密先看AP是否开启WPS,参考0x04，
否则只能跑字典，参考0x03。
