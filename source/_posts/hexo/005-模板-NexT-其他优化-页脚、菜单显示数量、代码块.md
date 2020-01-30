---
title: 005-模板-NexT-其他优化-页脚、菜单显示数量、代码块
date: 2020-01-29 22:30:06
tags:
categories: 
- hexo
permalink:
---

关闭页脚的powered、hexo 首页文章只显示一部分、菜单显示数量、代码块

<!--more-->

# 关闭页脚的powered

进入：themes/next/_config.yml，找到footer 下 powered、theme 关闭即可

# hexo首页文章只显示一部分

方式一、Front-matter  
在Front-matter中添加了描述并将其值设置为文章摘要，则默认情况下，NexT会将描述摘录为首页的前导文本。如果没有描述，则全部内容将为首页中的前导文字。  
您可以通过在主题配置文件中将excerpt_description的值设置为false来禁用它。

方式二、在文章中加上`<!--more-->` 标记 ，该标记以后部分就不在显示了，只有展开全部才显示，这是hexo定义的。

方式三、插件【不推荐】

# 菜单标签显示数值

在menu_settings如果设置icon: false则无图标，badges: true则标签都会显示数字
``` text
menu_settings:
  icons: true
  badges: true #默认是false
```

# 代码块设置
``` text
codeblock:
  # Code Highlight theme
  # Available values: normal | night | night eighties | night blue | night bright | solarized | solarized dark | galactic
  # See: https://github.com/chriskempson/tomorrow-theme
  highlight_theme: normal
  # Add copy button on codeblock
  copy_button:
    enable: false
    # Show text copy result.
    show_result: false
    # Available values: default | flat | mac
    style:
```







