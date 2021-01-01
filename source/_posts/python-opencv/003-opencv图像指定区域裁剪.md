---
title: 003-opencv图像指定区域裁剪
categories:
  - python-opencv
abbrlink: a2f0259b
date: 2020-03-01 08:39:51
---

摘要：003-opencv图像指定区域裁剪

在工作中。在做数据集时，需要对图片进行处理，照相的图片我们只需要特定的部分，所以就想到裁剪一种所需的部分。当然若是图片有规律可循则使用opencv对其进行膨胀腐蚀等操作。这样更精准一些。

github:https://github.com/bjlhx15/python-algorithm.git 下 src/python-opencv/003-cut
<!-- more -->

# 指定图像位置的裁剪处理

```python
# -*- coding: UTF-8 -*-
import os   
import cv2
import numpy as np
 
# 遍历指定目录，显示目录下的所有文件名
def CropImage4File(filepath,destpath):
    pathDir =  os.listdir(filepath)    # 列出文件路径中的所有路径或文件
    for allDir in pathDir:
        child = os.path.join(filepath, allDir)
        dest = os.path.join(destpath,allDir)
        if os.path.isfile(child):
            image = cv2.imread(child)
            # 获取图像形状 返回 行数值，列数值列表
            sp=image.shape
            #图像的高度（行 范围）
            sz1 = sp[0]       
            #图像的宽度（列 范围）          
            sz2 = sp[1]                 
            #sz3 = sp[2]                #像素值由【RGB】三原色组成
            
            #你想对文件的操作
            a=int(sz1/2-64) # x start
            b=int(sz1/2+64) # x end
            c=int(sz2/2-64) # y start
            d=int(sz2/2+64) # y end
            cropImg = image[a:b,c:d]   #裁剪图像
            cv2.imwrite(dest,cropImg)  #写入图像路径
           
if __name__ == '__main__':
    filepath ='src/python-opencv/'             #源图像
    destpath='src/python-opencv/003-cut/dist_img'        # resized images saved here
    CropImage4File(filepath,destpath)
```

# 批量处理—指定图像位置的裁剪

我这个是用来截取发票的印章区域，用于图像分割（公司的数据集保密）
各位可以用自己的增值发票裁剪。适当的更改截取区域
```python
# -*- coding: UTF-8 -*-
"""
处理数据集 和 标签数据集的代码：（主要是对原始数据集裁剪）
    处理方式：分别处理
    注意修改 输入 输出目录 和 生成的文件名
    output_dir = "./label_temp"
    input_dir = "./label"
"""
import cv2
import os
import sys
import time


def get_img(input_dir):
    img_paths = []
    for (path,dirname,filenames) in os.walk(input_dir):
        for filename in filenames:
            img_paths.append(path+'/'+filename)
    print("img_paths:",img_paths)
    return img_paths


def cut_img(img_paths,output_dir):
    scale = len(img_paths)
    for i,img_path in enumerate(img_paths):
        a = "#"* int(i/1000)
        b = "."*(int(scale/1000)-int(i/1000))
        c = (i/scale)*100
        time.sleep(0.2)
        print('正在处理图像： %s' % img_path.split('/')[-1])
        img = cv2.imread(img_path)
        weight = img.shape[1]
        if weight>1600:                         # 正常发票
            cropImg = img[50:200, 700:1500]    # 裁剪【y1,y2：x1,x2】
            #cropImg = cv2.resize(cropImg, None, fx=0.5, fy=0.5,
                                 #interpolation=cv2.INTER_CUBIC) #缩小图像
            cv2.imwrite(output_dir + '/' + img_path.split('/')[-1], cropImg)
        else:                                        # 卷帘发票
            cropImg_01 = img[30:150, 50:600]
            cv2.imwrite(output_dir + '/'+img_path.split('/')[-1], cropImg_01)
        print('{:^3.3f}%[{}>>{}]'.format(c,a,b))

if __name__ == '__main__':
    output_dir = "src/python-opencv/003-cut/dist_img"           # 保存截取的图像目录
    input_dir = "src/python-opencv"                # 读取图片目录表
    img_paths = get_img(input_dir)
    print('图片获取完成 。。。！')
    cut_img(img_paths,output_dir)
```

# 多进程（加快处理）

```python
# -*- coding: UTF-8 -*-
#coding: utf-8
"""
采用多进程加快处理。添加了在读取图片时捕获异常，OpenCV对大分辨率或者tif格式图片支持不好
处理数据集 和 标签数据集的代码：（主要是对原始数据集裁剪）
    处理方式：分别处理
    注意修改 输入 输出目录 和 生成的文件名
    output_dir = "./label_temp"
    input_dir = "./label"
"""
import multiprocessing
import cv2
import os
import time


def get_img(input_dir):
    img_paths = []
    for (path,dirname,filenames) in os.walk(input_dir):
        for filename in filenames:
            img_paths.append(path+'/'+filename)
    print("img_paths:",img_paths)
    return img_paths


def cut_img(img_paths,output_dir):
    imread_failed = []
    try:
        img = cv2.imread(img_paths)
        height, weight = img.shape[:2]
        if (1.0 * height / weight) < 1.3:       # 正常发票
            cropImg = img[30:150, 50:600]     # 裁剪【y1,y2：x1,x2】
            cv2.imwrite(output_dir + '/1' + img_paths.split('/')[-1], cropImg)
        else:                                   # 卷帘发票
            cropImg_01 = img[30:150, 30:60]
            cv2.imwrite(output_dir + '/' + img_paths.split('/')[-1], cropImg_01)
    except Exception as r:
        print("error:",r)
        imread_failed.append(img_paths)
    return imread_failed


def main(input_dir,output_dir):
    img_paths = get_img(input_dir)
    scale = len(img_paths)

    results = []
    pool = multiprocessing.Pool(processes = 4)
    for i,img_path in enumerate(img_paths):
        a = "#"* int(i/10)
        b = "."*(int(scale/10)-int(i/10))
        c = (i/scale)*100
        results.append(pool.apply_async(cut_img, (img_path,output_dir )))
        print('{:^3.3f}%[{}>>{}]'.format(c, a, b)) # 进度条（可用tqdm）
    pool.close()                        # 调用join之前，先调用close函数，否则会出错。
    pool.join()                         # join函数等待所有子进程结束
    for result in results:
        print('image read failed!:', result.get())
    print ("All done.")



if __name__ == "__main__":
    output_dir = "src/python-opencv/003-cut/dist_img"           # 保存截取的图像目录
    input_dir = "src/python-opencv/003-cut/dist_img"                # 读取图片目录表
    main(input_dir, output_dir)
    # cv2.imshow('imgflip_1',imgflip_1) 
```
