---
title: 009-permalink永久链与image不显示问题
categories:
  - hexo
abbrlink: 4eaf081e
date: 2021-01-09 10:28:20
tags:
permalink:
---

摘要：文章底部版权、文章底部打赏
<!--more-->

# 配置

## 永久链

[配置]({% post_link '003-模板-scaffolds、permalink永久链接' %}#)

## asset-image配置

### 配置
配置_config.yml里面的post_asset_folder:false这个选项设置为true。Hexo 提供了一种更方便管理 Asset 的设定：post_asset_folder

```
post_asset_folder: true
```

主要是为了和文章创建同名的文件夹放图片

### 图片插件安装
```shell
npm install hexo-asset-image --save
```

# 问题与方案

## 问题

如果永久链使用标题，此问题不存在

永久链配置使用了:abbrlink 以及使用
``` yaml
permalink: articles/:year:month:day/:abbrlink.html  # 此处可以自己设置，也可以直接使用 :/abbrlink
abbrlink:
    alg: crc32   #算法： crc16(default) and crc32
    rep: hex     #进制： dec(default) and hex
```

这时候原标题为 "009-permalink永久链与image不显示问题" 会变成 "4eaf081e"

但是文章内图片链接没有变。

## 原因
由于hexo3版本后对很多插件支持有问题，hexo-asset-image插件在处理data.permalink链接时出现路径错误，把年月去掉了，导致最后生成的路径为%d/xxx/xxx需要对其做兼容处理。通过判断当前版本是否等于3的版本做不同的路径分割。

## 方案

### JS自己修改

找到：node_modules/hexo-asset-image/index.js

在代码的函数前加入：
```
var version = String(hexo.version).split('.');
```

如下
```
var version = String(hexo.version).split('.');

hexo.extend.filter.register('after_post_render', function(data){
  var config = hexo.config;
  if(config.post_asset_folder){
    var link = data.permalink;
    var beginPos = getPosition(link, '/', 3) + 1;
```


