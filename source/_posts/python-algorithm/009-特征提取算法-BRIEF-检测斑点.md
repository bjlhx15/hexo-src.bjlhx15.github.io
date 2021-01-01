---
title: 009-特征提取算法-BRIEF-检测斑点
categories:
  - python-algorithm
abbrlink: 6944d2af
date: 2020-02-29 15:54:50
---

摘要：009-特征提取算法-BRIEF-检测斑点
<!-- more -->

# 概述

并不是特征检测算法，它是一个描述符。关键点描述符是图像的一种表示，因为可以比较两个图像的关键点描述符，并找到他们的共同之处，因此可以作为特征匹配的一种方法。

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




