---
title: 使用Fluxion钓鱼
date: 2017-12-18 08:35:41
categories:
- Kali
---
## 简介
之前介绍了无线密码攻击的几种方式，使用Aircrack-ng的弊端是暴力破解最终可能得不到正确的秘钥。

这里介绍的另外一种新姿势，就是针对用户进行的攻击，没有哪一个系统能扛得住社会工程学技术的入侵，Fluxion就是利用了钓鱼的方法来伪造登录界面进行入侵。(类似的工具还有WIFI-Phisher)

以下操作使用的是我寝室的TP-LINK作为实验。

## 功能
1. 扫描能够接收到的WIFI信号  　　
2. 抓取握手包(这一步的目的是为了验证WiFi密码是否正确)  　　
3. 使用WEB接口  　　
4. 启动一个假的AP实例来模拟原本的接入点  　　
5. 然后会生成一个MDK3进程。如果普通用户已经连接到这个WiFi，也会输入WiFi密码  　　
6. 随后启动一个模拟的DNS服务器并且抓取所有的DNS请求，并且会把这些请求重新定向到一个含有恶意脚本的HOST地址  　　
7. 随后会弹出一个窗口提示用户输入正确的WiFi密码  　　
8. 用户输入的密码将和第二步抓到的握手包做比较来核实密码是否正确  　　
9. 这个程序是自动化运行的，并且能够很快的抓取到WiFi密码

## 原理
我根据自己的理解，画了下图：
![process](http://imglf6.nosdn.127.net/img/c09lVS9TR3YrUGFBUXdEOVVoejh4QiswNUd3eXh0TzBjSGJrWWJ6Y0ZEWXNaVzd6dk5YSllRPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)

## 操作步骤
### 克隆
```
# git clone --recursive https://github.com/FluxionNetwork/fluxion.git 
```
![git_clone](http://imglf5.nosdn.127.net/img/c09lVS9TR3YrUGFBUXdEOVVoejh4SlNhSVJSZkIwRnFVSkNoM1VXZEkxQW9tbERCSks0QUZRPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)

### 开始
启动脚本
```
# cd fluxion
# ./fluxion
```
![init](http://imglf5.nosdn.127.net/img/c09lVS9TR3YrUGFBUXdEOVVoejh4QVFnL2Zzclp3dmVWcUxjQmxTcVJabzJxQ1ZvalNRQ1VRPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)
这时fluxion会检查它必须的组件，如果没有的话它会自动安装。

### 选项
等待检查和安装后，会出现选项界面，以下根据个人情况选择。
依次选择语言，选择网卡wlan0，选择监控所有信道
![option1](http://imglf4.nosdn.127.net/img/c09lVS9TR3YrUGFBUXdEOVVoejh4Q3QzdDBJajdnN3diTlJTaVdXYitxallLMTYrRG11UW9RPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)
之后它会将附近所有信道的AP列出
![option2](http://imglf4.nosdn.127.net/img/c09lVS9TR3YrUGFBUXdEOVVoejh4SWxQd0Y2VGRZYjlwSkxwQ0Q3bjZzZG9ISU8wUTFJWWNnPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)
等待一段时间，按下ctr+c停止扫描，选择需要的AP，我选择69。
![option3](http://imglf3.nosdn.127.net/img/c09lVS9TR3YrUGFBUXdEOVVoejh4RG0rWGp4MXhhUVArRkhxTmNUMUJaaVN1bXRhUyt4MHlBPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)
这里选择1，
![option4](http://imglf6.nosdn.127.net/img/c09lVS9TR3YrUGFBUXdEOVVoejh4TjRMWFhrRWlpcEs4VTVHbDFiUXhOeU9qdWl1bmVmNGtnPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)
之后选择我的wlan0网卡，选择推荐的1选项，
![option5](http://imglf5.nosdn.127.net/img/c09lVS9TR3YrUGFBUXdEOVVoejh4SzUvOVNYNkpBdStvRlhQYnpGVHBONmZzcTI5enBBQVlnPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)
开启新的一个终端，使用airodump抓一个握手包。
我抓到握手包是`/root/tool/w-01.cap`
之后然后选择创建ssl证书:
![option6](http://imglf5.nosdn.127.net/img/c09lVS9TR3YrUGFBUXdEOVVoejh4TCthTkVRRXBDMzgwSUtzMzZiVGxqb0JycWdoby8vR3NRPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)
选择钓鱼的认证网页界面：
![option7](http://imglf6.nosdn.127.net/img/c09lVS9TR3YrUGFBUXdEOVVoejh4QWJxUnpTZjNIN3BsNy91b3NISnAzdERDa2lPekJMdmdRPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)
然后fluxion自动启动一系列服务：开启同名热点，开启钓鱼页面，阻截客户端与AP的连接：
![service_start](http://imglf4.nosdn.127.net/img/c09lVS9TR3YrUGFBUXdEOVVoejh4SnBUaFlqQjBVMVF5cGdHbXRNc3N3WWdueDJubEY5dkNRPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)

## 现象
此时我打开手机，发现手机与原来的合法热点已经断开连接并提示验证错误。此时可以看到列表里出现了一个同名热点。在连接上钓鱼热点后，尝试任何联网行为它都将定向到192.168.254.1，我先尝试输入一次错误的密码，它会提示错误(与之前抓到的握手包进行对比校验)；当我输入正确的后，他将悄悄记录密码，然后退出Fluxion相关的所有任务，仿佛什么也没有发生过。
![phone1](http://imglf4.nosdn.127.net/img/c09lVS9TR3YrUGFBUXdEOVVoejh4SlNhSVJSZkIwRnFNYm55NWJkODUxblNvRzRPN2o2M2xRPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)
![phone2](http://imglf6.nosdn.127.net/img/c09lVS9TR3YrUGFBUXdEOVVoejh4RHoveC8rK29zQlVUNEVyQmJGS1g2T01MWEhBdDY4bmRRPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)
![phone3](http://imglf3.nosdn.127.net/img/c09lVS9TR3YrUGFBUXdEOVVoejh4SldISHIwT3JpYkk0NFU1TGlkY1MwWE51ck85bzN6NkZ3PT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)
![phone4](http://imglf6.nosdn.127.net/img/c09lVS9TR3YrUGFBUXdEOVVoejh4UFo2Q2VWOTc4ZXRpTjNmL2N5NFhYN2JGL3pMZVJlVEtRPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)

回到fluxion，可以看到这边已经得到密码。

其实通过这种方法已经无关乎密码的复杂程度了，我这里虽然使用的是最简单的密码，但密码更复杂些，若用户没有一点防范之心它一样会乖乖到我这边来。
![key_found](http://imglf3.nosdn.127.net/img/c09lVS9TR3YrUGFBUXdEOVVoejh4Q2k0TG1ENXZod2hDQUZGcVBIUFBhR0pqTkNxU05aRE5nPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)
![exit](http://imglf4.nosdn.127.net/img/c09lVS9TR3YrUGFBUXdEOVVoejh4TVV3MVZBZVplUFFZRTFMREpCNG9TY1hnWUpvRTdobld3PT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)

## 感想
Fluxion这个工具不同于Airecrack和Reaver，它是通过社工的方法得到密码。总的来说这是个相当方便的一键脚本，完全傻瓜式的操作，针对没有相关防范意识的用户是非常快捷且有效的攻击方式。

