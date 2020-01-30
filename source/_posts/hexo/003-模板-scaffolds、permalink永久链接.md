---
title: 003-模板-scaffolds、permalink永久链接
categories:
  - hexo
tags:
  - hexo
abbrlink: 5c36d9d6
date: 2020-01-29 17:50:49
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

### Hexo-abbrlink生成唯一永久文章链接
hexo-next文章链接默认的生成规则是：:year/:month/:day/:title，是按照年、月、日、标题来生成的。

如果文章标题是中文的话，URL链接是也会是中文，

方案一、使用hexo-permalink-pinyin插件，将中文转英文

缺陷，比如修改了文章标题，重新hexo三连后，URL就变了，以前的文章地址变成了404。而且这样生成的URL层级也很深，不利于SEO。

方案二、Hexo-abbrlink

一、不用增加属性，也不用考虑分类中文化的问题。二、URL层级更短，更利于SEO。(一般SEO只爬三层)

在执行 `hexo g` 的时候根据 文件内的title 生成CRC，为了降低碰撞，建议文件内的title，date最好修改

并且 URL ：articles/:abbrlink.html 可设置为：`articles/:year:month:day/:abbrlink.html`
1. 插件安装
``` bash
npm install hexo-abbrlink --save
```
2. 配置
``` yaml
permalink: articles/:year:month:day/:abbrlink.html  # 此处可以自己设置，也可以直接使用 :/abbrlink
abbrlink:
    alg: crc32   #算法： crc16(default) and crc32
    rep: hex     #进制： dec(default) and hex
```
生成的链接将会是这样的(官方样例)：
``` text
crc16 & hex
https://post.zz173.com/posts/66c8.html

crc16 & dec
https://post.zz173.com/posts/65535.html

crc32 & hex
https://post.zz173.com/posts/8ddf18fb.html

crc32 & dec
https://post.zz173.com/posts/1690090958.html
```
生成完后，原md文件的Front-matter 内会增加abbrlink 字段，值为生成的ID 。这个字段确保了在我们修改了Front-matter 内的博客标题title或创建日期date字段之后而不会改变链接地址。
