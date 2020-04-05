---
title: '这是我在Hexo的第一篇文章！Hexo+Gitpage搭建日记'
date: 2017-09-07 13:05:17
categories: 
- 前端
---
## 部署
操作还是挺简单的，具体参照: [click here](https://blog.csdn.net/csdn_yudong/article/details/70837277)
重复的轮子就不造了,感谢原作者"Yu丶"的详尽教程

## 踩过的坑
这其中必须得提一下部署过程中踩过的坑，纠结了很久。。
前面的搭建环境都没有问题，到最后上传部署到gitpage时，
运行`hexo d`没有任何提示。就像这样:
![error](https://s1.ax1x.com/2018/03/22/9H5mOP.png)
就是这个东西！deploy选项下面，
在配置项的前面必须有两个空格，冒号后面必须有一个空格！
![space](https://s1.ax1x.com/2018/03/22/9H50kF.png)
所以说，格式非常重要，必须一丝不苟地对待...
还有注意本地部署三部曲和部署三部曲：
```
$ hexo clean
$ hexo g
$ hexo s --debug(hexo d)
```

## 主题
在Github上搜索了相关的主题，发现Indigo这个样式挺合我意的。
果断使用它了！[click here](https://github.com/yscoder/hexo-theme-indigo/wiki)
感谢"yscoder"大神的无私奉献，让我这样的小白也能用上漂亮的主题！
根据以上文档进行自定义修改：
### 站点配置
```
# Hexo Configuration
## Docs: https://hexo.io/docs/configuration.html
## Source: https://github.com/hexojs/hexo/

# Site
title: Sevge's Blog
subtitle: 
description: My Blog
author: Sevge
language: zh-CN
timezone:

# URL
## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
url: https://sevge.github.io/about/
root: /
permalink: :year/:month/:day/:title/
permalink_defaults:

# Directory
source_dir: source
public_dir: public
tag_dir: tags
archive_dir: archives
category_dir: categories
code_dir: downloads/code
i18n_dir: :lang
skip_render:

# Writing
new_post_name: :title.md # File name of new posts
default_layout: post
titlecase: false # Transform title into titlecase
external_link: true # Open external links in new tab
filename_case: 0
render_drafts: false
post_asset_folder: false
relative_link: false
future: true
highlight:
  enable: true
  line_number: true
  auto_detect: false
  tab_replace:
  
# Home page setting
# path: Root path for your blogs index page. (default = '')
# per_page: Posts displayed per page. (0 = disable pagination)
# order_by: Posts order. (Order by date descending by default)
index_generator:
  path: ''
  per_page: 10
  order_by: -date
  
# Category & Tag
default_category: uncategorized
category_map:
tag_map:

# Date / Time format
## Hexo uses Moment.js to parse and display date
## You can customize the date format as defined in
## http://momentjs.com/docs/#/displaying/format/
date_format: YYYY-MM-DD
time_format: HH:mm:ss

# Pagination
## Set per_page to 0 to disable pagination
per_page: 7
pagination_dir: page

# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: indigo

feed:
  type: atom
  path: atom.xml
  limit: 0

jsonContent:
  meta: false
  pages: false
  posts:
    title: true
    date: true
    path: true
    text: true
    raw: false
    content: false
    slug: false
    updated: false
    comments: false
    link: false
    permalink: false
    excerpt: false
    categories: false
    tags: true

# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repo: git@github.com:Sevge/sevge.github.io.git
  branch: master
```
### 主题配置
```
# hexo-theme-indigo
# https://github.com/yscoder/hexo-theme-indigo

# 添加新菜单项遵循以下规则
# menu:
#  link:               fontawesome图标，省略前缀，本主题前缀为 icon-，必须
#    text: About       菜单显示的文字，如果省略即默认与图标一致，首字母会转大写
#    url: /about       链接，绝对或相对路径，必须。
#    target: _blank    是否跳出，省略则在当前页面打开
menu:
  home:
    text: 主页
    url: /
  archives:
    text: 归档
    url: /archives
#  tags:
#    text: 标签
#    url: /tags
  th-list:
    text: 分类
    url: /categories
#  github:
#    url: https://github.com/sevge
#    target: _blank
  link:
    text: 关于
    url: /about

# 你的头像url
avatar: /img/avatar.jpg
# avatar link
avatar_link: /
# 头像背景图
brand: /img/brand.jpg
# favicon
favicon: /favicon.ico

# email
email: sevge6582@gmail.com

# 设置 Android L Chrome 浏览器状态栏颜色
color: '#3F51B5'

# 页面标题
tags_title: Tags
archives_title: Archives
categories_title: Categories

# 文章截断
excerpt_render: false
excerpt_length: 200
excerpt_link: 阅读全文...
mathjax: false
archive_yearly: true

# 是否显示文章最后更新时间
show_last_updated: false

# 是否开启分享
share: true

# 是否开启打赏，关闭 reward: false
reward: false
#  title: 谢谢大爷~
#  wechat: /img/wechat.jpg     #微信，关闭设为 false
#  alipay: /img/alipay.jpg     #支付宝，关闭设为 false

# 是否开启搜索
search: true

# 是否大屏幕下文章页隐藏导航
hideMenu: true

# 是否开启toc
# toc: false
toc:
  list_number: true  # 是否显示数字排序

# 文章页留言内容，hexo中所有变量及辅助函数等均可调用，具体请查阅 hexo.io
postMessage: 
#这里可以写作者留言，标签和 hexo 中所有变量及辅助函数等均可调用，示例：<a href="<%- url_for(page.path).replace(/index\.html$/, '') %>" target="_blank" rel="external"><%- page.permalink.replace(/index\.html$/, '') %></a>

# 站长统计，如要开启，输入CNZZ站点id，如 cnzz: 1255152447
cnzz: false

# 百度统计，如要开启，改为你的 key
baidu_tongji: false

# 腾讯分析，如要开启，输入站点id
tajs: false

# google
google_analytics: false
google_site_verification: false

# less
less:
  compress: true
  paths:
    - source/css/style.less

# 以下评论插件开启一个即可
# 是否开启 disqus
disqus_shortname: false
# 是否开启友言评论, 填写友言用户id
uyan_uid: false
# 是否使用 gitment，https://github.com/imsun/gitment
gitment: false
# gitment:
#   owner:
#   repo:
#   client_id:
#   client_secret:

# Valine Comment system. https://valine.js.org
valine:
  enable: false # 如果你想使用valine，请将值设置为 true
  appId:  # your leancloud appId
  appKey:  # your leancloud appKey
  notify: false # Mail notify
  verify: false # Verify code
  avatar: mm # Gravatar style : mm/identicon/monsterid/wavatar/retro/hide
  placeholder: Just go go # Comment Box placeholder
  guest_info: nick,mail,link # Comment header info
  pageSize: 10 # comment list page size

# 是否开启Hyper Comments，填写id则启用，false则禁用。http://hypercomments.com
# Hyper Comments support. Write your id here, or false to disable
hyper_id: false


# 规范网址
# 让搜索引擎重定向你的不同域名、不同子域、同域不同目录的站点到你期望的路径
# https://support.google.com/webmasters/answer/139066
# 假设配置为 canonical: http://imys.net，那么从搜索引擎中 www.imys.net 进入会重定向到 imys.net
canonical: false

# 版权起始年份
since_year: 2017

# 用户页面中作者相关的描述性文字，如不需要设为 false
about: 只是一只奔跑的柯基啦(> ~ <)

# “不蒜子”访问量统计，详见 http://ibruce.info/2015/04/04/busuanzi/
visit_counter: false
#  site_uv: 站点总访客数：
#  site_pv: 站点总访问量：

# 动态定义title
title_change:
  normal: (つェ⊂)咦!又好了!
  leave: 死鬼去哪里了！

# 设置为 true 发布后将使用 unpkg cdn 最新的主题样式
# 如果想让你的自定义样式生效，把此项设为 false
cdn: false

# 设置为 true 将使用 lightbox render 图片
lightbox: true

# icp备案号  ICP_license: 京ICP备1234556号-1
ICP_license: false
```
### 主题配色
由于个人喜欢低调的灰色，参照[Material Design Color Palette Generator](https://www.materialpalette.com/)将站点默认的靛蓝修改成灰色：
编辑`theme/indigo/source/css/_partial/variable.less`,更改对应的颜色变量
```
@darkPrimaryColor: #616161;
@primaryColor: #9e9e9e;
@lightPrimaryColor: #f5f5f5;
@textPrimaryColor: #212121;
@accentColor: #536dfe;
@primaryTextColor: #212121;
@secondaryTextColor: #757575;
@dividerColor: #bdbdbd;
@borderColor: #dadada;
@backColor: #f6f6f6;
@codeBg: #f5f5f5;
```

## 总结
完成！
不管怎么说，作为菜鸟的我也开始了写博客的旅程。

今后还请多多指教了！
