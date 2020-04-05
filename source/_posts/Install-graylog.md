---
title: RASPBIAN下从0部署GRAYLOG及其环境
date: 2018-03-20 11:36:40
categories:
- 树莓派
---
## 前言
虽然在树莓派上安装的同样是Linux系统，但是官方文档基本都是基于AMD64的操作，在部署过程中遇到了一些麻烦无从解决。期间折腾了很多，还尝试通过docker安装，甚至还为此专门看了一遍docker文档。。。

不多废话，从0开始记录一下这中间正确的过程。

## 准备工作
使用"SD FORMATTER"格式化sd卡；"WIN32 DISK IMAGEER"烧录官方系统RASPBIAN；完成后，在SD卡(boot)根目录下新建文本文件，重命名为ssh(无后缀)；拔出sd卡，放入树莓派中启动。我这边树莓派通过网线接入了无线路由器，电脑直接通过ssh连接：(pi,raspberry)
![x-shell](http://imglf5.nosdn.127.net/img/c09lVS9TR3YrUGJ6Wm1EWFlGdmJ6UkFWTzhYRkxRRVEvYzJFbnZZaXFsTXVaU1A4VmdtT3BBPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)
习惯性设置：
```
pi@raspberrypi:~ $ sudo passwd root
Enter new UNIX password: 
Retype new UNIX password: 
passwd: password updated successfully
pi@raspberrypi:~ $ su root
Password:
root@raspberrypi:/home/pi# sed -i 's|mirrordirector.raspbian.org|mirrors.ustc.edu.cn/raspbian|g' /etc/apt/sources.list
root@raspberrypi:/home/pi# sed -i 's|archive.raspbian.org|mirrors.ustc.edu.cn/raspbian|g' /etc/apt/sources.list
root@raspberrypi:/home/pi# apt-get update
root@raspberrypi:/home/pi# apt-get install vim
```

树莓派ram只有1g，要安装并使用Graylog的话内存有点欠， `vim /etc/dphys-swapfile`, 将默认为  `CON_SWAPSIZE= 100 M`的交换空间，  更改为` 1024 M`。重新启动dphys-swapfile 文件服务：
```
root@raspberrypi:/home/pi# vim /etc/dphys-swapfile 
root@raspberrypi:/home/pi# /etc/init.d/dphys-swapfile restart
[ ok ] Restarting dphys-swapfile (via systemctl): dphys-swapfile.service.
root@raspberrypi:/home/pi# free -m
              total        used        free      shared  buff/cache   available
Mem:            927          84         387          12         454         776
Swap:          1023           0        1023
```
![swap](http://imglf6.nosdn.127.net/img/c09lVS9TR3YrUGJ6Wm1EWFlGdmJ6WDZsSDF3cFgwOExXcmhJY2I0eWg4eUpxWFhUUjg0OHdBPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)

最后，在bashrc中添加一条别名记录`alias ls=’ls –color’`
```
root@raspberrypi:/home/pi/software# vim ~/.bashrc 
root@raspberrypi:/home/pi/software# exit
root@raspberrypi:/home/pi/software# su root
```

## 安装依赖
官方文档给出的依赖如下：
![requirements](http://imglf4.nosdn.127.net/img/c09lVS9TR3YrUGJ6Wm1EWFlGdmJ6UlY2VzZubnJ6YXVTTnhkZDlUUUJHSzUrQ2diNzY1WkVBPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)

经过测试，树莓派仓库里的mongodb版本为2.4可以直接使用，jdk也可以通过仓库安装，但是elasticsearch版本达不到标准。

### 安装JAVA
在安装时遇到一个错误：
```
root@raspberrypi:/home/pi# apt-get install openjdk-8-jre
Error: missing `server' JVM at `/usr/lib/jvm/java-8-openjdk-armhf/jre/lib/arm/server/libjvm.so'.
Please install or use the JRE or JDK that contains these missing components.
E: /etc/ca-certificates/update.d/jks-keystore exited with code 1.
done.
Setting up libatk-wrapper-java (0.33.3-13+deb9u1) ...
Processing triggers for hicolor-icon-theme (0.15-1) ...
dpkg: dependency problems prevent configuration of openjdk-8-jre-headless:armhf:
 openjdk-8-jre-headless:armhf depends on ca-certificates-java; however:
  Package ca-certificates-java is not configured yet.

dpkg: error processing package openjdk-8-jre-headless:armhf (--configure):
 dependency problems - leaving unconfigured
dpkg: dependency problems prevent configuration of openjdk-8-jre:armhf:
 openjdk-8-jre:armhf depends on openjdk-8-jre-headless (= 8u151-b12-1~deb9u1); however:
  Package openjdk-8-jre-headless:armhf is not configured yet.

dpkg: error processing package openjdk-8-jre:armhf (--configure):
 dependency problems - leaving unconfigured
Setting up libatk-wrapper-java-jni:armhf (0.33.3-13+deb9u1) ...
Processing triggers for libc-bin (2.24-11+deb9u3) ...
Errors were encountered while processing:
 ca-certificates-java
 openjdk-8-jre-headless:armhf
 openjdk-8-jre:armhf
E: Sub-process /usr/bin/dpkg returned an error code (1)
```
谷歌一下，在[树莓派官方论坛](https://www.raspberrypi.org/forums/viewtopic.php?t=197824)找到解决方案：
```
sudo apt-get purge openjdk-8-jre-headless
sudo apt-get install openjdk-8-jre-headless
sudo apt-get install openjdk-8-jre
sudo apt-get install openjdk-8-jdk
```
** 2018.06.06 **
在之后的过程中发现一个问题，树莓派每次开机启动WLAN0、WLAN1、WLAN2命名是不确定的。
树莓派板载无线网卡不支持监控模式，USB网卡配置文件中设置使用固定的无线网卡名WLAN0、WLAN1，混乱的命名就导致了整个服务无法正常启动。
于是，还是得将板载无线网卡的默认名称修改，谷歌上也有个[相同问题](https://raspberrypi.stackexchange.com/questions/63749/how-do-you-unconfuse-raspbian-when-it-has-wlan0-and-wlan1-reversed?utm_medium=organic&utm_source=google_rich_qa&utm_campaign=google_rich_qa)的童鞋,最终解决方法如下：
新建一个文件`/etc/udev/rules.d/70-my_network_interfaces.rules`，其中`ATTR`、`NAME`根据实际修改，文件内容如下：
```
# Built-in wifi interface used in hostapd - identify device by MAC address
SUBSYSTEM=="net", ACTION=="add", ATTR{address}=="b8:27:eb:01:02:03", NAME="WlanBorad"
```

### 安装MONGODB
直接通过仓库安装：
```
apt-get install mongodb
root@raspberrypi:/home/pi# systemctl enable mongodb
Failed to enable unit: File mongo.service: No such file or directory
root@raspberrypi:/home/pi# systemctl enable mongodb
Synchronizing state of mongodb.service with SysV service script with /lib/systemd/systemd-sysv-install.
Executing: /lib/systemd/systemd-sysv-install enable mongodb
root@raspberrypi:/home/pi# systemctl start mongodb
```
安装后测试一下是否可用：
```
root@raspberrypi:/home/pi/software/elasticsearch-5.5.2/bin# mongo
MongoDB shell version: 2.4.14
connecting to: test
Server has startup warnings: 
Fri Mar 16 12:13:53.806 [initandlisten] 
Fri Mar 16 12:13:53.806 [initandlisten] ** NOTE: This is a 32 bit MongoDB binary.
Fri Mar 16 12:13:53.806 [initandlisten] **       32 bit builds are limited to less than 2GB of data (or less with --journal).
Fri Mar 16 12:13:53.806 [initandlisten] **       See http://dochub.mongodb.org/core/32bit
Fri Mar 16 12:13:53.819 [initandlisten] 
```

### 安装ELASTICSEARCH
上面提到，目前最新版本的graylog2.4.3还不支持elasticsearch 6.x，这里选择下载安装5.x版本。
这里参考了[简书](https://www.jianshu.com/p/05052dfc21f6)"水车3060"的源码安装方式。
```
root@raspberrypi:/home/pi# mkdir software
root@raspberrypi:/home/pi# cd software/
root@raspberrypi:/home/pi/software# wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-5.5.2.tar.gz
root@raspberrypi:/home/pi/software# tar -xzvf elasticsearch-5.5.2.tar.gz
```
修改`/elasticsearch-5.5.2/config`目录下的配置文件：`elasticsearch.yml`和`jvp.option`：
![cluster_name](http://imglf3.nosdn.127.net/img/c09lVS9TR3YrUGJ6Wm1EWFlGdmJ6VUgxWmtSNENWekVpZytCUFJzTEI5TVAvM25Dc212TVlRPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)
![XMxs](http://imglf6.nosdn.127.net/img/c09lVS9TR3YrUGJ6Wm1EWFlGdmJ6ZEhCVmp2amJJSmxhUmpaUVpxUWVUUHJqdW51Z2k0TDFRPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)
```
root@raspberrypi:/home/pi/software/elasticsearch-5.5.2# groupadd elsearch
root@raspberrypi:/home/pi/software/elasticsearch-5.5.2# useradd elsearch -g elsearch
root@raspberrypi:/home/pi/software# chown -R elsearch:elsearch  elasticsearch-5.5.2
root@raspberrypi:/home/pi/software/elasticsearch-5.5.2# cd bin/
root@raspberrypi:/home/pi/software/elasticsearch-5.5.2/bin# ./elasticsearch&
```
安装后测试一下是否可用：
```
root@raspberrypi:/home/pi/software/elasticsearch-5.5.2/bin# curl 127.0.0.1:9200
{
  "name" : "dEcU6MS",
  "cluster_name" : "graylog",
  "cluster_uuid" : "1XH8EGmsRC-NIChy7aRkmQ",
  "version" : {
    "number" : "5.5.2",
    "build_hash" : "b2f0c09",
    "build_date" : "2017-08-14T12:33:14.154Z",
    "build_snapshot" : false,
    "lucene_version" : "6.6.0"
  },
  "tagline" : "You Know, for Search"
}
```

## 安装和配置GRAYLOG
参照[官方文档](http://docs.graylog.org/en/2.4/pages/installation/manual_setup.html)，下载tarball：
```
$ tar xvfz graylog-VERSION.tgz
$ cd graylog-VERSION
```
![graylog_conf1](http://imglf3.nosdn.127.net/img/c09lVS9TR3YrUGJ6Wm1EWFlGdmJ6U01OU3NrYXp5RHhvbm1tVVVSYUNtYkFYT2VveENXYnd3PT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)
![graylog_conf2](http://imglf6.nosdn.127.net/img/c09lVS9TR3YrUGJ6Wm1EWFlGdmJ6ZUpOZXdXVi8rUUZQTnRkWktIM012SkRCTUNPaTRqdXRRPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)
根据官方给出的示例，自己配置一下：
> password_secret =
> root_password_sha2 =
> root_timezone = Asia/Shanghai
> rest_listen_uri =  http://0.0.0.0:9000/api/
> web_listen_uri = http://0.0.0.0:9000/
> allow_highlighting = true （运行查询结果高亮）
> elasticsearch_shards = 1 （当前只安装了一个elasticsearch）
> elasticsearch_index_prefix = graylog

```
root@raspberrypi:/home/pi/software/graylog-2.4.3# cd bin/
root@raspberrypi:/home/pi/software/graylog-2.4.3/bin# ./graylogctl start
```

## 完成
等待一段时间(以树莓派的CPU，你懂得)，可以使用浏览器打开`graylog-server-address`(192.168.0.177)页面管理日志了。
![getting_started](http://imglf3.nosdn.127.net/img/c09lVS9TR3YrUGFPeFZ3QW9IMkYwRHVwQUpkM1JGeFVKNmVjVnZOWGFLTnlSeDUxazZEM21nPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)
不过，使用一些天以后发现，部署Graylog及其环境运行在树莓派下，就像大象骑在蚱蜢身上，真的很慢很慢，HAH。。。。。。。