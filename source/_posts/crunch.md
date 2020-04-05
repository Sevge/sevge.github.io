---
title: 密码生成工具Crunch的使用
date: 2017-10-07 13:05:17
categories:
- Kali
---
## 前言
很多时候网络上下载的、系统或软件自带的字典效果不尽如人意，这个时候我们可能就需要根据自己的需求生成一个按照我们已经知道的信息来组合的字典。

## 介绍
Crunch是一种创建密码字典工具，该字典常用来暴力破解。使用Crunch工具生成的密码可以发送到终端、文件或者另一个程序。Crunch默认安装在kali环境中，Crunch可以按照指定的规则生成密码字典，生成的字典字符序列可以输出到屏幕、文件或重定向到另一个程序中，Crunch可以参数可能的组合和排列，其最新版本为3.6。并具备如下特征：
> 1. Crunch可以以组合和排列的方式生成字典
> 2. 它可以通过行数或文件大小中止输出
> 3. 支持恢复
> 4. 支持数字和符号模式
> 5. 分别支持大小写字符模式
> 6. 在生成多个文件时添加状态报告
> 7. 新的-l选项支持@，%^
> 8. 新的-d选项可以限制重复的字符，可以通过man文件查看详细信息
> 9. 现在支持unicode

Crunch其实最厉害的是知道密码的一部分细节后，可以针对性的生成字典，这在渗透中就特别有用。

## 使用
现在的KALI中一般自带Crunch。在终端下输入Crunch，执行以上命令后，将输出如下所示的信息：
![Crunch_info](http://imglf3.nosdn.127.net/img/c09lVS9TR3YrUFlidld6eGYrakwxRnpIanBSR1NjbSs2VDhXRHpFclY1YmlxQ082RjlWNHhRPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)
输出的信息显示了Crunch命令版本及语法格式：
``` 
Crunch <min> <max> [options] 
```
### 常用选项
(1) -b 数字[类型] **指定输出文件的大小**，仅仅使用“-o”选项时生效;例如60mb，例如格式： “Crunch 4 5 -b 20mib -o START”会生成4个文件：aaaa-gvfed.txt，gvfee-ombqy.txt，ombqz-wcydt.txt，wcydu-zzzzz.txt，其中每一个文件的开始和最后字符串将作为文件的文件命名;类型有效值为KB、MB、GB、KIB，MIB，和GIB。前三种类型是基于1000，而最后三种类型是基于1024，注意数字与类型之间没有空格。例如“500mb”正确，而“500 MB”则不正确，执行命令后如图所示。aaaa-gvfed.txt，gvfee-ombqy.txt，ombqz-wcydt.txt大小将是20M，以1024为基数，也即20480kb，一般以mib为参数。
![Crunch_option1](http://imglf5.nosdn.127.net/img/c09lVS9TR3YrUFlidld6eGYrakwxRnpIanBSR1NjbSsyRmZiRzJGN1VLR0JTdGtJTlRScytnPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)
(2) -c 数字 指定写入输出文件的行数，也即包含**密码的个数**（行数），例如使用字符规则mixalpha-numeric-all-space，生成最小和最大字符串为1的且每一个文件保存60个字符串的密码字典：
![Crunch_option2](http://imglf3.nosdn.127.net/img/c09lVS9TR3YrUFlidld6eGYrakwxRDdMWTlxcStPSDkvRzdRK0ZqVHF2eUozZFg4Nm8yZEFRPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)
![Crunch_option22](http://imglf4.nosdn.127.net/img/c09lVS9TR3YrUFlidld6eGYrakwxUHJQa3ljNTQ0a3hIbkdVdXBvTFZJWERQMHh0aWVFSXRnPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)
(3) -d 数字符号，**限制出现相同元素的个数**(至少出现元素个数)，“-d 2@”限制小写字母输出像aab和aac，aaa不会产生，因为这是连续3个字母，格式是数字+符号，数字是连续字母出现的次数，符号是限制字符串的字符，例如@,%^(“@”代表小写字母，“,”代表大写字符，“%”代表数字，“^”代表特殊字符)

(4) -e 字符串，**定义停止生成密码**，比如-e 222222：到222222停止生成密码:
![Crunch_option4](http://imglf4.nosdn.127.net/img/c09lVS9TR3YrUFlidld6eGYrakwxRVVqVWRSaDA4cnQ0bURwSlBuSk9EeExkcnJJZkRCTnlBPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)
(5) -f /path/to/charset.lst charset-name，从charset.lst指定字符集，也即**调用密码库文件**，比如kali中的charset.lst 在/usr/share/Crunch/charset.lst，则参数为“-f /usr/share/Crunch/charset.lst”

(6) -o wordlist.txt，**指定输出文件的名称**，例如wordlist.txt

(7) -p 字符串 或者-p 单词1 单词2 ...**以排列组合的方式来生成字典**。
![Crunch_option7](http://imglf5.nosdn.127.net/img/c09lVS9TR3YrUFlidld6eGYrakwxQ2JvQlZCRGovUjR2LzUyMjhOMFhZT0QxY2ZuVTRnYjhnPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)
(8) -q filename.txt，读取filename.txt

(9) -s **指定一个开始的字符**。

(10) -t @,%^，指定模式，@,%^分别代表意义如下：
> 1. @ 插入小写字母
> 2. , 插入大写字母
> 3. % 插入数字
> 4. ^ 插入特殊符号

(11) z gzip, bzip2, lzma, and 7z，从-o选项压缩输出结果，支持gzip, bzip2, lzma, and 7z格式，gzip是最快压缩率最低，bzip2是稍微慢于gzip，但比其压缩率搞，7z最慢，但压缩率最高。

## 实例
### 生成关键字的所有组合
``` 
$ Crunch 9 9 -p wang 1997 0101 
```
### 制作8位密码字典
``` 
$ Crunch 8 8 charset.lst numeric -o num8.dic 
```
### 制作139开头的手机字典
可以每次生成文件大小为20M，自动生成文件：
``` 
$ Crunch 11 11  +0123456789 -t 139%%%%%%%% -b 20mib -o START
```
![Crunch_case1](http://imglf3.nosdn.127.net/img/c09lVS9TR3YrUFlidld6eGYrakwxQWZURXpndFUzYmVtckZtdUh3WTZzeTZ1T3kzUzNxd2VnPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)

## 其他
另外，KAILI还自带了一些字典在/usr/share/wordlists/文件夹下，例如rockyou.txt.gz字典，将字典解压后其实就是一个rockyou.txt文件，里边包含了WPA的常用密码.

## 搭配工具
校园网登陆认证默认使用身份证后6位作为密码
![HUNAU](http://imglf6.nosdn.127.net/img/c09lVS9TR3YrUFlidld6eGYrakwxR3BEUXEvVUU4U1cyNUtZWUtPZGUvZkVKWGtyK2NiM1JRPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)
可以使用Crunch方便地生成需要的字典
![IDpasswd](http://imglf6.nosdn.127.net/img/c09lVS9TR3YrUFlidld6eGYrakwxRkM3L29xRzU4NjZsYzN3bVdmQWY1VFc5U1ROeXZzREZ3PT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)
考虑到还有以X结尾的号码:
![IDpasswdX](http://imglf6.nosdn.127.net/img/c09lVS9TR3YrUFlidld6eGYrakwxTS9oTCtFY2dMODAwYWd5SW9rLzMreTRULzlFWUtZRXpBPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)
然后可以使用Burpsuite...

## 总结
Crunch只是一个生成字典的工具，理论上支持搭配所有暴力破解的工具，比如跑抓到的WIFI握手包，跑压缩包密码等。不管怎么说，以后它将是我如影随形的伙伴了！

[Crunch Me!](https://github.com/Crunchsec/Crunch)
