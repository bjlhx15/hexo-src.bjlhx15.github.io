---
title: 001-mac安装、工具vscode配置、工程目录
categories:
  - python-syntax
abbrlink: 769cb3c4
date: 2020-02-14 14:16:27
---

摘要：
<!-- more -->

# 安装
[001-mac搭建Python开发环境、Anaconda、zsh兼容](https://www.cnblogs.com/bjlhx/p/11940108.html)

## 检测安装情况
一般默认安装了 py2
``` BASH
# py 3 版本检测
python3 -V
# py 2 版本检测
python2 -V
# Anaconda 版本检测
conda --version

```

## vscodoe安装Python插件
搜索后安装“Python”即可,成功后重启

如果你同时安装了多个版本的Python（如Python2.7，Python3.x和Anaconda），你可以通过点击左下角的语言（这里的Python x.x.x）或在命令盘中选择select interpreter来切换Python解释器。VSCode默认用PEP8标准来格式化Python代码，但你也可以选用其他标准。

# python项目工程结构
参看几个比较流行的python开源项目
[flask](https://github.com/pallets/flask)
[requests](https://github.com/psf/requests)
[thefuck](https://github.com/nvbn/thefuck)
[compose](https://github.com/docker/compose)
[tensorflow](https://github.com/tensorflow/tensorflow)
[django](https://github.com/django/django)

如下
```text
Project/
|-- bin/    存放项目的一些可执行文件，或 script/。
|   |-- foo
|
|-- project/ 存放项目的所有源代码。(1) 源代码中的所有模块、包都应该放在此目录。不要置于顶层目录。(2) 其子目录tests/存放单元测试代码； (3) 程序的入口最好命名为main.py。
|   |-- __init__.py
|   |-- main.py   
|   |-- moduleA
|   |   |-- __init__.py
|   |   |-- packageA.py
|   |-- moduleB
|   |   |-- __init__.py
|   |   |-- packageB.py
|
|-- tests/
|   |-- __init__.py
|   |-- test_main.py
|
|-- docs/ 文档
|   |-- conf.py
|   |-- abc.rst
|
|-- setup.py  安装、部署、打包的脚本
|-- requirements.txt 存放软件依赖的外部Python包列表
|-- README  说明
```

[查看](https://stackoverflow.com/questions/193161/what-is-the-best-project-structure-for-a-python-application)


