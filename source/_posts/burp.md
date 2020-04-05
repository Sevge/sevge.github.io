---
title: Burp Suite模块之Lntruder暴力破解网页登陆
date: 2017-10-21 12:36:18
categories:
- 网络
---
## 发现
本学期升级的校园网，网页登陆的认证端很是简陋，不需要验证码，明文传输
![HUNAU](http://imglf5.nosdn.127.net/img/c09lVS9TR3YrUGJTN3BjTUdoQTZLTEtKcFFWc1cvYnVmc3lxaEluZjVaa2I3Q3I4MGtPemt3PT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)
![Auth_Info](http://imglf3.nosdn.127.net/img/c09lVS9TR3YrUGJTN3BjTUdoQTZLTHIwUlc2Nm5Bc0t1ZUdLQ3d6NUdlbWU4V01OZngvVHdRPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)
而且，新升级的网络，默认密码都是由身份证后6位组成，
于是可以使用之前所说的Crunch生成的对应字典来玩玩。

这里以我自己的账号作为测试。

## 配置环境
![BurpSet](http://imglf6.nosdn.127.net/img/c09lVS9TR3YrUGJTN3BjTUdoQTZLTkRUVzk3cmtVZ2ZqQWVuL3M0VmY4V1FUSTlFaHNnWll3PT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)
![FirefoxSet](http://imglf5.nosdn.127.net/img/c09lVS9TR3YrUGJTN3BjTUdoQTZLTDlYTFBNaFM2VWJycU1RZit0NTUwejI0ZWhkZzN6MEdRPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)

## 抓取
在跳转的登陆界面
`http://10.100.0.12:9090/zportal/loginForWeb?wlanuserip=37c61a32243725e8412223107e8670d4&wlanacname=00aab905808bf54238202dd3074e226b&ssid=99f34848c4e3872f&nasip=3a55a6e233ce66a3e3c9d19c2572b2ea&snmpagentip=&mac=e2483bb22f79a96b0b178e83ca255d91&t=wireless-v2&url=63651eaa103df95e80e6576b018a1055&apmac=&nasid=00aab905808bf54238202dd3074e226b&vid=6ff7431ed4e21b22&port=e2bcde16e9a8b04a&nasportid=a25b45948c15af40d095454b31cfa807fcfd7ec39a63f462933adb33ce566a2a
`
中抓取到：
![CatchInfo](http://imglf3.nosdn.127.net/img/c09lVS9TR3YrUGJTN3BjTUdoQTZLTHIwUlc2Nm5Bc0t1ZUdLQ3d6NUdlbWU4V01OZngvVHdRPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)

## 使用
选中一下黄色标记的字符，右键Send to lntruder：
![Burp_case1](http://imglf6.nosdn.127.net/img/c09lVS9TR3YrUGJTN3BjTUdoQTZLSGE0b0tZdDRLSnF3anl4T2lvK05MRS80d1FSYTdEVUpRPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)
可以看到Lntruder对应的选项卡变为高亮，点击Lntruder选项卡:
![Burp_case2](http://imglf4.nosdn.127.net/img/c09lVS9TR3YrUGJTN3BjTUdoQTZLSkFlZ0dmWm5oeGNLblZlMnpKL3JXRkJzNW9jQmNScUdBPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)
点击进入Positions选项卡，看到下方有15个payload，我们不需要这些，点击Clear，然后下方编程0payload：
![Burp_case3](http://imglf6.nosdn.127.net/img/c09lVS9TR3YrUGJTN3BjTUdoQTZLTXNWTTB2U2pEaExJUWFHRGFtMTJNZzdMdlZSVDR5M01BPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)
翻下去找到我们需要的变量，选中“12121”点击右侧Add，可以看到下方显示1payload：
![Burp_case4](http://imglf4.nosdn.127.net/img/c09lVS9TR3YrUGJTN3BjTUdoQTZLTjhsUUFCaEJsdXZTcFpzRVRGU1VUMUkrN3hnSDZlL0NRPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)
点击payload选项卡，payload type我选择跑字典：
![Burp_case5](http://imglf6.nosdn.127.net/img/c09lVS9TR3YrUGJTN3BjTUdoQTZLRWVwbDhhQWpFTFAyRmR5NzIrd3dLTnVHUGVNV29uYzZnPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)
接着option选项卡里，修改线程为10,可以根据实际网络情况加大。有些网站可能会有保护措施，重复登录多次后会封IP，但明显这个认证端没有这个保护
![Burp_case6](http://imglf3.nosdn.127.net/img/c09lVS9TR3YrUGJTN3BjTUdoQTZLSXZUVVMwT2tvQm5SSGFNTVV1YkpHM05qd0lBekRqbStnPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)
最后点击最上方的选项卡lntruder的start attck：
![Burp_case7](http://imglf5.nosdn.127.net/img/c09lVS9TR3YrUGJTN3BjTUdoQTZLQUpsbkcxbWsvNVdXWHh5YmtJa1pZN0ZQbnQvZDhydFhRPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)
接下来就是跑字典的过程了，我自定义的身份证后6位密码有310000组:
![Burp_case8](http://imglf5.nosdn.127.net/img/c09lVS9TR3YrUGJTN3BjTUdoQTZLSWV2Sk5BRVRYYzZxREJKLyswei94bHhZbEdnSmIyaCtBPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)
等待一段时间后，点击length排序（登陆密码错误时和登陆成功后的response包不同，length必然有差别）。注意到response里结果为成功，找到密码为“297410”：
![Burp_Success](http://imglf4.nosdn.127.net/img/c09lVS9TR3YrUGJTN3BjTUdoQTZLSWV2Sk5BRVRYYzZiQWRLczFHREV3Sm1DbUZ2THpZY0h3PT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)
当然，"Intruder"只是Burpsuite其中的一个模块，它的功能和用处远远不止这些。

以上测试基于我的个人账号进行，切勿用于不当用途！
