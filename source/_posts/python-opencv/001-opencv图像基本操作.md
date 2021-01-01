---
title: 001-opencv图像基本操作
categories:
  - python-opencv
abbrlink: 2c2f4b26
date: 2020-02-29 22:04:17
---

摘要：001-opencv图像基本操作
github:https://github.com/bjlhx15/python-algorithm.git 下 src/python-opencv/001-base
<!-- more -->

# 图片 读、写、显示、属性查看

```python
# -*- coding: UTF-8 -*-
import numpy as np
import cv2

# -----------------------------直接读取--------------------------------
def direct_read(imgPath):
    img = cv2.imread(imgPath,0)    #读取图像（直接读取）

    # img = cv2.imread(imgPath,cv2.IMREAD_COLOR)   #读取图像（BGR，忽略alpha通道）
    # img = cv2.imread(imgPath,cv2.IMREAD_GRAYSCALE)   #读取图像（读入灰度图片）
    # img = cv2.imread(imgPath,cv2.IMREAD_UNCHANGED)   #读取图像（完整图片，包括alpha通道）
    
    # -----------------------------图片的属性 （长、宽、通道数）--------------------------------

    """
    输出：
        图片形状： (366, 445, 3)
        图片大小： 488610
        图片类型： uint8
    """
    print ("图片形状：",img.shape)
    print ("图片大小：",img.size)
    print ("图片类型：",img.dtype)

    cv2.imshow('image',img)         #显示图像
    k = cv2.waitKey(0)
    if k == 27:                     # 按 Esc 退出（关闭显示窗口）
        cv2.destroyAllWindows()
    elif k == ord('s'):             # 按 's' 保存并退出 （关闭显示窗口）
        cv2.imwrite('test.png',img)
        cv2.destroyAllWindows() 

# -----------------------------保存------------------------

"""
写入（保存）图片选取参数：
	cv2.imwrite(file，img，num)
	file：文件名
	img： 图像
	num： 可选参数（针对特定的格式）：
		  对于JPEG，表示图像的质量，用0-100的整数表示，默认95，越高画质越好，文件越大
		  对于png， 表示压缩级别。范围0到9，默认3，越高文件越小，画质越差
"""
def write(imgPath):
    grayImage = cv2.imread(imgPath, cv2.IMREAD_GRAYSCALE)
    cv2.imwrite('test.png', grayImage) 
    cv2.imwrite('test.jpeg', grayImage , (cv2.IMWRITE_JPEG_QUALITY, 80))
    cv2.imwrite('test.png', grayImage , (cv2.IMWRITE_PNG_COMPRESSION, 5))

if __name__ == "__main__":
    imgPath = 'src/python-opencv/a.jpg'
    direct_read(imgPath)
```

# 图像增强（缩放、平移、旋转、翻转）

```python
# -*- coding: UTF-8 -*-
import numpy as np
import cv2

# （缩放、平移、旋转、翻转）

# -----------------------------图像缩放--------------------------------
"""
cv2.resize(src,dsize,dst=None,fx=None,fy=None,interpolation=None)  
	scr:   原图
	dsize：输出图像尺寸
	fx:    沿水平轴的比例因子(大于1放大图像；小于1缩小图像)
	fy:    沿垂直轴的比例因子（同上）
	interpolation：插值方法（见附录）
""" 
def img_resize(imgPath):
    # 方法1：直接设定缩放因子
    img = cv2.imread(imgPath)
    res = cv2.resize(img,None,fx=2, fy=2, interpolation = cv2.INTER_CUBIC)

    print(img.shape)         #输出为（366，445，3）【宽:x轴，高:y轴，通道数】
    print(img.shape[:2])     #输出为（366，445）
    cv2.imshow('res', res)

    #方法2：获取原图像的水平方向尺寸和垂直方向尺寸。再缩放
    height, width = img.shape[:2]
    res2 = cv2.resize(img,(2*width, 3*height), interpolation = cv2.INTER_CUBIC)
    cv2.imshow('res2', res2)

    cv2.waitKey(0)
    cv2.destroyAllWindows() 

# -----------------------------图像平移--------------------------------
def img_warp(imgPath):
    img = cv2.imread(imgPath,0)
    rows,cols = img.shape
    
    M = np.float32([[1,0,100],[0,1,50]])
    dst = cv2.warpAffine(img,M,(cols,rows))
    
    cv2.imshow('img',dst)                  #展示图片并退出
    cv2.waitKey(0)
    cv2.destroyAllWindows() 

# -----------------------------图像旋转--------------------------------
"""
cv2.getRotationMatrix2D()
详细参数：
	得到变换的矩阵，通过这个矩阵再利用warpAffine来进行变换
	第一个参数就是旋转中心，元组的形式，这里设置成相片中心
	第二个参数90，是旋转的角度
	第三个参数1，表示放缩的系数，1表示保持原图大小
"""
def img_rot(imgPath):
    img = cv2.imread(imgPath)
    rows, cols,_ = img.shape

    Matrix = cv2.getRotationMatrix2D((cols / 2, rows / 2), 90, 1)
    img1 = cv2.warpAffine(img, Matrix, (cols, rows))

    cv2.imshow('img', img)
    cv2.imshow('img1', img1) 
    cv2.waitKey(0)
    cv2.destroyAllWindows()

# -----------------------------图像(X,Y轴翻转)--------------------------------
"""
 cv2.flip(img,flipcode) 翻转图像，flipcode控制翻转效果。
	flipcode = 0：沿x轴翻转
	flipcode > 0：沿y轴翻转
	flipcode < 0：x,y轴同时翻转
"""
def img_flip(imgPath):
    img = cv2.imread(imgPath,0)
    imgflip_1 = cv2.flip(img,1)
    imgflip_0 = cv2.flip(img,0)
    imgflip_2 = cv2.flip(img,-1)

    cv2.imshow('imgflip_1',imgflip_1) 
    cv2.imshow('imgflip_0',imgflip_0) 
    cv2.imshow('imgflip_2',imgflip_2) 
    cv2.waitKey(0)
    cv2.destroyAllWindows() 

if __name__ == "__main__":
    imgPath = 'src/python-opencv/a.jpg'
    # img_resize(imgPath)
    # img_warp(imgPath)
    img_rot(imgPath)
    # img_flip(imgPath)
```

# 仿射变换（图像位置校正）

```python
# -*- coding: UTF-8 -*-
import cv2
import numpy as np
 
#  仿射变换（图像位置校正）
def img_three(imgPath):
    # ---------------------------三点得到一个变换矩阵 ---------------------------
    """
        三点确定一个平面，通过确定三个点的关系来得到转换矩阵
        然后再通过warpAffine来进行变换
    """
    img = cv2.imread(imgPath)
    rows,cols,_ = img.shape
    points1 = np.float32([[50,50],[200,50],[50,200]])
    points2 = np.float32([[10,100],[200,50],[100,250]])

    matrix = cv2.getAffineTransform(points1,points2)
    output = cv2.warpAffine(img,matrix,(cols,rows))

    cv2.imshow('input1',img)
    cv2.imshow('output1',output)
    cv2.waitKey(0)
    cv2.destroyAllWindows()

def img_four(imgPath):
    # ---------------------------四点得到一个变换矩阵---------------------------
    """
        进行透视变换
        可以先用四个点来确定一个3*3的变换矩阵（cv2.getPerspectiveTransform）
        然后通过cv2.warpPerspective和上述矩阵对图像进行变换
    """
    img = cv2.imread(imgPath)
    rows,cols,_ = img.shape
    points1 = np.float32([[56,65],[368,52],[28,387],[389,390]])
    points2 = np.float32([[0,0],[300,0],[0,300],[300,300]])

    matrix = cv2.getPerspectiveTransform(points1,points2)
    # 将四个点组成的平面转换成另四个点组成的一个平面
    output = cv2.warpPerspective(img, matrix, (cols, rows))
    # 通过warpPerspective函数来进行变换

    cv2.imshow('input2',img)
    cv2.imshow('output2',output)
    cv2.waitKey(0)
    cv2.destroyAllWindows()

if __name__ == "__main__":
    imgPath = 'src/python-opencv/a.jpg'
    # img_three(imgPath)
    img_four(imgPath)
```

# 颜色变换（色调，明暗，直方图和Gamma曲线）

```python
# -*- coding: UTF-8 -*-
import cv2
import numpy as np
import matplotlib.pyplot as plt
 
#  颜色变换（色调，明暗，直方图和Gamma曲线）

def img_color(imgPath):
    original_img = cv2.imread(imgPath)
    img = cv2.resize(original_img,None,fx=0.8,fy=0.8,
                    interpolation=cv2.INTER_AREA)            # 图像缩小

    Make_border_img = cv2.copyMakeBorder(img, 30, 30, 0, 0,
                                        cv2.BORDER_CONSTANT,
                                        value=(0, 0, 0))    # 绘制边框(原图上下贴30像素黑边)

    img_hsv = cv2.cvtColor(img,cv2.COLOR_BGR2HSV)            # 转成 HSV 格式
    colorless_hsv = img_hsv.copy()
    colorless_hsv[:,:,1] = 0.5 * colorless_hsv[:,:,1]        # 减小饱和度会让图像损失鲜艳，变得更灰
    colorless_img = cv2.cvtColor(colorless_hsv,cv2.COLOR_HSV2BGR)

    darker_hsv = img_hsv.copy()
    darker_hsv[:,:,2] = 0.5 * darker_hsv[:,:,2]             # 减小明度为原来一半
    darker_img = cv2.cvtColor(darker_hsv,cv2.COLOR_HSV2BGR)

    img_corrected = gamma_trans(img,0.5)                   # 执行Gamma矫正

    cv2.imshow("original_img",img)                         # 原图
    cv2.imshow("Make_border_img",Make_border_img)          # 添加黑边
    cv2.imshow('colorless_jpg',colorless_img)              # 图像变灰
    cv2.imshow('darker_jpg',darker_img)                    # 图像变暗
    cv2.imshow('gamma_corrected_jpg',img_corrected)        # gamma校正

    # cv2.waitKey(0)
    # cv2.destroyAllWindows()

    hist_b = cv2.calcHist([img],[0],None,[256],[0,256])    # 分通道计算每个通道的直方图
    hist_g = cv2.calcHist([img],[1],None,[256],[0,256])
    hist_r = cv2.calcHist([img],[2],None,[256],[0,256])

    plt.plot(hist_b, label='B', color='blue')              # 显示3个通道的颜色直方图
    plt.plot(hist_g, label='G', color='green')
    plt.plot(hist_r, label='R', color='red')
    plt.legend(loc='best')
    plt.xlim([0, 256])
    plt.show()

def gamma_trans(img,gamma):
    gamma_table = [np.power(x/255.0,gamma)*255.0 for x in range(256)]
    gamma_table = np.round(np.array(gamma_table)).astype(np.uint8)
    return cv2.LUT(img,gamma_table)                    # 实现这个映射用的是OpenCV的查表函数


if __name__ == "__main__":
    imgPath = 'src/python-opencv/a.jpg'
    # img_three(imgPath)
    img_color(imgPath)
```

## 本地摄像头，视频处理

```python
# -*- coding: UTF-8 -*-
import cv2
import numpy as np
import matplotlib.pyplot as plt
 
"""
cv2.cvtColor(input_image, flag)函数实现图片颜色空间的转换：
	flag 参数决定变换类型。如 BGR->Gray           
	flag 就可以设置为 cv2.COLOR_BGR2GRAY
		         cv2.COLOR_BGR2HSV 。
""" 

if __name__ == "__main__":
    cap = cv2.VideoCapture(0)
 
    while(1):
        _, frame = cap.read()                           # 读取视频的每一帧
        hsv = cv2.cvtColor(frame, cv2.COLOR_BGR2HSV)    # 将图片颜色空间从 BGR -> HSV 空间

        lower_blue = np.array([110,50,50])              # 定义在HSV空间中蓝色的范围
        upper_blue = np.array([130,255,255])
    
        mask = cv2.inRange(hsv, lower_blue, upper_blue) # 根据以上定义的蓝色的阈值得到蓝色的部分
        res = cv2.bitwise_and(frame,frame, mask= mask)
    
        cv2.imshow('frame',frame)
        cv2.imshow('mask',mask)
        cv2.imshow('res',res)
        k = cv2.waitKey(5) & 0xFF
        if k == 27:
            break
    cv2.destroyAllWindows()
```

# 通道的拆分/合并处理/添加边距

```python
# -*- coding: UTF-8 -*-
import cv2
import numpy as np
import matplotlib.pyplot as plt

# 通道的拆分/合并处理/添加边距

def img_channel(imgPath):
    # 拆分/合并处理
    img = cv2.imread(imgPath)
    b,g,r = cv2.split(img)
    img = cv2.merge((b,g,r)) 

    # -----------------------------添加边距--------------------------------
    """cv2.copyMakeBorder函数""" 
    BLUE = [255,0,0]
    img1 = cv2.imread(imgPath)
    
    replicate = cv2.copyMakeBorder(img1,10,10,10,10,cv2.BORDER_REPLICATE)
    reflect = cv2.copyMakeBorder(img1,10,10,10,10,cv2.BORDER_REFLECT)
    reflect101 = cv2.copyMakeBorder(img1,10,10,10,10,cv2.BORDER_REFLECT_101)
    wrap = cv2.copyMakeBorder(img1,10,10,10,10,cv2.BORDER_WRAP)
    constant= cv2.copyMakeBorder(img1,10,10,10,10,cv2.BORDER_CONSTANT,value=BLUE)

    cv2.imshow("img",img)
    cv2.imshow("replicate",replicate)
    cv2.imshow("reflect",reflect)
    cv2.imshow("reflect101",reflect101)
    cv2.imshow("wrap",wrap)
    cv2.imshow("constant",constant)


    cv2.waitKey(0)
    cv2.destroyAllWindows()

if __name__ == "__main__":
    imgPath = 'src/python-opencv/a.jpg'
    # img_three(imgPath)
    img_channel(imgPath)
```

# 读取图像添加文字（可用于图像添加水印）

```python
# -*- coding: UTF-8 -*-
import cv2
import numpy as np
import matplotlib.pyplot as plt

# 通道的拆分/合并处理/添加边距

def img_channel(imgPath):
    
    img=cv2.imread(imgPath,cv2.IMREAD_COLOR)    # 打开文件
    cv2.imshow('Original_img',img )
    font = cv2.FONT_HERSHEY_DUPLEX              # 设置字体
    img_word = cv2.putText(img, "liuyan", (5, 150), font, 2.5, (0, 0, 0), 2,)
    """
    cv2.putText()
    传入参数：图片对象、文本、像素、字体、字体大小、颜色、字体粗细
    其中：像素为文字位置
    """

    # cv2.imshow('Original_img',img )
    cv2.imshow('Add_text_img',img_word )

    # cv2.imwrite('5.png',img_word )      # 保存

    cv2.waitKey(0)
    cv2.destroyAllWindows()             # 关闭所有窗口
    # cv2.destroyWindow(wname)            # 关闭指定窗口

if __name__ == "__main__":
    imgPath = 'src/python-opencv/a.jpg'
    img_channel(imgPath)
```

```
interpolation   所用的插值方法

	INTER_NEAREST 	最近邻插值
	INTER_LINEAR 	双线性插值（默认设置）
	INTER_AREA 	使用像素区域关系进行重采样。  
	                它可能是图像抽取的首选方法，因为它会产生无云纹理的结果。 
	                但是当图像缩放时，它类似于INTER_NEAREST方法。
	                
	INTER_CUBIC 	4x4像素邻域的双三次插值
	INTER_LANCZOS4 	8x8像素邻域的Lanczos插值 
```

# 二值化
将一个灰色的图片，变成要么是白色要么就是黑色。（大于规定thresh值就是设置的最大值（常为255，也就是白色））

Python: cv2.threshold(src, thresh, maxval, type[, dst]) → retval, dst
- src：表示的是图片源
- thresh：表示的是阈值（起始值）
- maxval：表示的是最大值
- type：表示的是这里划分的时候使用的是什么类型的算法，常用值为0（cv2.THRESH_BINARY）
```python
# -*- coding: UTF-8 -*-
import cv2
import numpy as np
# 通道的拆分/合并处理/添加边距

def img_channel(imgPath):
    
    img=cv2.imread(imgPath,cv2.IMREAD_COLOR)    # 打开文件
    # img = np.zeros((200, 200), dtype=np.uint8)
    img[50:150, 50:150] = 255  # create a image with black color in the middle and its background is white.
    cv2.imshow("Image-Old", img)
    ret, thresh = cv2.threshold(img, 127, 255, cv2.THRESH_BINARY)
    cv2.imshow("Image-New", thresh)
    cv2.waitKey()
    cv2.destroyAllWindows()

if __name__ == "__main__":
    imgPath = 'src/python-opencv/a.jpg'
    img_channel(imgPath)
```
# 反相
## 黑白反相（二值化图）
```python
# -*- coding: UTF-8 -*-
import cv2
import numpy as np
# 黑白反相（二值化图）
def inverse_color(image):
    height,width = image.shape
    img2 = image.copy()
    for i in range(height):
        for j in range(width):
            img2[i,j] = (255-image[i,j])
    return img2 

def img_channel(imgPath):
    
    img=cv2.imread(imgPath,0)    # 打开文件
    cv2.imshow("Image-Old", img)

    img_new = inverse_color(img)
    cv2.imshow("img_new", img_new)
    cv2.waitKey()
    cv2.destroyAllWindows()

if __name__ == "__main__":
    imgPath = 'src/python-opencv/a.jpg'
    img_channel(imgPath)
```
## 彩色反相（原始图）
```python
# -*- coding: UTF-8 -*-
import cv2
import numpy as np
# 彩色反相（二值化图）
def inverse_color(image): 
    height,width,_ = image.shape 
    img2 = image.copy() 
    for i in range(height): 
        for j in range(width): 
            img2[i,j] = (255-image[i,j][0],255-image[i,j][1],255-image[i,j][2]) 
    return img2

def img_channel(imgPath):
    
    img=cv2.imread(imgPath,cv2.IMREAD_COLOR)    # 打开文件
    cv2.imshow("Image-Old", img)

    img_new = inverse_color(img)
    cv2.imshow("img_new", img_new)
    cv2.waitKey()
    cv2.destroyAllWindows()

if __name__ == "__main__":
    imgPath = 'src/python-opencv/a.jpg'
    img_channel(imgPath)
```



