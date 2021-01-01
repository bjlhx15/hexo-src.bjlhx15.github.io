---
title: 002-ocr-tesseract-ocr
categories:
  - python-algorithm
abbrlink: ff5ff1f0
date: 2020-02-20 21:10:01
---

摘要：002-ocr-tesseract-ocr
<!-- more -->

# 概述
从一张图片中识别出中文，通过python来实现

Tesseract的OCR引擎目前已作为开源项目发布在Google Project，其项目主页在这里查看https://github.com/tesseract-ocr，

它支持中文OCR，并提供了一个命令行工具。python中对应的包是pytesseract. 通过这个工具我们可以识别图片上的文字。

## 准备环境

### 开发环境如下：

```text
macosx
python 3.7
brew
```

### 安装tesseract 和 pytesseract
``` BASH
# 安装tesseract
brew install tesseract

# 安装python对应的包：pytesseract
pip install pytesseract
```

### 中文字体下载安装
## 代码使用

要识别中文需要下载对应的训练集：https://github.com/tesseract-ocr/tessdata

下载”chi_sim.traineddata”，然后copy到训练数据集的存放路径，如：

/usr/local/Cellar/tesseract/4.1.1/share/tessdata

```python
# -*- coding: UTF-8 -*-

import pytesseract
from PIL import Image

image = Image.open('/Users/lihongxu6/IdeaProjectsGit/python-algorithm/tests/ocr/timg.jpeg')
code = pytesseract.image_to_string(image, lang='chi_sim')
print(code)
# 你 一 定 要 过 的 好
# 不 然 对 不 起 我 的 不 打 扰
```

