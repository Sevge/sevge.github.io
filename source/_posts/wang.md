---
title: 被玩坏的校园网
date: 2017-09-27 17:21:42
categories:
- 网络
---

## 校园网的升级
随着时代的发展，学校也紧跟潮流在今年升(zhang)级(jia)了校园网。之前校园网是纯有线锐捷客户端认证，现在校园网将电信联通合并，加上无线网络覆盖全校，实在是皆大欢喜，可喜可贺，可喜可贺啊！

### 有线认证方式
使用了多年的有线网络，通过锐捷客户端认证，原来￥80一学期还不限速的网络一去不复返咯。
![Ruijie](http://imglf5.nosdn.127.net/img/c09lVS9TR3YrUGJXNW4zQXB0dTJQNTA0VFNyUS9YMmVuY1RjbG9LeHVBVDJBVy9UL3VGdmpBPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)

### 无线认证方式
第一种方式是连接“HUNAU”，连接后无网络访问权限，通过网页认证，使用未购买套餐的账号登陆可以上学校内网。
![Wireless1](http://imglf5.nosdn.127.net/img/c09lVS9TR3YrUGJXNW4zQXB0dTJQenBCeWM3cncxNDlZU1duZThpTTNTTml5YU8zWndLNFRBPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)
![HUNAU](http://imglf5.nosdn.127.net/img/c09lVS9TR3YrUGJXNW4zQXB0dTJQenVRMHFKYzQyRzVYaXh2VEhRTEplQ3VJMkJNKzVMaFBRPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)
第二种方式是连接“HUNAU-Auto”，热点自带的认证登陆，使用未购买套餐的账号连接也可以上内网
![Wireless2](http://imglf6.nosdn.127.net/img/c09lVS9TR3YrUGJXNW4zQXB0dTJQeHpBL3dvY1BrVW8ybGd0aWpIS0xrSk5GUnMyTjlBdnlnPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)
当然，身为穷逼的我，肯定是续费了￥80一学期的纯有线网络(如今限速)，看着那高带宽和无线网络的价格，实在是消瘦不起。

## 已知的方式
目前我知道的绕过学校认证和限速的方法有以下几种：
> 1.DNS隧道
> 2.内网VPN
> 3.修改MAC
> 4.我瞎折腾的方法

这里记录一下几种方式的部署和使用，以下纯属实验，只是为了更深入地理解计算机网络相关知识啦~！

仅供参考，切勿用于不当目的！

## DNS隧道
### 简介
在连接到某个需要 Web 认证的热点之前，我们已经获得了一个内网 IP，此时，如果我们访问某个 HTTP 网站，网关会对这个 HTTP 响应报文劫持并篡改，302 重定向给我们一个 Web 认证界面（所以点 HTTPS 的网站是不可能跳转到 web 认证页面的）。[详细原理](http://www.ruijie.com.cn/fw/wt/36502)

我们看到了，网关（或者说交换机）都默认放行 DHCP 和 DNS 报文，也就是 UDP53 与 UDP 67。有些网关甚至不会报文进行检查，这也就意味着任何形式的数据包都可以顺畅通过。

既然如此，我们就可以在公网搞一台服务器，然后借此来免费上网，顺便还能防止网络审计(其实只是把钱花在服务器上了)。我们这次免费上网的主要突破点就是 UDP 53，当然了，据一位朋友实践，UDP 67、68也可以绕过 Web 认证。

### 环境监测
在部署之前有必要进行一下环境监测，以免造成因为环境不允许而做的一大堆无用功。
打开 cmd（GNU/Linux的终端），输入如下内容
```
nslookup www.baidu.com
```
或者使用[脚本](https://github.com/BennyThink/UDP53-Filter-Type)测试下,这里可以参照博主"土豆不好吃"的文章。[click me](https://www.bennythink.com/udp53.html)

若测试环境允许，可以进行下一步的搭建服务

### 服务搭建
这里也有很详尽的教程，参照CSDN博主"玖洲维城网络科技"的文章。[click me](https://blog.csdn.net/qq_35422558/article/details/78018089)

按照文章所述在服务器安装好Softether，在我的电脑安装上Openvpn并连接就能直接畅游网络了。

### 使用心得
以上，需要一台公网服务器，然后就可以无视认证直接连接到因特网，但由于我使用的是国内的学生机，所以受到带宽的限制...

### 下载地址
Softethrer Server64位版本: 链接: https://pan.baidu.com/s/1vNvIJscFLQj42XZ_Rnk6Rg 密码: xgjc
Softethrer Server树莓派ARM32位版本: 链接:https://pan.baidu.com/s/1c2GWopy 密码:fv4b
Softether的WINDOWS管理端：链接: https://pan.baidu.com/s/1bUrtKi 密码: y8s1
Softether的WINDOWS客户端：链接: https://pan.baidu.com/s/1c24fbLA 密码: fddr
OPVEN_GUI客户端：链接:https://pan.baidu.com/s/1nvkPPfN 密码:fc55

## 内网VPN
### 简介
前面说到，最新的校园网套餐实行了分档次的套餐，不同档次不同速率，这里的不同速率是指访问外网的速率，而内网的访问速率是没有限制的。

既然这样，那么假如我在学校内网有一台能够上网的机器，利用点对点连接，把它当做跳板不就行了？
![Vpn_inschool](http://imglf3.nosdn.127.net/img/c09lVS9TR3YrUFlTc2ZGSkl5eGdZQUFtdFpJdUdjd256emhDYVZtRThYS09WZ1BjRXBFSTlnPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)
### 服务搭建
恰好我在实验室有这样的资源，一台IP地址为"10.x.x.x"的服务器(其实买个树莓派插在实验室也一样)。按照上述[服务搭建](https://blog.csdn.net/qq_35422558/article/details/78018089)的步骤在这台服务器上安装Softether，然后在我的电脑上使用Openvpn的配置文件直接连接,搞定！
### 使用心得
以上，在登陆认证并获得内网访问权限后，通过Openvpn-Gui或者Softether客户端连接到我的内网服务器，不会受到任何限速！

## 修改MAC
### 简介
MAC地址在网卡中是固定的，每张网卡的MAC地址都不一样。网卡在制作过程中，厂家会在它的EPROM里面烧录上一组数字，这组数字，每张网卡都各不相同，这就是网卡的MAC(物理)地址。

由于MAC地址的唯一性，因此它主要用来识别网络中用户的身份。例如ADSL上网时，电信用它来记费，确认是你上的网；在校园网中，MAC地址也可以用来识别用户。对于校园网的正式用户登陆并认证后，其MAC地址会登记在服务器端，假如你是非法用户，服务器中就没有你的网卡MAC地址，这样当你试图连上网时，服务器就会立刻认出你、阻止你连上网络。

### 操作步骤
1. 扫描校园局域网中的在线主机，这个根据不同情况操作起来不太一样，我使用的是IP SCANNER。

2. 这里以我室友的MAC地址作为示例，看到室友的MAC地址如图：
![The_mac](http://imglf6.nosdn.127.net/img/c09lVS9TR3YrUGJ6Wm1EWFlGdmJ6UXZNUXhGTmNzWGUyd3Yyb0F5ZGpGQTBDUVMzdDhINS9BPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)
3. 在我的电脑中打开"网络与共享中心"-"更改适配器设置"-"以太网"-"属性"-"配置"-"高级"，找到"网络地址"，修改成上图的地址。
![change](http://imglf5.nosdn.127.net/img/c09lVS9TR3YrUGJ6Wm1EWFlGdmJ6WDJ2bmZaU2ZqQjdjRVlFSHZLaTNmdFhCTXovVzYyc3R3PT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)
4. 当我按下确定按钮的那一刻，室友的电脑已经断网，而我这边已经可以愉快地上网冲浪了~
![fail](http://imglf3.nosdn.127.net/img/c09lVS9TR3YrUGJ6Wm1EWFlGdmJ6VERZZ3dNUGF4VUJqaVQrSUM1UDlodkxSUXJHL2NldHRnPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)

### 使用心得
因为MAC地址是唯一的，为了防止室友跑过来打我，我马上将设置恢复了初始状态......

## 瞎折腾
### 简介
这种方式完全是我瞎折腾出来的，原理猜测是：无线认证和有线认证是分开的两台服务器，但两者是有同步消息的。在我的电脑，两个客户端(无线认证和有线认证)同时认证卡住两台服务器同步消息的时间，使两台服务器消息矛盾，绕开了他们的限制。

### 操作步骤
1. 将有线网卡禁用，通过无线认证第一种方式连接"HUNAU"，打开浏览器自动跳出登陆网页，这是无线认证服务器的认证端网页，先保留着，URL如下，其中有mac、userip等参数：
`http://10.100.0.12:9090/zportal/loginForWeb?wlanuserip=37c61a32243725e8412223107e8670d4&wlanacname=00aab905808bf54238202dd3074e226b&ssid=99f34848c4e3872f&nasip=3a55a6e233ce66a3e3c9d19c2572b2ea&snmpagentip=&mac=e2483bb22f79a96b0b178e83ca255d91&t=wireless-v2&url=1cd4c9d683b191233d7be2539eb794196a2a58b150f012b910b74a2fde544bac1f10cdd1bb45c9aeec992c5a1746e8ea6ea1b480e0d713af4a1725a95600c0b3&apmac=&nasid=00aab905808bf54238202dd3074e226b&vid=6ff7431ed4e21b22&port=e2bcde16e9a8b04a&nasportid=a25b45948c15af40d095454b31cfa807fcfd7ec39a63f462933adb33ce566a2a
`

2. 将无线网卡禁用，打开有线网卡，打开有线认证方式的锐捷客户端。

3. 回到浏览器的网页上，打开记住密码和自动登陆：
![Login_page](http://imglf5.nosdn.127.net/img/c09lVS9TR3YrUGJXNW4zQXB0dTJQeFVUQnJ4MXNaeVovakoxa1dTR3Y5TWR4dkJjRTRPYTJBPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)
4. 最重要的一步！锐捷认证客户端点连接，然后马上回到网页认证点登陆，手要快！确保几乎在同时认证。

5. 然后可以看到两者同时登陆上。
若如平时正常登陆有线锐捷客户端，之后再登陆这个网页认证端，其中必有一个被挤下线，也不能访问因特网。
![Login_success](http://imglf6.nosdn.127.net/img/c09lVS9TR3YrUGJXNW4zQXB0dTJQNHNHSWIrMGhBblNCWUVpcEtMMFNYVnJIL1VCR2EvZnNBPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)
![Auth_success](http://imglf6.nosdn.127.net/img/c09lVS9TR3YrUGJXNW4zQXB0dTJQMkpYNk01eGcyWUl0TXBPYWUzNVBXSmR6eUhwMzljSjNnPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)

### 使用心得
网速测试是20M带宽，这种方式好像同时绕过了限速，看到室友2M、4M蛋疼的小水管，躲在角落偷偷笑。

使用发现，通过这种方式连接的网络十来分钟后会断掉，之后不能解析地址，不能ping通8.8.8.8，QQ不能发送和接收消息，但是奇怪的是正在观看的直播不会断。

## 未来
有待进一步探究...