---
title: 005-特征提取算法-Harris-检测角点
categories:
  - python-algorithm
abbrlink: 49e94f08
date: 2020-02-29 10:55:37
---

摘要：005-特征提取算法-Harris-检测角点
<!-- more -->

# 概述

角点：三维图像亮度变化剧烈的点或者图像边缘曲线上曲率极大值的点

使用cornerHarris来识别角点

```python
# -*- coding: UTF-8 -*-
import cv2
import numpy as np

def a(imgPath):
    img = cv2.imread(imgPath)
    gray = cv2.cvtColor(img,cv2.COLOR_BGR2GRAY)
    gray = np.float32(gray)
    dst = cv2.cornerHarris(gray,2,3,0.04)
    dst = cv2.dilate(dst,None)
    img[dst>0.01 * dst.max()] = [0,0,255]
    cv2.imshow('corners',img)
    cv2.imwrite('corners.png',img)
    if cv2.waitKey(0) & 0xff == 27:
        cv2.destroyALLWindows()

if __name__ == "__main__":
    print ('---------检测角点------------')
    a("/Users/lihongxu6/IdeaProjectsGit/python-algorithm/tests/feature/a.jpg")
```




