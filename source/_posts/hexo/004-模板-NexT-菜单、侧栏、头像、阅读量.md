---
title: 0004-模板-NexT-菜单、侧栏、头像、阅读量
categories:
  - hexo
abbrlink: e89588
date: 2020-01-29 20:55:26
tags:
---

Hexo 框架允许我们更换合适的主题，以便于构建不同风格的网站，这里介绍目前最常使用的一款主题之NexT

设置Scheme、设置动态背景、设置侧栏行为、设置菜单、设置头像、添加社交链接、添加文字统计功能、添加阅读量统计功能

<!--more-->
# NexT 安装

几个概念：

在使用 Hexo 框架建立的网站中，存在两份重要的配置文件，它们的文件名称都是 _config.yml

一份是 站点配置文件，位于 站点根目录 下，用于网站的基础配置

另外一份是 主题配置文件，位于 themes 目录 下，用于主题的相关配置

不同的主题会有不同的主题配置文件，由主题作者所提供

## 下载 NexT

在 站点根目录 中打开 git bash 窗口，使用如下命令下载 NexT 主题文件到 themes 目录 中

``` bash
git clone https://github.com/theme-next/hexo-theme-next themes/next
或
git clone https://github.com/iissnan/hexo-theme-next themes/next
或
git clone git://github.com/iissnan/hexo-theme-next themes/next
```

## 启用 NexT
打开 站点配置文件， 将 theme 选项的值改为 next，注意要在属性和值之间要加上一个空格

``` text
# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: next
```
此时，登陆自己的站点，应该可以看到更改已经成功

# NexT 配置

## 设置Scheme
Scheme 是用于 改变网站布局 的一个设置项，NexT 目前提供四种 Scheme：
``` text
Muse ：默认 Scheme，黑白主调，大量留白
Mist：Muse 的紧凑版本，整洁有序的单栏外观
Pisces：双栏 Scheme，小家碧玉的清新
Gemini：新增 Scheme
```
更改时，打开 主题配置文件，通过搜索关键字 Scheme Settings 定位，然后将使用的 scheme 打开注释即可
``` text
# ---------------------------------------------------------------
# Scheme Settings
# ---------------------------------------------------------------

# Schemes
#scheme: Muse
#scheme: Mist
scheme: Pisces
#scheme: Gemini
```

## 设置动态背景
更改时，打开 主题配置文件，通过搜索关键字 Canvas-nest 定位，然后将 canvas_nest 的值改成 true 即可
``` text
# Canvas-nest
canvas_nest: true
```


## 设置侧栏行为
默认情况下，侧栏仅在文章页面（拥有目录列表时）才显示，并放置于右侧位置

可以通过修改 主题配置文件 中的 Sidebar Settings 字段控制侧栏的行为

（1）侧栏位置：position
left：靠左放置;
right：靠右放置;

（2）侧栏显示时机：display
post：默认行为，在文章页面（拥有目录列表时）显示;
always：在所有页面中都显示;
hide：在所有页面中都隐藏;
remove：完全移除
``` text
# Sidebar Position, available value: left | right (only for Pisces | Gemini).
position: left
#position: right

# Sidebar Display, available value (only for Muse | Mist):
#  - post    expand on posts automatically. Default.
#  - always  expand for all pages automatically
#  - hide    expand only when click on the sidebar toggle icon.
#  - remove  Totally remove sidebar including sidebar toggle.
display: post
#display: always
#display: hide
#display: remove
```


## 设置菜单
（1）设置菜单项

打开 主题配置文件，搜索关键字 Menu Settings 进行定位，各个菜单项通过 # 注释开启或关闭
```text
# ---------------------------------------------------------------
# Menu Settings
# ---------------------------------------------------------------

# When running the site in a subdirectory (e.g. domain.tld/blog), remove the leading slash from link value (/archives -> archives).
# Usage: `Key: /link/ || icon`
# Key is the name of menu item. If translate for this menu will find in languages - this translate will be loaded; if not - Key name will be used. Key is case-senstive.
# Value before `||` delimeter is the target link.
# Value after `||` delimeter is the name of FontAwesome icon. If icon (with or without delimeter) is not specified, question icon will be loaded.
menu:
  home: / || home
  #about: /about/ || user
  #tags: /tags/ || tags
  categories: /categories/ || th
  archives: /archives/ || archive
  #schedule: /schedule/ || calendar
  #sitemap: /sitemap.xml || sitemap
  #commonweal: /404/ || heartbeat
```
部分菜单项的功能描述如下：
home：主页;about：关于;tags：标签;categories：分类;archives：归档;

菜单具体 参看002


## 设置头像
打开 主题配置文件， 搜索关键字 Sidebar Avatar 进行定位，将 avatar 的值设置成头像图片的链接地址即可
```text
# Sidebar Avatar
# in theme directory(source/images): /images/avatar.gif
# in site  directory(source/uploads): /uploads/avatar.gif
avatar: <url>
```
头像图片的链接地址可以是：  
- 完整的互联网地址：例如，https://www.example.com/avatar.jpg
- 站点内的相对地址：例如，假设图片命名为 avatar.jpg，存放在 source/images/ 目录下，则链接地址可以写成 /images/avatar.jpg

## 添加社交链接
打开 主题配置文件，搜索关键字 Social Links 进行定位，social 的值按 Key: permalink || icon 格式设置
```text
# Social Links.
# Usage: `Key: permalink || icon`
# Key is the link label showing to end users.
# Value before `||` delimeter is the target permalink.
# Value after `||` delimeter is the name of FontAwesome icon. If icon (with or without delimeter) is not specified, globe icon will be loaded.
social:
  GitHub: https://github.com/Forwhfang || Github
  CSDN: https://blog.csdn.net/wsmrzx || CSDN
  cnblogs: https://www.cnblogs.com/wsmrzx || cnblogs
```


## 添加文字统计功能
进入 站点根目录，打开 git bash 窗口，输入如下命令安装插件
`pm install hexo-wordcount --save`  
然后打开 主题配置文件，进行如下配置
```text
# Post wordcount display settings
# Dependencies: https://github.com/willin/hexo-wordcount
post_wordcount:
  item_text: true
  wordcount: true
  min2read: true
  totalcount: true
  separated_meta: true
```


## 添加阅读量统计功能
在 主题配置文件 中修改 busuanzi_count 字段启用不蒜子统计功能
``` text
# Show PV/UV of the website/page with busuanzi.
# Get more information on http://ibruce.info/2015/04/04/busuanzi/
busuanzi_count:
  # count values only if the other configs are false
  enable: true
  # custom uv span for the whole site
  site_uv: true
  site_uv_header: <i class="fa fa-user"></i>
  site_uv_footer: 
  # custom pv span for the whole site
  site_pv: true
  site_pv_header: <i class="fa fa-eye"></i>
  site_pv_footer: 
  # custom pv span for one page only
  page_pv: true
  page_pv_header: <i class="fa fa-file-o"></i>
  page_pv_footer: 
```
【参考资料】

- [iissnan/hexo-theme-start](http://theme-next.iissnan.com/getting-started.html)
- [iissnan/hexo-theme-next](https://github.com/iissnan/hexo-theme-next/wiki)
- [third-party-services](http://theme-next.iissnan.com/third-party-services.html)
- [hexo.io/plugins](https://hexo.io/plugins/)
