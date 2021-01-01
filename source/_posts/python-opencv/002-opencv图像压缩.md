---
title: 002-opencv图像压缩
categories:
  - python-opencv
abbrlink: 576a7ff9
date: 2020-02-29 23:11:48
---

摘要：002-opencv图像压缩
github:https://github.com/bjlhx15/python-algorithm.git 下 src/python-opencv/002-enhance
<!-- more -->

# 简单的图像压缩
cv2的ImageEnhance模块
- ImageEnhance.Color(image) 颜色增强类：用于调整图像的颜色均衡
- ImageEnhance.Brightness(image) 亮度增强类：用于调整图像的亮度。
- ImageEnhance.Contrast(image) 对比度增强类：用于调整图像的对比度。

```python
# -*- coding: UTF-8 -*-
import numpy as np
import cv2
from PIL import Image
from PIL import ImageEnhance
from PIL import ImageFilter

# -----------------------------直接读取--------------------------------
def func(imgPath,enhance):
    # img = cv2.imread(imgPath,0)    #读取图像（直接读取）

    image=cv2.imread(imgPath)
    res = cv2.resize(image, (image.shape[1],image.shape[0]), interpolation=cv2.INTER_AREA)
    imgE = Image.fromarray(cv2.cvtColor(res,cv2.COLOR_BGR2RGB))
    imgEH = ImageEnhance.Contrast(imgE)
    # 当参数为1.2 灰度图243KB，当参数为2.8 灰度图124KB.（亮度提升后转灰度图，图片会黑白分化）
    gray=imgEH.enhance(enhance).convert("L")
    gray.save(r"src/python-opencv/002-enhance/dist_img/test_001_simple.gray."+str(enhance)+".png")

    #图像增强
    # 创建滤波器，使用不同的卷积核
    gray2=gray.filter(ImageFilter.DETAIL)
    gray2.save(r"src/python-opencv/002-enhance/dist_img/test_001_simple.gray2."+str(enhance)+".png")

    # #图像点运算
    gray3=gray2.point(lambda i:i*0.9)
    gray3.save(r"src/python-opencv/002-enhance/dist_img/test_001_simple.gray3."+str(enhance)+".png")

    # k = cv2.waitKey(0)
    # if k == 27:                     # 按 Esc 退出（关闭显示窗口）
    #     cv2.destroyAllWindows()

if __name__ == "__main__":
    imgPath = 'src/python-opencv/a.jpg'
    enhance=1.2
    func(imgPath,enhance)
    enhance=2.8
    func(imgPath,enhance)
```

# 基于机器学习的图像压缩方法

关于PCA降维，SVD理论详情：https://blog.csdn.net/wsp_1138886114/article/details/80967843

## SVD图像压缩
```python
# -*- coding: UTF-8 -*-
import numpy as np
import cv2
from PIL import Image
from PIL import ImageEnhance
from PIL import ImageFilter

def rebuild_img(u, sigma, v, p):  # p表示奇异值的百分比
    print('sigma.shape', sigma.shape)
    print('sum(sigma)', sum(sigma))
    m , n= len(u),len(v)
    a = np.zeros((m, n))         # 创建空图片

    count = (int)(sum(sigma))
    curSum = 0
    k = 0
    while curSum <= count * p:
        uk = u[:, k].reshape(m, 1)
        vk = v[k].reshape(1, n)
        a += sigma[k] * np.dot(uk, vk)
        curSum += sigma[k]
        k += 1
    print('==k===:', k)

    a[a < 0] = 0
    a[a > 255] = 255
    return np.rint(a).astype("uint8")


if __name__ == '__main__':
    img = cv2.imread(r'src/python-opencv/a.jpg')

    for p in np.arange(0.1, 1, 0.2):
        u, sigma, v = np.linalg.svd(img[:, :, 0])
        R = rebuild_img(u, sigma, v, p)

        u, sigma, v = np.linalg.svd(img[:, :, 1])
        G = rebuild_img(u, sigma, v, p)

        u, sigma, v = np.linalg.svd(img[:, :, 2])
        B = rebuild_img(u, sigma, v, p)

        I = np.stack((R, G, B), 2)
        cv2.imshow("svd_" + str(p * 100),I)
        # cv2.imwrite("src/python-opencv/002-enhance/dist_img_" + str(p * 100) + ".jpg", I)
    cv2.imshow("img" , img)
    cv2.waitKey(0)
    cv2.destroyAllWindows()
```

奇异值分解能够有效的降低数据的维数，以本文的图片为例，从450维降到149维后，还保留了90%的信息
虽然奇异值分解很有效，但是不能滥用，一般情况下要求降维后信息的损失度不能超过5%，甚至是1%
Ng的视频中提到常见的错误使用降维的情况，在这里也贴出来：
使用降维解决过拟合问题
不论什么情况，先用降维处理一下数据，即把降维当做模型训练的必须步骤

## PCA图像压缩

pca函数实现图像的降维
关于PCA理论详情请点击：https://blog.csdn.net/wsp_1138886114/article/details/80967843

```python
# -*- coding: UTF-8 -*-
import numpy as np
import cv2
from PIL import Image
from PIL import ImageEnhance
from PIL import ImageFilter

def comp_2d(image_2d,rate):
    height,width = image_2d.shape[:2]

    # 执行下面这一行报错显示无法广播。我修改了dtype还是报错，不知道为何。
    # cov_mat = image_2d - np.mean(image_2d, axis=1) 。
    # print("data.type:", image_2d.astype(np.float64).dtype)
    # print("mean.type:", np.mean(image_2d, axis=1).dtype)
    #
    # print("data.shape:", image_2d.astype(np.float64).shape)
    # print("mean.shape:", np.mean(image_2d, axis=1).shape)

    # 我自己广播代码为如下三行代码
    mean_array = np.mean(image_2d, axis=1)
    mean_array = mean_array[:, np.newaxis]
    mean_array = np.tile(mean_array, width)

    cov_mat = image_2d.astype(np.float64) - mean_array
    eig_val, eig_vec = np.linalg.eigh(np.cov(cov_mat))  # 求特征值 特征向量
    p = np.size(eig_vec, axis=1)
    idx = np.argsort(eig_val)
    idx = idx[::-1]
    eig_vec = eig_vec[:, idx]
    numpc = rate
    if numpc < p or numpc > 0:
        eig_vec = eig_vec[:, range(numpc)]
    score = np.dot(eig_vec.T, cov_mat)
    recon = np.dot(eig_vec, score) + mean_array
    recon_img_mat = np.uint8(np.absolute(recon))
    return recon_img_mat

if __name__ == '__main__':
    data = cv2.imread(r'src/python-opencv/a.jpg')
    height, width = data.shape[:2]
    a_g = data[:, :, 0]
    a_b = data[:, :, 1]
    a_r = data[:, :, 2]
    rates = [30,60,90]  #主成分前30，60，90个k值
    for rate in rates:
        g_recon, b_recon, r_recon = comp_2d(a_g,rate), comp_2d(a_b,rate), comp_2d(a_r,rate)
        result = cv2.merge([g_recon, b_recon, r_recon])
        cv2.imshow('result_'+str(rate),result)
    cv2.waitKey(0)
    cv2.destroyAllWindows()
```
显然，k=30时就包含了矩阵的至少70%的信息含量，当k=90时就包含了矩阵的至少90%信息含量。

# 基于K-means图像压缩

## 肘部法则

```python
# -*- coding: UTF-8 -*-
import numpy as np
import cv2
from PIL import Image
from PIL import ImageEnhance
from PIL import ImageFilter

from sklearn.cluster import KMeans
from scipy.spatial.distance import cdist
import matplotlib.pyplot as plt

plt.rcParams['font.sans-serif']= ['SimHei']    #中文注释
plt.rcParams['axes.unicode_minus'] = False     #显示正负号

def func():
    cluster1 = np.random.uniform(0.5,1.5,(2,5))    #生成（0.5,1.5）之间的随机数（2行5列）
    cluster2 = np.random.uniform(3.5,4.5,(2,5))
    X = np.hstack((cluster1,cluster2)).T           #列拼接 并转置（10行2列）

    K = range(1, 6)
    meandistortions = []    #存放聚类中心列表
    for k in K:
        kmeans = KMeans(n_clusters=k)
        kmeans.fit(X)       #拟合训练
        
        #任一点到 簇中心点（1,2,3,4,5）的最小距离（计算过程：求和再求平均值）
        meandistortions.append(sum(np.min(cdist(X,kmeans.cluster_centers_,'euclidean'), axis=1)) / X.shape[0])
        print("第 {} 次-聚类中心".format(k))
        print(cdist(X,kmeans.cluster_centers_,'euclidean'))
        
        print("第 {} 次聚类时----任一点到这{}个聚类中心其中一个的最小值".format(k,k))
        print(np.min(cdist(X,kmeans.cluster_centers_,'euclidean'), axis=1))
    print(meandistortions)
    plt.plot(K, meandistortions,'bx-') # 颜色blue，线条为-
    plt.xlabel('k')

    plt.ylabel('Ave Distor')           # plt.ylabel('平均畸变程度',fontproperties=font)
    plt.title('Elbow method value K')  # plt.title('用肘部法则来确定最佳的K值',fontproperties=font);
    plt.scatter(K,meandistortions)

if __name__ == '__main__':
    data = cv2.imread(r'src/python-opencv/a.jpg')
    func()
    # cv2.waitKey(0)
    # cv2.destroyAllWindows()
```

## 轮廓系数验证K值

```python
# -*- coding: UTF-8 -*-
import numpy as np
import cv2
from PIL import Image
from PIL import ImageEnhance
from PIL import ImageFilter

from sklearn.cluster import KMeans
from scipy.spatial.distance import cdist
import matplotlib.pyplot as plt
from sklearn import metrics

plt.rcParams['font.sans-serif']= ['SimHei']    #中文注释
plt.rcParams['axes.unicode_minus'] = False     #显示正负号

def func():
    plt.figure(figsize=(8, 10)) 
    plt.subplot(3, 2, 1)
    x1 = np.array([1, 2, 3, 1, 5, 6, 5, 5, 6, 7, 8, 9, 7, 9])
    x2 = np.array([1, 3, 2, 2, 8, 6, 7, 6, 7, 1, 2, 1, 1, 3])
    X = np.array(list(zip(x1, x2))).reshape(len(x1), 2)
    plt.xlim([0, 10])                                   # x轴的刻度
    plt.ylim([0, 10])                                   # y轴的刻度
    plt.title('Sample')
    plt.scatter(x1, x2)
    colors = ['b', 'g', 'r', 'c', 'm', 'y', 'k', 'b']  #样本点颜色
    markers = ['o', 's', 'D', 'v', '^', 'p', '*', '+'] #样本点形状
    tests = [2, 3, 4, 5, 8]                            #簇的个数
    subplot_counter = 1                                #训练模型
    for t in tests:
        subplot_counter += 1
        plt.subplot(3, 2, subplot_counter)
        kmeans_model = KMeans(n_clusters=t).fit(X)
        for i, l in enumerate(kmeans_model.labels_):
            plt.plot(x1[i], x2[i], color=colors[l], marker=markers[l],ls='None')
            plt.xlim([0, 10])
            plt.ylim([0, 10])                       #SCoefficient:轮廓系数[-1,1]
            plt.title('K = %s, SCoefficient = %.03f' % (t, metrics.silhouette_score
                                                        (X, kmeans_model.labels_,metric='euclidean')))
    plt.show()


if __name__ == '__main__':
    data = cv2.imread(r'src/python-opencv/a.jpg')
    func()
    # cv2.waitKey(0)
    # cv2.destroyAllWindows()
```

## Mini Batch K-Means（适合大数据的聚类算法）

```python
# -*- coding: UTF-8 -*-
import numpy as np
import cv2
import matplotlib.pyplot as plt
from sklearn.cluster import MiniBatchKMeans, KMeans
from sklearn import metrics
from sklearn.datasets.samples_generator import make_blobs

def func():
    # make_blobs 自定义数据集

    # X为样本特征，Y为样本簇类别， 共1000个样本，
    # 每个样本4个特征，共4个簇，
    # 簇中心在[-1,-1], [0,0],[1,1], [2,2]， 
    # 簇方差分别为[0.4, 0.2, 0.2]

    X, y = make_blobs(n_samples=1000, n_features=2, 
                    centers=[[-1,-1], [0,0], [1,1], [2,2]], 
                    cluster_std=[0.4, 0.2, 0.2, 0.2], 
                    random_state =9)
    plt.scatter(X[:, 0], X[:, 1], marker='o')
    plt.show()

    for index, k in enumerate((2,3,4,5)):
        plt.subplot(2,2,index+1)
        y_pred = MiniBatchKMeans(n_clusters=k, batch_size = 200, random_state=9).fit_predict(X)
        
        #用Calinski-Harabasz Index评估二分类的聚类分数 其方法是metrics.calinski_harabaz_score
        score= metrics.calinski_harabaz_score(X, y_pred)  
        plt.scatter(X[:, 0], X[:, 1], c=y_pred)
        plt.text(.99, .01, ('k=%d, score: %.2f' % (k,score)),
                    transform=plt.gca().transAxes, size=10,
                    horizontalalignment='right')
    plt.show()


if __name__ == '__main__':
    data = cv2.imread(r'src/python-opencv/a.jpg')
    func()
    # cv2.waitKey(0)
    # cv2.destroyAllWindows()
```

## 使用K-means压缩图片

```python
# -*- coding: UTF-8 -*-
import numpy as np
import cv2
import matplotlib.pyplot as plt
from sklearn.cluster import KMeans
from sklearn.metrics import pairwise_distances_argmin
from sklearn.datasets import load_sample_image
from sklearn.utils import shuffle
from time import time

def recreate_image(codebook, labels, w, h):
    # Recreate the (compressed) image from the code book & labels
    d = codebook.shape[1]
    image = np.zeros((w, h, d))
    label_idx = 0
    for i in range(w):
        for j in range(h):
            image[i][j] = codebook[labels[label_idx]]
            label_idx += 1
    return image

def func(imgPath):
    n_colors = 64
    china2 = load_sample_image(imgPath)  # 加载图片

    #转换为浮点数，PLTIMSID行为在浮点数据上很好地工作
    china2 = np.array(china2, dtype=np.float64) / 255

    #将图片转成二维数组
    w, h, d = original_shape = tuple(china2.shape)
    assert d == 3
    image_array = np.reshape(china2, (w * h, d))

    print("一个小样本数据的拟合模型")
    t0 = time()
    image_array_sample = shuffle(image_array, random_state=0)[:1000]
    kmeans = KMeans(n_clusters=n_colors, random_state=0).fit(image_array_sample)
    print("done in %0.3fs." % (time() - t0))

    # Get labels for all points
    print("Predicting color indices on the full image (k-means)")
    t0 = time()
    labels = kmeans.predict(image_array)
    print("done in %0.3fs." % (time() - t0))

    # codebook_random = shuffle(image_array, random_state=0)[:n_colors + 1]
    # print("Predicting color indices on the full image (random)")
    # t0 = time()
    # labels_random = pairwise_distances_argmin(codebook_random,
                                            # image_array,
                                            # axis=0)
    # print("done in %0.3fs." % (time() - t0))


    # Display all results, alongside original image
    plt.figure(1)
    plt.clf()
    ax = plt.axes([0, 0, 1, 1])
    plt.axis('off')
    plt.title('Original image (96,615 colors)')
    plt.imshow(china2)

    plt.figure(2)
    plt.clf()
    ax = plt.axes([0, 0, 1, 1])
    plt.axis('off')
    plt.title('Quantized image (64 colors, K-Means)')
    plt.imshow(recreate_image(kmeans.cluster_centers_, labels, w, h))

    # plt.figure(3)
    # plt.clf()
    # ax = plt.axes([0, 0, 1, 1])
    # plt.axis('off')
    # plt.title('Quantized image (64 colors, Random)')
    # plt.imshow(recreate_image(codebook_random, labels_random, w, h))
    plt.show()

if __name__ == '__main__':
    imgPath = 'a.jpg'
    func(imgPath)
    # cv2.waitKey(0)
    # cv2.destroyAllWindows()
```