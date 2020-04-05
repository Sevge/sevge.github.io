---
title: Common WiFi Attacks And How To Detect Them
date: 2018-01-19 11:36:57
categories:
- 网络
---
## 原版
### The issue with 802.11 Management frames
The 802.11 WiFi standard contains a special frame (think "packets" in classic, wired networking) type for network and connection management. For example, your computer is not actively "scanning for networks" when you hit the tray icon to see all networks in range, but it passively listens for so-called "beacon" management frames from access points broadcasting to the world that they are there and available.

Another management frame is the "probe-request" ("Hi, is my home network in range?") that your devices are sending to see if networks they connected before are in range. If such a network is in range, the relevant access points would respond with a "probe-response" frame ("Hi, yes I'm here! You can connect to me without waiting for a beacon frame.")

The problem with management frames is that they are completely unencrypted. This makes WiFi easy to use because, for example, you can see networks and their names around you without exchanging some key or password first, but it also makes WiFi networks prone to many kinds of attacks.
![wifi](http://imglf3.nosdn.127.net/img/c09lVS9TR3YrUGFyM1dJczg1WlByMjhxSUdLbUYrUnhhZWcyY0xDajJod2h5bm5QZ1YzVkpBPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)
### Common attacks explained
#### Sniffing traffic
Virtually all WiFi traffic can be sniffed with adapters in monitor mode. Most Linux distributions support to put certain WiFi chipsets into this special mode that will process all traffic in the air and not only that of a network you are connected to. Everyone can get WiFi adapters with such a chipset from Amazon, some for less than $20.

Encrypted networks will also not really protect you. WEP encryption can be cracked in a matter of minutes and even WPA2-PSK is not secure if you know the passphrase of a network (for example, because it's the office network and you work there, of because the local coffee shop has it written on the door) and can listen to the association process of the device. This works because the device-specific encryption between you and the access point uses a combination of the network passphrase and another key that is publicly exchanged (remember, management frames are not encrypted) during the association process. An attacker could force a new authentication process by spoofing a deauthentication frame that will disconnect your device for a moment. (more on that below)

#### Detecting sniffers
Sniffing traffic is passive and cannot be detected. As a user, consider all WiFi traffic on open or closed to be public and make sure to use encryption on higher layers, like HTTPs. (Really, you should be doing this anyways, in any network.)

#### Brute-forcing access
Like any other password, passphrases for wireless networks can be brute-forced. WEP can be cracked by analyzing recorded traffic within minutes and has been rendered useless. For WPA secured networks you'd need a standard dictionary attack that just tries a lot of passwords.

#### Detecting brute force attacks
Brute-forcing by actually authenticating to an access point is extremely slow and not even necessary. Most brute force cracking tools work against recorded (sniffed) WiFi traffic. An attacker could just quietly sit in the car in front of your office, recording traffic for some time and then crack the password at home.

Like sniffing, this approach cannot be detected. The only protection is to use a strong password and to avoid WEP.

#### Jamming
The obvious way of jamming WiFi networks would be just to pump the relevant frequencies full of garbage. However, this would require fairly specialist equipment and maybe even quite some transmitting power.

Surprisingly, the 802.11 standard brings a much easier way: Deauthentication and disassociation frames. Those "deauth" frames are supposed to be used in different scenarios, and the standard has more than 40 pre-defined reason codes. I selected a few to give you an idea of some legitimate use-cases:

>    1. Previous authentication no longer valid
>    2. Disassociated due to inactivity
>    3. Disassociated because AP is unable to handle all currently associated STAs
>    4. Association denied due to requesting STA not supporting all of the data rates in the BSSBasicRateSet parameter
>    5. Requested from peer STA as the STA is leaving the BSS (or resetting)

Because deauth frames are management frames, they are unencrypted, and anyone can spoof them even when not connected to a network.

Attackers in range can send constant deauth frames that appear to come from the access point you are connected to (by just setting the "transmitter" address in the frame) and your device will listen to that instruction. There are "jammer" scripts that sniff out a list of all access points and clients, while constantly sending deauth frames to all of them.

#### Detecting jammers
A tool like nzyme (to be released - see introduction) would sniff out the deauth frames, and Graylog could alert on unusual levels of this frame subtype.

#### Rogue access points
Let's talk about how your phone automatically connects to WiFi networks it thinks it knows. There are two different ways this can happens:

>    It picks up beacon frames ("Hi, I'm network X and I'm here.") of a network it knows and starts associating with the closest (strongest signal) access point.

>    It sends a probe-request frame ("Hello, is an access point serving network X around?") for a known network and an access point serving such a network responds with a probe-response frame. ("Hello, yep I'm here!") Your phone will then connect to that access point.

Here is the problem: Any device can send beacon and probe-response frames for any network.

Attackers can walk around with a rogue access point that responds to any probe-request with a probe-response, or they could start sending beacons for a corporate network they are targeting.

Some devices now have protections and will warn you if you they are about to connect to a network that is not encrypted but was previously encrypted. However, this does not help if an attacker knows the password or just targets an unencrypted network of your coffee shop. Your phone would blindly connect, and now you have an attacker sitting in the middle of your connection, listening to all your communications or starting attacks like DNS or ARP poisoning. An attacker could even show you a malicious captive portal (the sign-in website some WiFi networks show you before they'll let you in) to phish or gather more information about your browser.

Take a look at a miniaturized attack platform like the famous [WiFi Pineapple](https://www.wifipineapple.com/) to get an idea of how easy it is to launch these kinds of attacks.

Rogue access points are notoriously hard to spot because it's complicated to locate them physically and they usually blend into the existing access point infrastructure quite well - at least on the surface. Here are some ways to still spot them using my to-be-released tool nzyme and Graylog:

#### Detecting rogue access points
##### BSSID whitelisting
Like other network devices, every WiFi access point has a MAC address that is part of every message it sends. A simple way to detect rogue access points is to keep a list of your trusted access points and their MAC addresses and to match this against the MAC addresses that you see in the air. The problem is that an attacker can easily spoof the MAC address and, by doing that, circumvent this protective measure.
##### Non-synchronized MAC timestamps
It is important that every access point that spawns the same network has a highly synchronized internal clock. For that reason, the access points are constantly exchanging timestamps for synchronization in their beacon frames. The unit here is microseconds, and the goal is to stay synchronized within a delta of 25µs.

Most rogue access points will not attempt to synchronize the timestamps properly, and you can detect that slip.
##### Wrong channel
You could keep a list of what channels your access points are operating on and find out if a rogue access point is using a channel your infrastructure is not supposed to use. For an attacker, being detected by this method is extremely easy: Recon the site first and configure the rogue access point to only use already used channels. Another caveat here is that many access points will dynamically switch channels based on capacity anyways.
##### Crypto drop
An attacker who does not know the password of an encrypted network she targets might start a rogue access point that spins up an open network instead. Search for networks with your name, but no (or the wrong) encryption.
##### Signal strength anomalies
There are many ways to spot a rogue access point by analyzing signal strength baselines and looking for anomalies. If an attacker sits on the parking lot and is spoofing one of your access points, including its MAC address (BSSID), it will suddenly have a change in the mean signal strength because he is further away from the sensor (nzyme) then the real access point.

[Written by Lennart Koopmann](https://wtf.horse/2017/09/19/common-wifi-attacks-explained/)

## 翻译
### 802.11管理帧存在的问题
802.11 WiFi标准包含一种专门针对网络和连接管理的特殊帧类型。比如说，当你点击电脑左下方的托盘图标来查看当前范围内的所有可用网络时，你的电脑并不会主动“扫描网络”，但它会被动监听WiFi热点所广播出来的“beacon”管理帧(用来表明该热点可用)。

另一种管理帧名叫“probe-request”，它的作用是代表WiFi网络的可访问距离，你的设备会发送这种管理帧来查看之前连接过的网络当前是否在周围。如果距离内存在已访问过的网络，相应的热点将会用“probe-response”帧予以响应。

而这些管理帧存在的问题就是，它们完全没有经过任何的加密。这样做的目的是为了增加WiFi的易用性，因为你完全不需要进行任何的密钥交换或密码确认就可以查看到周围的WiFi网络以及热点名称，但这也增加了WiFi网络的攻击面。

### 常见攻击技术介绍
#### 嗅探流量 
实际上，所有的WiFi流量都是可以通过监听模式的适配器来嗅探的。大多数Linux发行版都支持WiFi芯片切换到这种特殊模式，并处理周围环境中的所有WiFi流量（不仅是你连接到的网络）。任何人都可以直接在X宝买到带有这种芯片的无线网卡，而且价格都不贵。

而且，加密网络其实也保护不了你。破解WEP加密也只是几分钟的时间而已，甚至连WPA2-PSK都是不安全的（如果你知道密码的话）。比如说，你可以窃听办公室的WiFi网络，因为你知道密码，楼下咖啡厅的WiFi也不安全，因为他们的WiFi密码一般都写在桌子上。因为你和热点之间设备特定的加密使用的是一套网络密码组合，而另一个密钥是在协商过程中通过公开交换获取的（别忘了管理帧是没有经过加密的）。攻击者将能够通过伪造去认证帧来强制发起新的认证过程，而这将导致你的设备跟热点之间出现短暂的掉线。

#### 检测网络嗅探活动
嗅探流量是一种被动行为，所以它是不能被检测到的。作为一名用户而言，你可以认为所有的WiFi流量都是开放的，所以你一定要确保使用了更高层的加密手段，例如HTTPS。

#### 暴力破解访问
对热点进行暴力破解攻击其实是非常耗时间的，而且也完全没有必要，大多数暴力破解工具都可以记录（嗅探）WiFi流量。攻击者可以安静地坐在你办公室楼下的咖啡厅，记录你办公室网络的流量，然后回家再慢慢破解你的WiFi密码。

与流量嗅探一样，这种行为同样是无法被检测到的。我唯一能给你的建议就是使用健壮的WiFi密码，并且不要使用WEP。

#### WiFi干扰
一般来说，检测WiFi干扰行为将需要相对专业的设备才进行，而且有时甚至还需要使用到信号发射塔。但是有趣的是，802.11标准给我们提供了一种更简单的方法：去认证帧和去关联帧。这些“去认证”帧可以被用于多种不同的场景，而且该标准提供了超过40种预定义的原因代码。下面给出的是一些合法的常用示例：
> 1. 之前的身份认证失效；
> 2. 由于不活动而导致的连接断开；
> 3. 由于访问点无法处理当前所有的关联STA而导致的连接断开；
> 4. 由于SAT不支持BSSBasicRateSet参数种的数据率而导致的拒绝连接；

因为去认证帧属于管理帧的一种，所以它们是没有经过加密的，而攻击者甚至可以在无需连接该网络的情况下伪造这种帧。信号范围内的攻击者可以向目标用户所连接的热点发送连续的去认证帧来达到干扰WiFi的目的。

#### 检测WiFi干扰器
类似nzyme（即将发布）这样的工具可以发现这种去认证帧，而且我们还可以通过查看WiFi日志来发现这种帧。

#### 流氓热点
接下来，我们讨论一下手机在自动连接至WiFi网络时会发生什么情况。一般来说，这种情况主要会发生在以下两种场景:
> 手机获取已知WiFi网络的beacon帧，然后开始与距离最近（信号最强）的热点进行连接。

> 手机给已知WiFi网络发送一个probe-request帧，可提供网络服务的接入点将响应一个probe-response帧。接下来，你的手机将会跟这个响应接入点进行连接。

这里的问题就在于：任何设备都可以给任何网络发送beacon帧和probe-response帧。

攻击者可以搭建一个便携式的流氓接入点，这个接入点不仅能够响应（probe-response）任何的probe-request帧，而且它们还能够给任何的目标网络发送beacon帧。

现在的很多设备也都部署了相应的保护机制，如果你准备连接到一个之前加密但当前未加密的网络，那么设备将会给你发出警告提醒。不过，如果攻击者知道你之前所连接的WiFi密码或者说本身他攻击的就是一个开放网络的话，这种保护机制就没有任何效果了。此时，你的手机将会毫不犹豫地连接到流氓热点，而攻击者将能够获取到你所有的网络流量（类似中间人攻击）。除此之外，攻击者甚至还可以让用户的浏览器呈现恶意页面并发动网络钓鱼攻击。关于这个方面，大家可以参考一下著名的网络钓鱼攻击平台WiFi Pineapple，你很快就会知道这种攻击是有多么简单了。

其实大家都知道，流氓接入点是很难被发现的。我们不仅很难去对它们进行物理定位，而且我们也无法从众多合法热点中发现那些流氓接入点。

#### 检测流氓接入点
##### BSSID白名单
跟其他网络设备一样，每一个WiFi接入点都有自己的MAC地址，而MAC地址也是它会发送的数据的其中一部分。一种检测流氓热点的方法就是设置一个可信接入点白名单，然后用MAC地址做标识来进行热点匹配。但是问题就在于，攻击者仍然可以轻而易举地伪造MAC地址。
##### 非同步的MAC时间戳
生成相同网络的接入点都会拥有高度同步的内部时钟。因此，接入点会不断地交换时间戳以实现同步，这个时间是毫秒级的，同步增量为25微秒。大多数流氓热点在尝试进行时间戳同步时往往会出现各种各样的错误，你可以通过检测这种错误来发现流氓热点。
##### 错误的信道
你可以设置一个列表来存储所有受信任接入点的信道，如果信道不同，则说明该接入点有问题。但是对于攻击者来说，这种保护方式也是能够轻松绕过的。
##### 丧失加密
如果攻击者不知道他的目标加密网络的密码，一般会启动一个开放的网络，启动一个相同名称的网络，但通常没有加密。
##### 信号强度异常
我们还可以通过分析WiFi信号的强度来检测流氓热点。如果攻击者伪造了一个接入点的话，你会发现其MAC地址（BBSID）和信号强度会突然发生改变。

[参照FREEBUF](http://www.freebuf.com/articles/wireless/148773.html)
