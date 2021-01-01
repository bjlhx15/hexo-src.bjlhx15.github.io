---
title: 006-特征提取算法-SIFT-检测斑点
categories:
  - python-algorithm
abbrlink: 811a1f63
date: 2020-02-29 11:05:42
---

摘要：006-特征提取算法-SIFT-检测斑点
<!-- more -->

# 概述

Harris的cornerHarris函数可以很好的检测角点，并且在图像旋转的情况下也能检测到，然而减少或者增加图像的大小，可能会丢失图像的某些部分，甚至会增加角点的质量。所以采用一种与图像比例变化无关的角点检测方法来解决特特征损失现象！也就是尺度不变特征变换（SIFT）。


尺度不变特征变换匹配算法
Scale Invariant Feature Transform(SIFT)

## 

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




