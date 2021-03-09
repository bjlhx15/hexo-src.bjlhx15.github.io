---
title: 006-java-图片压缩Thumbnails
categories:
  - java-base
abbrlink: 815ac2f8
date: 2021-01-24 13:53:43
---

摘要：
   - Thumbnailator 是一个优秀的图片处理的Google开源Java类库。处理效果远比Java API的好。从API提供现有的图像文件和图像对象的类中简化了处理过程
   - Thumbnailator是一个用来生成图像缩略图的 Java类库，通过很简单的代码即可生成图片缩略图，也可直接对一整个目录的图片生成缩略图。
<!-- more -->

# jdk核心类

Thumbnailator官网：http://code.google.com/p/thumbnailator/

github地址：https://github.com/coobird/thumbnailator

## 首先添加依赖：
```xml
        <dependency>
            <groupId>net.coobird</groupId>
            <artifactId>thumbnailator</artifactId>
            <version>0.4.8</version>
        </dependency>
```

# 基础使用

```java
    Thumbnails.of("原图文件的路径")
        .scale(1f)
        .outputQuality(0.5f)
        .toFile("压缩后文件的路径");
```

## 输入

建造者模式
```
路径：Thumbnails.Builder<File> of(String... var0) 
文件类：Thumbnails.Builder<File> of(File... var0)
URL：Thumbnails.Builder<URL> of(URL... var0) 
输入流： Thumbnails.Builder<? extends InputStream> of(InputStream... var0)
图片Buffered：Thumbnails.Builder<BufferedImage> of(BufferedImage... var0) 
```

## 输出

```
到路径输出：ttoFile(String var1)
到Rename：oFiles(Rename var1)
到File：toFile(File var1)
到File及重命名：toFiles(File var1, Rename var2)
到多个：toFiles(Iterable<File> var1)

到输出流：toOutputStream(OutputStream var1)
到多个输出流：toOutputStreams(Iterable<? extends OutputStream> var1)
```

## 常规用法示例

```java
public class ThumbnailsTest {

    @Test
    public void handle() throws Exception {

        String inputPath = "src/main/resources/WechatIMG2.jpeg";
        String outputPathPrefix = "src/main/resources/img/WechatIMG_";
        String watermark = "src/main/resources/watermark.png";

        /**
         * size(width,height) 指定大小进行缩放，若图片横比200小，高比300小，不变
         * 若图片横比200小，高比300大，高缩小到300，图片比例不变 若图片横比200大，高比300小，横缩小到200，图片比例不变
         * 若图片横比200大，高比300大，图片按比例缩小，横为200或高为300
         */
        Thumbnails.of(inputPath).size(200, 300).toFile(outputPathPrefix + "200x300.jpg");
        Thumbnails.of(inputPath).size(2560, 2048).toFile(outputPathPrefix + "2560x2048.jpg");

        /**
         * scale(比例):按照比例进行缩放
         */
        Thumbnails.of(inputPath).scale(0.25f).toFile(outputPathPrefix + "25%.jpg");
        Thumbnails.of(inputPath).scale(1.10f).toFile(outputPathPrefix + "110%.jpg");

        /**
         * 不按照比例，指定大小进行缩放 设为false
         * keepAspectRatio(true) 默认是按照比例缩放的
         */
        Thumbnails.of(inputPath).size(120, 120).keepAspectRatio(false).toFile(outputPathPrefix + "120x120.jpg");

        /**
         * rotate(角度),旋转，正数：顺时针 负数：逆时针
         */
        Thumbnails.of(inputPath).size(1280, 1024).rotate(90).toFile(outputPathPrefix+"+90.jpg");
        Thumbnails.of(inputPath).size(1280, 1024).rotate(-90).toFile(outputPathPrefix+"-90.jpg");

        /**
         * watermark 水印 (位置，水印图，透明度)
         */
        Thumbnails.of(inputPath).size(1280, 1024)
                .watermark(Positions.BOTTOM_RIGHT, ImageIO.read(new File(watermark)), 0.5f)
                .outputQuality(0.8f).toFile(outputPathPrefix + "watermark_bottom_right.jpg");
        Thumbnails.of(inputPath).size(1280, 1024)
                .watermark(Positions.CENTER, ImageIO.read(new File(watermark)), 0.5f)
                .outputQuality(0.8f).toFile(outputPathPrefix + "watermark_center.jpg");

        /**
         * 裁剪
         */
        // 图片中心400*400的区域
        Thumbnails.of(inputPath).sourceRegion(Positions.CENTER, 400, 400).size(200, 200).keepAspectRatio(false)
                .toFile(outputPathPrefix + "region_center.jpg");
        //图片右下400*400的区域
        Thumbnails.of(inputPath).sourceRegion(Positions.BOTTOM_RIGHT, 400, 400).size(200, 200).keepAspectRatio(false)
                .toFile(outputPathPrefix + "region_bootom_right.jpg");
        //指定坐标
        Thumbnails.of(inputPath).sourceRegion(600, 500, 400, 400).size(200, 200).keepAspectRatio(false)
                .toFile(outputPathPrefix + "region_coord.jpg");

        /**
         * outputFormat 转化图像格式(图像格式)
         */
        Thumbnails.of(inputPath).size(1280, 1024).outputFormat("png").toFile(outputPathPrefix + "1280x1024.png");
        Thumbnails.of(inputPath).size(1280, 1024).outputFormat("gif").toFile(outputPathPrefix + "1280x1024.gif");

        /**
         * toOutputStream 输出到OutputStream(流对象)
         */
        OutputStream os = new FileOutputStream(outputPathPrefix + "1280x1024_OutputStream.png");
        Thumbnails.of(inputPath).size(1280, 1024).toOutputStream(os);

        /**
         * asBufferedImage() 输出到BufferedImage,返回BufferedImage
         */
        BufferedImage thumbnail = Thumbnails.of(inputPath).size(1280, 1024).asBufferedImage();
        ImageIO.write(thumbnail, "jpg", new File(outputPathPrefix + "1280x1024_BufferedImage.jpg"));
    }

}
```
工具源码本身最后还是调用jdk中的ImageIO.createImageOutputStream(fos);来实现的；

# 迭代压缩文件大小

将一个5M图像文件压缩至 指定大小的图片

## 工具代码

```java
public class CompressImg {
    public static ByteArrayOutputStream commpressPicForScale(InputStream inputStream, double size) throws Exception {
        ByteArrayOutputStream outputStream = new ByteArrayOutputStream();
        int available = inputStream.available();
        if (available < size * 1024) {
            inputToOut(inputStream, outputStream);
            return outputStream;
        }

        Thumbnails.of(inputStream)
                .outputQuality(0.5f)
                .scale(0.8f)
                .toOutputStream(outputStream);
        ByteArrayInputStream byteArrayInputStream = new ByteArrayInputStream(outputStream.toByteArray());
        return commpressPicForScale(byteArrayInputStream, size);
    }

    public static ByteArrayInputStream commpressPicForScaleInput(InputStream inputStream, double size) throws Exception {
        ByteArrayOutputStream outputStream = commpressPicForScale(inputStream, size);
        ByteArrayInputStream byteArrayInputStream = new ByteArrayInputStream(outputStream.toByteArray());
        return byteArrayInputStream;
    }

    public static void inputToOut(InputStream source, OutputStream target) throws IOException {
        byte[] buf = new byte[8192];
        int length;
        while ((length = source.read(buf)) > 0) {
            target.write(buf, 0, length);
        }
    }
}
```

## 测试代码
```java
public class CompressImgTest {

    /**
     * 压缩至500kb,输出流
     */
    @Test
    public void handle() throws Exception {
        File file = new File("src/main/resources/WechatIMG2.jpeg");
        InputStream inputStream = new FileInputStream(file);
        ByteArrayOutputStream byteArrayOutputStream = CompressImg.commpressPicForScale(inputStream, 500);
        try (OutputStream outputStream = new FileOutputStream("src/main/resources/WechatIMG2_new.jpeg")) {
            byteArrayOutputStream.writeTo(outputStream);
        }
    }

    /**
     * 压缩至300kb,输入流
     */
    @Test
    public void commpressPicForScaleInput() throws Exception {
        File file = new File("src/main/resources/WechatIMG2.jpeg");
        InputStream inputStream = new FileInputStream(file);
        ByteArrayInputStream inputStream2 = CompressImg.commpressPicForScaleInput(inputStream, 300);
        
        try (FileOutputStream fileOutputStream = new FileOutputStream("src/main/resources/WechatIMG2_new.jpeg")) {
            IOUtils.copy(inputStream2, fileOutputStream);
        }
    }
}
```

# 注意点

1、透明背景变黑
```
.imageType(BufferedImage.TYPE_INT_ARGB) //解决透明背景变黑问题
```
