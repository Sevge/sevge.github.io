---
title: 基于”会动的简历“写给成哥的信
date: 2018-05-22 10:28:41
categories:
- 前端
---

### 0x01
毕业季又悄然来临，
掐指一算，还有14天就高考了。
虽然我早已经过了高考这道槛，
但这也意味着学校即将换届。
成哥是我大一刚入学以来的助班，
助班的职责是带领大一的学弟学妹军训。
但成哥做的远远不止于此，
他在之后的日子也同样关心着我们这个班级，
每年平安夜都将苹果送到每一个同学手中。
我想成哥是所有助班中的唯一了

### 0x02
成哥对我们的好，大家都是记得的。
班级瞒着成哥悄悄准备一场聚会，
并要求每个同学写一些寄语给他。
很多同学是通过书信的方式来留言，
我想，既然是计算机系的，
怎么能用这么老土的方式？

### 0x03
之前在群里看到过一个[会动的简历模板](
http://www.sitexa.org/anires/public/?from=groupmessage&isappinstalled=0),
感觉很有意思。
之后辗转得到了[最初的版本](https://jirengu-inc.github.io/animating-resume/public/),
在github上得到了[源码](https://github.com/jirengu-inc/animating-resume)。
索性打算使用这种方式来把我的想法传达给成哥了。

### 0x04
按照README.md的说明，克隆项目到本地。
修改项目名，修改`config/index.js`第十行的路径名。
修改`src`目录下的`App.vue`和`Mobile.vue`，
之后执行：
```
npm install
npm run dev
```
等待几分钟，本地页面部署完成。
打开`http://localhost:8080`可以看到效果。
接下来得把它部署到公网上去。

### 0x05
最初使用最粗暴的方法，
直接在我的阿里云服务器上安装Git和Nodejs，
然后同本地部署方法一样
```
npm install
npm run dev&
```
这样就能直接通过访问我的[公网IP](http://39.107.244.191:8080)来查看。
之后部署到GiteePage上，
按照说明，需要先本地编译再上传：
```
npm run build
git add .
git commit -m "upup"
git push -u origin master -f
```
等待上传后，可以访问[GiteePage](http://sevge.gitee.io/public/)来查看内容了。

** 这里需要吐槽的是通过WC或QQ把GiteePage分享出去，
腾讯认定为危险网站？？？！！！ **
<div id="music163player">
<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86 src="//music.163.com/outchain/player?type=2&id=362450&auto=1&height=66"></iframe>
</div>
