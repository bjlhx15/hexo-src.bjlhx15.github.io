---
title: 008-模板-NexT-版权、打赏
categories:
  - hexo
date: 2020-01-30 21:03:35
tags:
permalink:
---

摘要：文章底部版权、文章底部打赏
<!--more-->

# 文章底部版权

## 概述

在 Next7.2.0官方对版权声明的设置做出了大改动，在主题配置文件_config.yml中已经找不到设置版权声明的post_copyright选项

旧版本：Next下，设置post_copyright为true，或是修改themes/next/layout/_macro/post-copyright.swig文件，自定义版权声明样式

新版本：Next下，网上一片教程，没有post_copyright选项了，各种改代码。

## 探究新版版权

发现：themes/next/layout/_partials/post/post-copyright.swig 还是有版权定义。

查看版权配置文件：themes/next/layout/_macro/post.swig

打开，直接搜搜：post-copyright.swig，
``` text
      {%- if theme.creative_commons.license and theme.creative_commons.post %}
        {{ partial('_partials/post/post-copyright.swig') }}
      {%- endif %}
```
查看样式什么时候导入：themes/next/source/css/_common/components/post/post.styl
`@import 'post-copyright' if (hexo-config('creative_commons.post'));`  
即配置文件中 有creative_commons

``` yaml
creative_commons:
  license: by-nc-sa
  sidebar: true
  post: true
  language:
```

# 文章底部打赏

## 配置使用
查看 源码 发现 reward，配置：reward_settings

1. 打赏图片增加
支付宝，微信获取付款图片  
source/images/reward/bjlhx-wx.bmp  
source/images/reward/bjlhx-wx.bmp

2. 配置
``` yaml
# Reward (Donate)
# Front-matter variable (unsupport animation).
reward_settings:
  # If true, reward will be displayed in every article by default.
  enable: true
  animation: true
  comment: 一分也是爱，两分情更浓【还没有人赞赏，支持一下呗】

reward:
  wechatpay: /images/reward/bjlhx-wx.bmp
  alipay: /images/reward/bjlhx-zfb.bmp
  #bitcoin: /images/bitcoin.png
```
