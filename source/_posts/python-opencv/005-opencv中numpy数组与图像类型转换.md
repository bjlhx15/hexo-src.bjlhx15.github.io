---
title: 005-opencv中numpy数组与图像类型转换
categories:
  - python-opencv
abbrlink: b2b98e8d
date: 2020-03-02 14:30:32
---

摘要：005-opencv中numpy数组与图像类型转换

github:https://github.com/bjlhx15/python-algorithm.git 下 src/python-opencv/005
<!-- more -->

# numpy数组与图像类型转换
Python OpenCV存储图像使用的是Numpy存储，所以可以将Numpy当做图像类型操作，操作之前还需进行类型转换，转换到int8类型

## 遍历图片三个通道像素点，并修改相应的RGB

