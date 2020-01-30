---
title: 003-模板-scaffolds
date: 2020-01-29 17:50:49
categories: 
- hexo
tags:
- hexo
---

模板以scaffolds/post.md为例，对updated、permalink等参数进行说明

<!--more-->
# 概述
## scaffolds\post.md 说明
打开.\scaffolds\post.md文件，默认参数如下：
``` text
---
title: {{ title }}
date: {{ date }}
tags:
---
```
当我们在命令行中输入：
``` bash
hexo new ABC
```
则会在.\source\_post\目录下产生一个ABC.md文件，内容如下：
``` text
---
title: ABC
date: 2015-12-29 20:20:47
tags:
---
```

## permalink设置

修改.\scaffolds\post.md，增加一个permalink属性：
``` text
---
title: {{ title }}
date: {{ date }}
tags:
permalink:
---
```
让permalink为空，则Hexo会使用默认设置。默认设置是什么呢？就是你的根目录下的_config.yml中定义好的内容：
``` text
url: http://blog.bjlhx.top
root: /
permalink: :year/:month/:day/:title/
permalink_defaults:
```
:year表示年份，:month表示月份，:date表示日期。最终的展示效果为https://likianta.coding.me/2017/09/04/title/这种形式。

方式一、可以把图中的斜杠改为短横线，效果会变成https://likianta.coding.me/2017-09-04/title/。

方式二、将默认设置改成了permalink: :year/:category/:title，其最终的网址就是https://likianta.coding.me/2017/xx分组/xx标题。

分类设置:参看上文

另外需要注意的是，.\scaffolds\post.md中的permalink请一定要留空。

官网中虽然说.\scaffolds\post.md中的permalink内容可以覆盖根目录的默认设置，但实测发现会引起网址bug。

比如说你在.\scaffolds\post.md中修改为：

``` text
permalink: :year:month/:title
```

Hexo会误把它当成一个字符串进行解析，结果就会生成：https://likianta.coding.me/2017/09/04/:year:month/:title/（一个错误的URL路径）。

不过如果我们在具体的文章中手动写上的话是不会报错的：

.\source\_post\ABC.md:

``` text
---
title: 003-scaffolds.md
date: 2020-01-29 17:50:48
permalink: https://bjlhx.top/2020/01/29/ABC/
```



