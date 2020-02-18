---
title: 001-html，css，js加载顺序
categories:
  - html_js_css
abbrlink: 6bc16603
date: 2020-02-12 17:00:38
---

摘要：


<!-- more -->
示例：
```html
<head lang="en">
  <meta charset="utf-8">
  <title></title>
  <link rel="stylesheet" href="css/*.css" rel="external nofollow" >
  <script src="js/*.js></script>
</head>
```

# DOM加载

DOM文档的加载顺序是由上而下的顺序加载；
```
1. 浏览器一边下载HTML网页，一边开始解析
2. 解析过程中，发现<script>标签
3. 暂停解析，网页渲染的控制权转交给JavaScript引擎
4. 如果<script>标签引用了外部脚本，就下载该脚本，否则就直接执行
5. 执行完毕，控制权交还渲染引擎，恢复往下解析HTML网页
```

## DOM加载到link标签

css文件的加载是与DOM的加载并行的，也就是说，css在加载时Dom还在继续加载构建，而过程中遇到的css样式或者img，则会向服务器发送一个请求，待资源返回后，将其添加到dom中的相对应位置中；

## DOM加载到script标签

由于js文件不会与DOM并行加载，因此需要等待js整个文件加载完之后才能继续DOM的加载，倘若js脚本文件过大，则可能导致浏览器页面显示滞后，出现“假死”状态，这种效应称之为“阻塞效应”；会导致出现非常不好的用户体验；

而这个特性也是为什么在js文件中开头需要$(document).ready(function(){})或者（function(){}）或者window.onload,即是让DOM文档加载完成之后才执行js文件，这样才不会出现查找不到DOM节点等问题；

js阻塞其他资源的加载的原因是：浏览器为了防止js修改DOM树，需要重新构建DOM树的情况出现；

## 解决方法

html需要等head中所有的js和css加载完成后才会开始绘制，但是html不需要等待放在body最后的js下载执行就会开始绘制,因此将js放在body的最后面，可以避免资源阻塞，同时使静态的html页面迅速显示。将脚本文件都放在网页尾部加载，还有一个好处。在DOM结构生成之前就调用DOM，JavaScript会报错，如果脚本都在网页尾部加载，就不存在这个问题，因为这时DOM肯定已经生成了。

前提，js是外部脚本；

### defer

外链的js如果含有defer="true"属性，将会并行加载js，到页面全部加载完成后才会执行，会按顺序执行。

defer属性的作用是，告诉浏览器，等到DOM加载完成后，再执行指定脚本。

1. 浏览器开始解析HTML网页
2. 解析过程中，发现带有defer属性的script标签
3. 浏览器继续往下解析HTML网页，同时并行下载script标签中的外部脚本
4. 浏览器完成解析HTML网页，此时再执行下载的脚本
对于内置而不是连接外部脚本的script标签，以及动态生成的script标签，defer属性不起作用。
在script标签中添加 defer=“ture”，则会让js与DOM并行加载，待页面加载完成后再执行js文件，这样则不存在阻塞；

### async

在scirpt标签中添加 async=“ture”，这个属性告诉浏览器该js文件是异步加载执行的，也就是不依赖于其他js和css，也就是说无法保证js文件的加载顺序，但是同样有与DOM并行加载的效果；

async属性的作用是，使用另一个进程下载脚本，下载时不会阻塞渲染。

1. 浏览器开始解析HTML网页
2. 解析过程中，发现带有async属性的script标签
3. 浏览器继续往下解析HTML网页，同时并行下载script标签中的外部脚本
4. 脚本下载完成，浏览器暂停解析HTML网页，开始执行下载的脚本
5. 脚本执行完毕，浏览器恢复解析HTML网页

同时使用defer和async属性时，defer属性会失效；

可以将scirpt标签放在body标签之后，这样就不会出现加载的冲突了。

一般来说，如果脚本之间没有依赖关系，就使用async属性，如果脚本之间有依赖关系，就使用defer属性。如果同时使用async和defer属性，后者不起作用，浏览器行为由async属性决定。







但是