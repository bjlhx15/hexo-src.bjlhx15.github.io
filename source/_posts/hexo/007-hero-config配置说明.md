---
title: 007-hero-config配置说明
date: 2020-01-30 12:12:07
tags:
categories: 
- hexo
permalink:
copyright: true #新增,开启
---

位于站点根目录下的 _config.yml 文件，可以直接用记事本打开进行编辑，文件中的具体配置项.  
Site、URL、Directory、Writing、Home page setting、Category & Tag、Date / Time format、Pagination、Extensions、Deployment

<!--more-->

# Site
网站的个性化描述，大家需要根据自己的实际情况认真填写
``` text
Setting	描述
title	网站标题
subtitle	网站副标题
description	网站描述
keywords	网站关键字
author	网站作者
language	网站使用的语言，默认是en ，中文网站填zh-Hans
timezone	网站使用的时区，默认为 计算机的预设置，可以不填
```

# URL
关于博客文章 URL 的设置，一般不用进行更改
``` text
Setting	描述
url	网站的网址
root	网站的根目录， 也是存放文章的目录
permalink	文章的链接格式 ，默认为 :year/:month/:day/:title/
permalink_defaults	永久链接中每个段的默认值
```

# Directory
关于文件夹的设置，也是一般不用进行更改
``` text
Setting	描述
source_dir	资源文件夹 ，存放用户的资源文件，默认为 source
public_dir	公用文件夹 ，存放生成的静态文件，默认为 public
tag_dir	标签目录 ，默认为 tags
archive_dir	档案目录 ，默认为 archives
category_dir	分类目录 ，默认为 categories
code_dir	代码目录 ，默认为 downloads/code
i18n_dir	i18n目录 ，默认为 :lang
skip_render	储存站长验证文件，跳过指定文件的渲染
```

# Writing
这里是比较常用的写作设置，可以根据自己的写作习惯随时进行调整
``` text
Setting	描述
new_post_name	文章的文件名格式，默认为 :title.md
default_layout	预设的布局模板，默认为 post
titlecase	标题是否使用首字母大写 ，默认为 false
external_link	链接是否在新标签页中打开，默认为 true
filename_case	将文件名转换为 1 小写 或 2 大写，默认为 0
render_drafts	是否显示渲染草稿，默认为 false
post_asset_folder	是否启用 Asset 文件夹，默认为 false
relative_link	是否建立相对于根文件夹的链接，默认为 false
future	是否显示未来文章，默认为 true
highlight	代码块设置
```
## highlight
``` text
Setting	描述
enable	是否使用代码高亮 ，默认为 true
line_number	是否显示行号 ，默认为 true
auto_detect	是否自动检测语言 ，默认为 false
tab_replace	tab 替代设置
```

# Home page setting
首页设置，可以自己决定每页显示的文章数量和显示文章的顺序
``` text
Setting	描述
index_generator	主页设置
```

index_generator
``` text
Setting	描述
path	首页的根目录
per_page	每页显示文章的数量，默认为 10
order_by	显示文章的顺序，默认为 -date
```

# Category & Tag
这里是关于分类和标签的配置
``` text
Setting	描述
default_category	预设分类，默认为 uncategorized
category_map	分类别名
tag_map	标签别名
```

# Date / Time format
时间和日期的显示格式，一般没特殊要求的也不需要改
``` text
Setting	描述
date_format	日期格式，默认为 YYYY-MM-DD
time_format	时间格式，默认为 HH:mm:ss
```

# Pagination
这里是分页设置，可以自己决定单个页面上显示的文章数量和分页目录
``` text
Setting	描述
per_page	单个页面上显示的文章数量，默认为 10 ，用 0 表示禁用分页
pagination_dir	分页目录，默认为 page
```

# Extensions
这里可以设置主题类型和插件，之后的文章讲到更换博客主题时需要进行更改
``` text
Setting	描述
theme	博客使用的主题，默认为 landscape
```

# Deployment
这里是关于网站部署的配置，常用的有部署类型和部署地址
``` text
Setting	描述
deploy	网站部署配置
```
## deploy
``` text
Setting	描述
type	网站部署类型
repo	网站部署地址
```
[【参考资料】](https://hexo.io/docs/configuration)

