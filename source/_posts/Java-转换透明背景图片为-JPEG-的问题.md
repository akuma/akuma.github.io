---
title: Java 转换透明背景图片为 JPEG 的问题
date: 2016-12-10 19:24:57
tags: java image png jpeg
---

最近遇到一个 Java 转换透明背景图片为 JPEG 图片后背景变黑的问题。

具体情况是这样的，我们有个产品的安卓端将某个 WebView 截图后上传给服务端，服务端为了节省存储空间和带宽，会对图片进行缩放处理。而某些安卓机型上传的 JPEG 图片被服务端处理后变成了黑色。我用图像处理工具查看这些图片，发现它们其实只是背景变黑了。

<!--more-->

## 问题原因分析

通过分析安卓端和服务端的相关代码以及 Google 搜索相关资料，我终于发现了问题所在。

首先简单说一下 PNG 和 JPEG 的一些区别。JPEG 是一种有损压缩的图片格式，以 24 位颜色存储单个位图，不支持动画、不支持透明色。PNG 是一种无损压缩的图片格式，有 8 位、24 位、32 位三种形式，其中 8 位 PNG 支持两种不同的透明形式（索引透明和 Alpha 透明），24 位 PNG 不支持透明，32 位 PNG 在 24 位基础上增加了 8 位透明通道。

我们安卓端的截图是 32 位的 PNG 图片，但命名时的后缀名却是 `.jpg`。经过测试，有些安卓机型所截图片的背景色是白色的，有些则是透明的。我们的服务端接受到图片后，会根据图片的后缀名进行处理，目的是保持原有的格式。在这种情况下，对于背景透明的 PNG 图片，因为被服务端当做 JPEG 处理，所以 32 位色彩被强制转换成了 24 位色彩，背景就变黑了。

## 问题解决办法

知道了问题产生的原因后就好办了。我最后的解决办法是修改了服务端的图片处理代码，在处理原图片时判断图片的背景是否透明，如果背景透明而目标文件又是 JPEG 格式，则通过处理丢弃原图片中的 Alpha 通道。具体的实现代码如下：

```java
/**
 * 如果目标图片是 JPEG 图片并且图像是透明的，就丢弃 Alpha 通道，否则会导致转换出来的 JPEG 图片背景黑色。
 */
private static BufferedImage discardAlphaChannelForJpeg(BufferedImage image, String fileType) {
    if (isJpegImage(fileType) && image.getTransparency() == Transparency.TRANSLUCENT) {
        image = get24BitImage(image);                 // 第一种方式
        // image = get24BitImage(image, Color.BLACK); // 第二种方式，可以指定背景色
    }
    return image;
}

/**
 * 根据文件后缀名判断是否是 JPEG 图片。
 */
private static boolean isJpegImage(String fileType) {
    return "jpg".equalsIgnoreCase(fileType) || "jpeg".equalsIgnoreCase(fileType);
}

/**
 * 使用删除 Alpha 值的方式去掉图像的 Alpha 通道。
 */
private static BufferedImage get24BitImage(BufferedImage image) {
    int w = image.getWidth();
    int h = image.getHeight();
    int[] argb = getRGBs(image.getRGB(0, 0, w, h, null, 0, w));
    BufferedImage newImage = new BufferedImage(w, h, BufferedImage.TYPE_INT_RGB);
    newImage.setRGB(0, 0, w, h, argb, 0, w);
    return newImage;
}

/**
 * 使用绘制的方式去掉图像的 Alpha 值。
 */
@SuppressWarnings("unused")
private static BufferedImage get24BitImage(BufferedImage image, Color bgColor) {
    int w = image.getWidth();
    int h = image.getHeight();
    BufferedImage newImage = new BufferedImage(w, h, BufferedImage.TYPE_INT_RGB);
    Graphics2D graphic = newImage.createGraphics();
    graphic.setColor(bgColor);
    graphic.fillRect(0, 0, w, h);
    graphic.drawRenderedImage(image, null);
    graphic.dispose();
    return newImage;
}

/**
 * 将 32 位色彩转换成 24 位色彩（丢弃 Alpha 通道）。
 */
private static int[] getRGBs(int[] argbs) {
    int[] rgbs = new int[argbs.length];
    for (int i = 0; i < argbs.length; i++) {
        rgbs[i] = argbs[i] & 0xffffff;
    }
    return rgbs;
}
```
