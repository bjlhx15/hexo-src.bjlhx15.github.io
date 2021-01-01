---
title: 003-ocr-baidu
categories:
  - python-algorithm
abbrlink: ce469e7
date: 2020-02-20 21:36:09
---

摘要：003-ocr-baidu
<!-- more -->

# 概述
从一张图片中识别出中文，通过python来实现

## 百度注册
百度云注册账号 https://cloud.baidu.com/?from=console
管理应用 https://console.bce.baidu.com/ai/#/ai/ocr/overview/index 创建一个

进入链接之后创建应用，由于是从文字识别点进去的，所以默认选中的就是ocr相关内容，填好表格确认。

使用：AppID 、API Key、Secret Key

## 调用
官方指南：https://ai.baidu.com/docs#/OCR-Python-SDK/top
安装使用Python SDK： pip install baidu-aip
cv2 需要安装：pip install opencv_python
如果只需要预测文字以及框出文字区域，执行以下代码即可。

```python
import cv2
from aip import AipOcr

""" 你的 APPID AK SK  图2的内容"""
APP_ID = '14318340'
API_KEY = 'DUvK5jEkNmCIEz4cXH8VvIVC'
SECRET_KEY = '*******'

client = AipOcr(APP_ID, API_KEY, SECRET_KEY)

fname = 'picture/test4.jpg'

""" 读取图片 """
def get_file_content(filePath):
    with open(filePath, 'rb') as fp:
        return fp.read()

image = get_file_content(fname)

""" 调用通用文字识别, 图片参数为本地图片 """
results = client.general(image)["words_result"]  # 还可以使用身份证驾驶证模板，直接得到字典对应所需字段

img = cv2.imread(fname)
for result in results:
    text = result["words"]
    location = result["location"]

    print(text)
    # 画矩形框
    cv2.rectangle(img, (location["left"],location["top"]), (location["left"]+location["width"],location["top"]+location["height"]), (0,255,0), 2)

cv2.imwrite(fname[:-4]+"_result.jpg", img)
```