---
title: 001-tools-mac-sublime text3
categories:
  - tools
date: 2020-02-08 09:09:21
tags:
---

概述:mac下基础操作

<!--more-->
# 软件设置

## 停止更新提示：
1. preferences→setting→增加如下
``` 
{
    "font_size": 17,
    "update_check": false,
}
```
注意：一定要在每一行结束加逗号

## 插件安装包

### 安装 Package Control
1. 安装
- 方式一、在线安装
Mac: cmd+shift+p
输入;Install Package Control, 按 enter
- 方式二、离线安装
下载：https://packagecontrol.io/installation 下载 ： Package Control.sublime-package 即可

Mac位置:/Users/用户名/Library/Application Support/Sublime Text 3/Installed Packages

2. 使用
重启Sublime3,如果菜单->Preferences有Package Setting和Package Control就说明安装成功。

Ctrl+Shift+p输入install选中Install Package回车就可以安装插件。

一般有些慢，可以在 菜单->Preferences有Package Setting 的setting user增加
```
	"debug": true,
	"downloader_precedence":
	{
		"linux":["curl","urllib","wget"],
		"osx":["curl","urllib"],
		"windows":["wininet"]
	},
```

### 安装markdown 相关

1. 安装预览：cmd+ shift + p,输入 install package，注意看左下角在加载，完毕后 出现输出框，输入markdown preview 查找合适即可

```
ddsddfsfdd  sfs  sdf
```










