---
title: 006-模板-NexT-三方服务-搜索、统计分析、评论
date: 2020-01-29 23:09:31
tags:
categories: 
- hexo
permalink:
---

包含了 站内搜索、埋点统计等，注意：凡是网络上写着改代码的配置，几乎都不用，一般都有人写好git能直接用npm了

<!--more-->

# 搜索

## 站内搜索

本地搜索不需要任何外部第三方服务，并且可以由搜索引擎额外索引。该搜索方法推荐给大多数用户。

通过在站点根目录中运行以下命令来安装hexo-generator-searchdb：

``` bash 
npm install hexo-generator-searchdb
```
config配置

``` text hexo/_config.yml
search:
  path: search.xml
  field: post
  format: html
  limit: 10000
```

``` text next/_config.yml
# Local search
# Dependencies: https://github.com/theme-next/hexo-generator-searchdb
local_search:
  enable: true
  # If auto, trigger search by changing input.
  # If manual, trigger search by pressing enter key or search button.
  trigger: auto
  # Show top n results per article, show all results by setting to -1
  top_n_per_article: 1
  # Unescape html strings to the readable one.
  unescape: false
  # Preload the search data when the page loads.
  preload: false
```

# 统计和分析

## 1、分析-百度
1. 登录 [百度 分析](https://tongji.baidu.com/web/welcome/login)并找到网站代码获取页面。
2. 将脚本ID复制到hm.js？之后，如下图：  
![avatar](/images/post/hexo/analytics-baidu-id.png)
3. 配置
```text next/_config.yml
# Baidu Analytics ID
baidu_analytics: your_id
```
代码检测是否成功，一般20分钟，可以看浏览器控制台 [代码手工检查攻略](https://tongji.baidu.com/web/help/article?id=93&type=0)

查看百度[统计](https://tongji.baidu.com/web/10000139146/homepage/index)

# 评论

Hexo支持的评论比较多，Disqus、DisqusJS、LiveRe、Gitalk、Valine (China)、Changyan (China)  
支持多个评论：Multiple Comment System Support  
多说和网易云 不做了，其次畅言需要备案  
Disqus，Hypercomments和LiveRe都是国外的，加载速度慢，甚至有被墙的可能，  
valine 账户增加了 短信验证，实名认证等，需要个人信息 太多  
Gitment 基于git的issues,由于 Next 更新，Gitment 已经预置了，所以不需要自己再添加代码。但是 作者 又不更新了，授权比较多,目前gitment.browser.js内使用授权，作者暂不维护  
utterances 版本集成了utterances评论。这一工具原理和GITALK类似，但是索取的权限少，并且不用指定某个人来初始化。*推荐*

- 页面关闭评论
`comments: false`

## utterances *【推荐】*

源码[地址](https://github.com/theme-next/hexo-next-utteranc)
首先来[这里](https://github.com/apps/utterances)为utterances在github上授权。  
只有这样，才能让utterances有资格访问你的issue。还可指定utterances能够访问的仓库，可见其权限控制做的非常好。

授权完毕后，来到博客根目录，打开Git Bash，执行
```bash
npm install --save github:theme-next/hexo-next-utteranc
```
后运行可能缺少依赖next-util ，原因是设置了[淘宝的 npm 源](https://www.cnblogs.com/bjlhx/p/12239748.html)

打开主题配置文件_config.yml
``` text
# Demo: https://utteranc.es/  http://trumandu.github.io/about/
utteranc:
  enable: true
  repo: #Github repo such as :TrumanDu/comments
  pathname: pathname
  # theme: github-light,github-dark,github-dark-orange
  theme: github-light
  cdn: https://utteranc.es/client.js
```

## LiveRe
LiveRe是基于社交网站评论的内容平台，可帮助用户自由交流。

创建一个帐户或登录[LiveRe](http://livere.com)，单击安装按钮并选择免费的城市版本，然后单击立即安装按钮。

复制安装代码中的data-uid字段以获取LiveRe UID。

将获得的LiveRe UID添加到主题配置文件中的livere_uid部分，如下所示

``` text next/_config.yml
# Support for LiveRe comments system.
# You can get your uid from https://livere.com/insight/myCode (General web site)
livere_uid: your_uid
```

## Gitment

[Gitment](https://github.com/imsun/gitment) 是作者实现的一款基于 GitHub Issues 的评论系统。  
支持在前端直接引入，不需要任何后端代码。可以在页面进行登录、查看、评论、点赞等操作，同时有完整的 Markdown / GFM 和代码高亮支持。尤为适合各种基于 GitHub Pages 的静态博客或项目页面。
1. github注册  
首先要有github帐号  
接着注册 [OAuth Application](https://github.com/settings/profile)→[OAuth App](https://github.com/settings/developers)
注册特别简单。之后能够查看 clientId,sercet等
2. 引入gitment
在站点目录下，执行
``` bash
npm install --save gitment
```
打开主题配置文件_config.yml
```text
gitment:
  enable: true # 是否开启gitment评论系统
  mint: true #
  count: true # 是否显示评论数
  lazy: true # 懒加载，设置为ture时需手动展开评论
  cleanly: true # 是否隐藏'Powered by ...'
  language: en # 语言，置空则随主题的语言
  github_user: iamsea # Github用户名
  github_repo: comment # 在Github新建一个仓库用于存放评论，这是仓库名
  client_id: a6df579b14f7da8aAAAAc # 注册OAuth Application时生成
  client_secret: 1f6568974d6f3ed28055d2243d05457f7eAAAAAAAA # 注册OAuth Application时生成
  proxy_gateway: # Address of api proxy, See: https://github.com/aimingoo/intersect
  redirect_protocol: # Protocol of redirect_uri with force_redirect_protocol when mint enabled
```
github_repo # 在Github新建一个空仓库用于存放评论，这是仓库名

3. 之后生成并且部署才会生效，本地有时没有效果
``` bash
hexo g -d
```
部署之后，有可能碰到 Not Found Error，先不要着急，等一段时间再看看。
之后文章底部会出现 初始化本文的评论页，点击初始化。

# 日历插件
``` bash
npm install --save github:theme-next/theme-next-calendar
``` 
在NexT的主题配置文件中添加配置
``` text
CloudCalendar:
  enable: true
  language: zh-CN
  single: true
  root: /calendar/
  calendarCdn: //cdn.jsdelivr.net/gh/theme-next/theme-next-calendar/calendar.min.js
  langCdn: //cdn.jsdelivr.net/gh/theme-next/theme-next-calendar/languages.min.js
  #disableSidebar: false
```
即可使用。

此插件会在侧边栏的最下方添加一个日历。如侧边栏比较窄，视觉效果可能会稍差。此外，在低分辨率的屏幕上，增加日历会使侧边栏出现一个滚动条，可能会影响美观。

使用CDN的缺点是无法进行细节上的自定义配置，只能照着默认的来。如果想自己修改日历的颜色、位置等信息，请用这种方法安装，就可以自行修改文件。

# 更多服务
[third-party-services](https://theme-next.org/docs/third-party-services/)