---
title: 图说CDN
date: 2018-10-04 17:00:27
categories:
- 收藏
---
#### 什么是CDN?
　　618电商节、双11购物狂欢节，到底是什么在支撑数以万计的秒杀活动？这就不得不提一直隐姓埋名的 CDN 了，其全称是 Content Delivery Network，即内容分发网络。 
　　那到底 CDN 是什么鬼，我们还得从西天取经说起……
![0](http://imglf5.nosdn0.126.net/img/c09lVS9TR3YrUFkyTmVGdU5NMVF4Vkc0UkcxRHEyd2JiOGFLMHMwd0pFdlBHK2E0Q1QwSEJRPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)

#### 背景
　　1300年前，唐僧师徒取经要跋涉十万八千里，历经九九八十一难，一路打怪升级，最终才能修成正果，悟空加冕“斗战胜佛”。 
　　1300年后，西游互联网已经开通，雷音寺官网上线，取经只需打开网站，点击下载，凡夫俗子也可以轻易取得真经。
![1](http://imglf3.nosdn0.126.net/img/c09lVS9TR3YrUFkyTmVGdU5NMVF4Vkc0UkcxRHEyd2Jqc0xROWlHWHAxd0FoVzg2aTdYQld3PT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)
　　初时，唐僧师徒觉得当年的辛苦付出颇为不值，慨叹世事变迁，法术高强敌不过科技进步。 
然四大部洲善男信女众多，扎堆前往雷音寺官网下载经书，网站不堪重负，信徒叫苦不迭，神通广大的如来使出“Scacleup + Scaleout”心法，扩容雷音官网，仍不得其解，遂差遣悟空一查究竟。

#### 八十一难
　　悟空火眼金睛，半晌就把原因查了个一清二楚，原来信徒要想美美的访问雷音网，需要打败四个妖怪：
##### 第一怪，首里魔
![首里魔](http://imglf4.nosdn0.126.net/img/c09lVS9TR3YrUFkyTmVGdU5NMVF4UmNkNXdxbHBveFZ5V2FacUJLbzk0T29HWk4yblZ0VW9nPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)
　　“首里魔”又称“第一公里魔”，把持网站服务器接入西游互联网的路口带宽，这个带宽决定了能为信徒提供的访问速度和并发访问量。

##### 第二怪，骨干精
![骨干精](http://imglf4.nosdn0.126.net/img/c09lVS9TR3YrUFkyTmVGdU5NMVF4VjRTcEw0Q2JiUHdGN2pGOUw3SHIyOHhkWXk5b3NhTHhnPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)
　　“骨干精”藏于西游互联网的长途传输要道，出没于IDC、骨干网、城域网、接入网等洞穴，使用“时延”和“拥塞”两个妖术作法。

##### 第三怪，互联妖
![互联妖](http://imglf5.nosdn0.126.net/img/c09lVS9TR3YrUFkyTmVGdU5NMVF4UXR0eVRNVXM1V0M2N29Od2M0by9EZTFOWHg4TFBhdS9nPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)
　　西游互联网覆盖四大部洲，各部洲的网络独立运营，“互联妖”善于挑拨离间，让洲与洲之间的互联带宽成为瓶颈。

##### 第四怪，末里兽
![末里兽](http://imglf4.nosdn0.126.net/img/c09lVS9TR3YrUFkyTmVGdU5NMVF4UzNMeVZ2Z3VpTXBGcVI4MEswQjBKNzJuT3lmMTR1eVZBPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)
　　“末里兽”又称“最后一公里兽”，蹲守在上网信徒的家门口，把持用户访问西游互联网的通路，收取买路钱，钱少只能走羊肠小道。 

#### 悟空显神通
　　悟空看罢大怒，原来是这些妖孽作怪！ 
　　于是拔下一根毫毛，使出“CDN”大法，变作几百只小猴子，一声令下，每猴背熟一些经文，纷纷潜入到各大部洲的 IDC 山洞中，就近为善男信女们提供讲经服务，这些小猴子被俗称为“cache猴”。
　　小猴子们基于这样的规则干活： 
A．当某个信徒需要阅读经书，大家就挑选能最快到达信徒家的猴子前去讲经（可能距离最近，也可能是路最好走）； 
B．如果某部经书被很多信徒需要，它就会被距离这些信徒最近的小猴子烂熟于心。
　　可是猴子很多又生性顽劣，管好还是很费神的，于是悟空叫来了师父和师弟们帮忙，师徒同心，其利断金。
##### 沙僧
　　沙和尚任劳任怨，悟空让他承担“分发服务”：
![沙僧](http://imglf3.nosdn0.126.net/img/c09lVS9TR3YrUFkyTmVGdU5NMVF4WC9DaGF1eWh3a0xtbmIzeVdKQXg1ZnBaa1dxaU1oVUN3PT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)
　　老沙的主要职责是将经书内容从雷音寺中心向各部洲的“cache猴”推送和存储，承担实际的佛经流量全网分发工作和面向最终信徒的阅读请求服务。
##### 八戒
　　猪八戒肠肥肚圆，悟空让他承担“负载均衡”：
![八戒](http://imglf4.nosdn0.126.net/img/c09lVS9TR3YrUFkyTmVGdU5NMVF4YjE4V3pNQjE1UnhCSkZqRkN0WHhUa0pzNzFET2swcTVRPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)
　　八戒负责对所有发起阅经请求的信徒进行访问调度，确定提供给信徒的最终实际访问地址，告诉信徒那个小猴子最适合他。
##### 三藏
　　唐三藏高瞻远瞩，悟空请他承担“运营管理”：
![三藏](http://imglf4.nosdn0.126.net/img/c09lVS9TR3YrUFkyTmVGdU5NMVF4VlpWUHQxY2tZbGY1cVk4YWJkVGNZVldrajBQQlYwaFB3PT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)
　　唐僧负责对日常事务的监管、收支核算、团队状态的检查、分析，也承担与大客户–佛祖“疏通”关系等职责。 

#### 美满结局
　　在师徒四人的通力合作下，四个妖怪被打败，如来佛祖的心病治愈了，天下苍生得以美美滴上网取经。 
　　雷音寺赚得盆满钵满，不断推出新的服务，原来只有经书下载，现在可以在线浏览经书，还可以视频直播，观看佛祖在线讲经。 
　　于是唐僧师徒的 CDN 服务从原来只提供文件传输加速服务，到后来增加为流媒体加速服务、网页浏览加速服务等等。 
　　从此，天下再没有难取的经，悟空得到佛祖嘉奖，从“斗战胜佛”升级为“斗站胜佛”！

#### 完
　　好了，西游记的故事讲完了，小伙伴们也明白什么是 CDN 了。

#### 关于
　　CSDN上看到的一篇很有趣的文章，用西天取经的故事生动形象地展示了CDN的那些事。
　　个人觉得写的非常好，遂搬运。如涉侵权，请告知。

