---
title: 007-特征提取算法-SURF-检测斑点
categories:
  - python-algorithm
abbrlink: 4550d3a
date: 2020-02-29 15:19:07
---

摘要：005-特征提取算法-Harris-检测角点
<!-- more -->

# 概述

SURF采用快速Hessian算法检测关键点，然后SURF提取特征

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




