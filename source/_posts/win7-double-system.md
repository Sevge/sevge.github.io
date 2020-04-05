---
title: DXF客户端及环境部署过程
date: 2018-05-15 16:22:30
categories:
- WINDOWS
---
## 前言
最近自己搭建了一个毒奶粉私服玩，服务器搭建过程可以看[这里](http://note.youdao.com/noteshare?id=7c1896fcc95e48a8607a7bf79915c155)。
客户端搭建也需要一定折腾，所以不想折腾的童鞋可以点击右上角的"X"了。

## 配置环境
受WIN8/10某些系统补丁的影响，客户端本体只能在WIN7环境下才能正常运行。因此，第一步就是准备游戏所需的基本环境——WIN7系统。
鉴于在添加启动项的过程中有一些敏感操作，建议在整个过程开始前将所有安（liu）全（mang）软件关闭。

## 下载软件
目前国内已经有对应的软件能直接一键完成这些任务。以下提供对应的链接和操作过程：
### 下载分区助手
进入[分区助手](https://www.disktool.cn/download.html)下载页面，下载绿色版本，免于安装。
![img1](http://imglf5.nosdn.127.net/img/c09lVS9TR3YrUFlGcGNpOFIzNHZabFFuWEI2TnNVSXRJSXpIa2tGSzYwK0pzK1FJbndmenBRPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)

### 下载软媒魔方
进入[软媒魔方](http://mofang.ruanmei.com/)官网下载绿色版本。
![img2](http://imglf3.nosdn.127.net/img/c09lVS9TR3YrUFlGcGNpOFIzNHZaaXBNZCs4VXBEb3hwc0l3VkZ0V20veU0xbzdrcWRXS1pRPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)

## 分区和装载
下载完成以后，分别将以上得到的压缩包解压到相应的目录。
### 分区
在分区助手的目录下找到"PartAssit"，双击打开。
![img3](http://imglf3.nosdn.127.net/img/c09lVS9TR3YrUFlGcGNpOFIzNHZadkFocEYwRkttRmRCY1NlcHA5ZU9NV2JLVWVYMmNJSGNBPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)
查看界面上磁盘的使用情况，选择一块剩余空间比较大的磁盘。
选中，右键单击选择切割分区。
我选择分给它50G。
![img4](http://imglf5.nosdn.127.net/img/c09lVS9TR3YrUFlGcGNpOFIzNHZaa2YzY0tPcnc3MTlta3VrNyt3MEhxK3hMQWEwZFpZNWtnPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)

### 装载
在软媒魔方的目录下找到"hdboot"，双击打开。
![img5](http://imglf5.nosdn.127.net/img/c09lVS9TR3YrUFlGcGNpOFIzNHZabWZEd1JjdE5wZWQwUldZdXZGR0xhRXRNUk9HcmFOTnlBPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)
按照软件提供的必要参数，选择镜像文件，推荐MSDN吧提供的[WIN7旗舰版](http://tieba.baidu.com/p/3946202816)。
镜像解压位置选择刚刚分区分出来的一块盘。
第三第四选项根据自己需要选择，我的启动项描述为"WIN-7"。
最后，点击开始装机。
**！！！这里需要注意的是，在装机之前，可能需要在BIOS里设置一下SECURE BOOT为DISABLE，BOOT MODE设置为LEGACY AND UEFI BOTH。
这一步很重要，否则在装WIN7的过程中出现很多不可描述的错误！**

## 安装驱动
装上陈旧的WIN7后，一般因为没有自带驱动，声卡、网卡、显卡等设备可能没有正常运行。
推荐先从另外一台设备下载"驱动精灵万能网卡版"，复制到本地安装。
接着，老老实实把需要的驱动都安装上吧。
安装完成后，驱动精灵自动帮我们安装网卡驱动。只要有网络了，一切都好办了。
![img6](http://imglf4.nosdn.127.net/img/c09lVS9TR3YrUFlGcGNpOFIzNHZabmJJcFM2b2xlMi83QUNDZW5xaFlQSWNMWmNrdEpJRlNnPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)

## 安装DNF客户端
前面环境终于搭建好了！看到这里的兄弟给自己点32个赞！！！
到了这一步，说明一切都准备就绪了！接下来的操作就很简单了：
> 下载[客户端文件](https://pan.baidu.com/s/1A5tUCmYcDtzjSqXkWRw1nA)，下载[PVF文件](https://pan.baidu.com/s/1149huNIdUPonCtVz2nclVg)，下载[客户端登录器](https://pan.baidu.com/s/1Ox243M7IabZPbUiNxX2sgg)。
> 解压客户端文件(地下城与勇士10.4G)
> 参照斩龙pvf3.0的使用说明将对应的文件放入游戏对应的文件夹
> 将客户端登录器放入游戏根目录

![img7](http://imglf5.nosdn.127.net/img/c09lVS9TR3YrUFlGcGNpOFIzNHZaczhpQnBpdU5KRG4xUnROU3lubWlMbk8zUXEydU9HOWhBPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)
![img8](http://imglf5.nosdn.127.net/img/c09lVS9TR3YrUFlGcGNpOFIzNHZablk3c0JzTlg5eHl2S3lvZnJqK01MRWJGaEhHL1A1UjZBPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)

## 完成
至此，说明你已经跨越重重险阻。
接下来，打开游戏目录下的"DNFLogin"，享受惬意的游戏吧！

<div id="music163player">
<center>
<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=420 height=86 src="//music.163.com/outchain/player?type=2&id=37240594&auto=1&height=66"></iframe>
</center>
</div>


