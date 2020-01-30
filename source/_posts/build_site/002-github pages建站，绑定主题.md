---
title: 002-github pages建站，绑定主题
date: 2020-01-30 17:14:39
tags:
categories: 
- build_site
---

搭建属于自己的博客网站，这里使用 github pages建站，绑定主题
<!--more-->

# 概述
基于github建站，使用仓库存放静态代码

## 基础储备
github账号注册

git使用：https://www.cnblogs.com/bjlhx/category/993475.html　　

**基础步骤**
- Github Pages
- Hexo 博客框架
- 部署
- Next 主题
  
**涉及三个仓库**
- 静态代码部署仓库：username.github.io 作用：username 需要设置成每个人仓库自己的，主要是基于Github Pages 的 部署代码。
- 静态代码开发仓库：hexo-src.bjlhx15.github.io 作用：开发代码的仓库。
- 基于github评论仓库：ment.bjlhx15.github.io 作用：使用基于github issues机制的评论仓库

# 使用

## Github Pages使用
Github Pages 其实本身就是 Github 提供的博客服务。 我们在 Github 中创建一个特定格式的 Repository，Github Pages 就会将里面的信息生成一个网页，展示出来。

创建仓库：username.github.io：

创建即可，访问：https://bjlhx15.github.io/  博客首页

## 框架使用

Hexo：是一个博客框架。它把本地文件里的信息生成一个网页。

使用 Hexo 之前，需要先安装 Node.js 和 Git。检测安装
```bash
node -v
git --version
```
hexo安装：`npm install -g hexo-cli`

查看：`hexo -v`

备注：[006-node npm 报错 rollbackFailedOptional: verb npm-session](https://www.cnblogs.com/bjlhx/p/12239748.html)

### 框架结合源码
在其他目录下使用：hexo init，下载代码，因为 init 初始化需要一个空目录，否则报错

github上新建一个仓库：hexo-src.bjlhx15.github.io.git，拉取到本地，将上述代码拷贝至在仓库中即可

npm install 安装依赖包

hexo g 生成（generate）网页。 由于我们还没创建任何博客，生成的网页会展示 Hexo 里面自带了一个 Hello World 的博客。

hexo s 将生成的网页放在了本地服务器（server）

[更多](https://blog.bjlhx.top/categories/hexo/)

### 新建博客并发布
source/_posts 放置 XX.md 文章即可，执行hexo g、hexo s查看

发布，将生成的前端代码发布到：bjlhx15.github.io.git  仓库，注意不是2.3中的源码仓库 hexo-src

在根目录：_config.yml 下修改deploy：
``` text
deploy:
  type: 'git'
  repository: https://github.com/bjlhx15/bjlhx15.github.io.git
  branch: master
```
输入 `npm install hexo-deployer-git --save` 安装 hexo-deployer-git 此步骤只需要做一次。

输入 `hexo d`，会将本地代码部署至 部署仓库地址

至此基础版本搭建完成：https://bjlhx15.github.io/

## 使用Next主题
以 Next 为例。 大概思路就是把整个主题的文件克隆到我们的主题文件夹中。在配置文件中注明使用该主题。【更多】
1. 下载主题

在hexo-src源码的根目录下：`git clone https://github.com/theme-next/hexo-theme-next themes/next`

这样，该主题的文件就全部克隆到:   themes\next 下面。

2. 切换主题
``` text 根目录下_config.yml
# theme: landscape  next
theme: next
``` 
访问[https://bjlhx15.github.io](https://bjlhx15.github.io)即可

