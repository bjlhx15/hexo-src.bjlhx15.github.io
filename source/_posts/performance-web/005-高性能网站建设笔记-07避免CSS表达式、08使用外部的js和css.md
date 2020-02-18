---
title: 005-高性能网站建设笔记-07避免CSS表达式、08使用外部的js和css
categories:
  - performance-web
abbrlink: ac31f31
date: 2020-02-13 16:50:05
---

摘要：

<!-- more -->

# 07避免CSS表达式

浏览器兼容，大部分不支持css表达式

# 08使用外部的js和css

## 内联和外置
- 内联：默认的script 引入
- 外置：动态加载 脚本
切割公共部分，适当选择使用外部js和css
``` JS
<script>
function doOnload(){
  setTimeout("downloadComponents()",10);
}
window.onload=doOnload;

function doenloadComponents(){
  downloadJS("ss");
  downloadCSS("aaa");
}

function downloadJS(url){
  var elem = document.createElement("script");
  elem.src = url;
  document.body.appendChild(elem);
}

function downloadCSS(url){
  var elem = document.createElement("link");
  elem.rel = "stylesheet";
  elem.type = "text/css";
  elem.href = url;
  document.body.appendChild(elem);
}
</script>
```




