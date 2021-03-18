---
layout: post
title: "UI 设计与屏幕适配"
date: 2021-03-17 15:12:26 +0800
comments: true
categories: [Archives, iOS Development, Android, Web Development]
keywords: px, dp, dpi, ppi, in
description: UI design and screen adaptation
---

目前移动设备的尺寸很多，所以前端 (iOS, Android, Web, 小程序等)开发需要适配多种尺寸屏幕。在适配时我们可能会有困惑，设计稿通常是 px 来表示尺寸，设备的分辨率也是以 px 来表示，它们之间是一样的吗？还是存在什么关系？iOS 开发者用 point 来表示视图的尺寸； Android 开发者用 dp 来表示视图尺寸； Web 开发者用 px 来表示尺寸？它们和设计稿的 px 是什么关系？设计师应该以什么基准尺寸来设计会有利于屏幕适配？应该输出几套切图？除了切图，设计师还可以做些什么来配合开发者做屏幕适配？要搞清楚这些问题，我们需要翻翻历史了。  

Tim Chien 和 Robert Nyman 的这篇 [CSS Length Explained](https://hacks.mozilla.org/2013/09/css-length-explained/) 帮了我的大忙，本文就是基于它而写成。

##英寸
我们经常是用英寸为度量单位来表示手机屏幕尺寸，一英寸相当于2.54厘米或0.0254米。

## 设备像素(device pixel)

计算机屏幕显示事物的单位是像素。显示屏上的单个物理 "光点"，能够独立于它的邻居显示出完整的颜色，被称为像素（图片元素）。我们把屏幕上的物理像素称为 "设备像素"。

##DPI, PPI
DPI 是 dots per inch 的英文缩写，即每英寸点数； PPI 是 pixels per inch 的缩写，即每英寸像素。 它们都用来表示显示像素密度 (Display pixel density)。

计算机屏幕是由大量发光二极管整齐排列构成的集成电路，由于屏幕制造商工艺水平差异，每英寸集成电路上排列的二极管的数量会不一样，屏幕出厂时我们可以从厂商那里得知屏幕的 PPI。  

于是我们可以知道:  

```
width or height of one device pixel = 1 / device's DPI  
```

例如 MacBook Air(2011) 的 DPI 为 125 ， 所以：  

```
(width or height of one device pixel) = 1/125 inch = 0.008 inch = 0.02032 cm
```
<!--more-->
##The CSS pixel (px)

CSS像素的尺寸大致可以看成是人的肉眼能够舒适地看到的尺寸，不要太小，这样你就得眯着眼睛，也不要大到让你看到像素化。"看得很舒服 " 的定义比较笼统，[W3C CSS规范](http://www.w3.org/TR/CSS2/syndata.html%23length-units)中给我们一个推荐的参考。  

>The reference pixel is the visual angle of one pixel on a device with a pixel density of 96 DPI and a distance from the reader of an arm’s length.

##The viewing distance

如前所述，观看距离因人而异，因设备而异，这就是为什么我们必须将设备按外形因素分类的原因。推荐的参考观看距离("一臂之长")和参考像素密度("96 DPI")其实是历史数据。  

对于21世纪的日常设备，我们有不同的参考建议:  

| Device | Baseline pixel density | Width/height of one CSS pixel | Viewing distance|
| --- | --- | --- | --- |
| A 20th century PC with CRT display | 96 DPI | ~0.2646 mm (1/96in) | 28 in (71.12cm) |
| Modern laptop with LCD | 125 DPI | 0.2032 mm (1/125in) | 21.5 in (54.61cm)| 
| Smartphones/Tablets | 160 DPI | ~0.159mm (1/160 in) | 16.8in (42.672cm) |

因此，我们在 CSS 的世界里建立了一个基本的事实：一个 CSS 像素会以不同的物理尺寸显示，但它总是以正确的尺寸显示，让浏览者感到舒适。  

##Device pixel ratio (DPPX)

随着我们步入未来，现在很多智能手机在出厂时都采用了高密度的显示屏。为了保证 CSS 像素在每一个访问网络的设备(即一切有屏幕和网络连接的设备)上的尺寸一致，设备制造商不得不将多个设备像素映射到一个 CSS 像素上，以弥补它相对更大的物理尺寸。CSS 像素相对于设备像素的尺寸比就是设备像素比(DPPX)。 

我们以 iPhone 4 为最著名的例子。它配备了一块 326 DPI 的显示屏。根据我们上面的表格，作为一款智能手机，它的典型观看距离是 16.8 英寸，它的基准像素密度是 160DPI。为了创建一个 CSS 像素，苹果选择将设备像素比设置为 2，这就等于让 iOS Safari 显示网页的方式和 163 DPI 手机上一样。  

在我们继续之前，先回头看看上面的数字。其实我们可以做得更好，不把设备像素比设置为2，而是设置为`326/160=2.0375`，让一个 CSS 像素与参考尺寸相比完全一样。不幸的是，这样的比例会导致一个意想不到的结果：由于每个 CSS 像素并不是由整个设备像素来显示的，所以浏览器不得不对所有的位图图像、边框等进行反锯齿，因为它们几乎总是被当作整个 CSS 像素来显示。浏览器很难利用2.0375个设备像素来绘制你的1个CSS像素宽的边框：如果比例是简单的2，那就容易多了。  

顺带一提，163 DPI恰好是上一代 iPhone 的像素密度，所以网页的工作方式也是一样的，不需要开发者对自己的网站进行任何特殊的"升级"。  

设备制造商通常选择1.5，或2，或其他整数作为 DPPX 值。偶尔，有些设备决定不这么玩了，发货时使用1.325 DPPX这样的值；作为开发者，我们也许应该忽略这些设备。  

现在我们就比较清楚 CSS pixel 和 device pixel 的关系了。接下来我们看下 iOS 的 point 和 device pixel 的关系。

##point

> The coordinate system iOS uses to place content onscreen is based on measurements in points, which map to pixels in the display. A standard-resolution display has a `1:1` pixel density (or `@1x`), where one pixel is equal to one point. High-resolution displays have a higher pixel density, offering a scale factor of 2.0 or 3.0 (referred to as `@2x` and `@3x`). As a result, high-resolution displays demand images with more pixels.

从 Apple 这段描述可知， scale factor (`@1x`, `@2x` 和 `@3x`) 就是我们上面据说的设备像素比（DPPX)。point 和 css pixel 是对应的。

##dp

那 dp 和 device pixel 又是什么关系呢？  

> To preserve the visible size of your UI on screens with different densities, you must design your UI using density-independent pixels (dp) as your unit of measurement. One dp is a virtual pixel unit that's roughly equal to one pixel on a medium-density screen (160dpi; the "baseline" density). Android translates this value to the appropriate number of real pixels for each other density.

Google 这段描述更加直接，dp 是一个虚拟的像素单位，大致相当于中密度屏幕上的一个像素(160dpi;"基线"密度)，所以 dp 和 css pixel 也是对应的。而 xhdpi, xxhdpi 和 xxxhdpi 是表示设备像素比(DPPX)2、3 和 4。

现在我们还剩下设计稿的 px。我们回忆一下在前端开发时，如果我们不指定图片尺寸而直接去显示设计师的切图，这时图片是有一个固有尺寸的，在设备像素比为1的设备上，这个固有尺寸就是图片的尺寸，而在设备像素比为2上尺寸是图片的尺寸除以2，所以设计稿的 px是对应设备像素(device pixel)的，这也是为什么我们需要提供多套图片来做适配。假设我们不提供多套图片，现在我们有一个 `100 x 100 css pixel`的图片， 在设备像素比为3的设备上也会去加载 `100 x 100 device pixel` 尺寸的资源图，按上面的分析，实际它应该加载 `300 x 300 device pixel` 尺寸的资源图，那么相当于资源图上一个像素点会对应显示三个设备像素点，这样可能会出现模糊或锯齿的情况。

理清了各平台尺寸单位的关系以及它们与设备像素的关系后，我们来看下设备尺寸。

##设备尺寸

我们先看下 iOS 设备尺寸分布:  

| 型号 | points | 物理像素 | 设备像素比(DPPX) |
| --- | --- | --- | :---: |
| 2G,3G,3GS | 320 x 480 | 320 x 480| 1 |
| 4,4S | 320 x 480 | 640 x 960 | 2 |
| 5,5C,5S,SE | 320 x 568 | 640 x 1136 | 2 |
| 6,6S,7,8,SE2| 375 x 667 | 750 x 1334 | 2 |
| 6+,6S+,7+,8+ | 414 x 736 | 1080 x 1920 | 3 |
| 11Pro,X,Xs | 375 x 812 | 1125 x 2436 | 3 |
| 11, Xr | 414 x 896 | 828 x 1792 | 2 |
| 11Pro Max,Xs Max | 414 x 896 | 1242 x 2688 | 3 |
 

对于 iOS 来说，现在的主流设备应该是从 `6,6S,7,8,SE2` 开始，对应的设备像素是`750 x 1334 px`。  

再来看下 android 这边， Google 有一个 [Screen sizes and densities](https://developer.android.com/about/dashboards/index.html#Screens) 统计表，本文写作时查询的结果如下:  

| | ldpi | mdpi | tvdpi | hdpi | xhdpi | xxhdpi | Total |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Small | 0.1% |  |  | | 0.1% |  | 0.2% | 
| Normal |  | 0.3% | 0.3% | 14.8% | 41.3% | 26.1% | 82.8% | 
| Large |  | 1.7% | 2.2% | 0.8% | 3.2% | 2.0% | 9.9%  | 
| Xlarge |  | 4.2% | 0.2% | 2.3% | 0.4% |  | 7.1% | 
| Total | 0.1% | 6.2% | 2.7% |  17.9% | 45.0% | 28.1% |  | 

[Small,Normal,Large 和 Xlarge](https://developer.android.google.cn/guide/topics/resources/providing-resources#ScreenSizeQualifier) 是屏幕的尺寸分类，具体含义如下:

> * small: Screens that are of similar size to a low-density QVGA screen. The minimum layout size for a small screen is approximately 320x426 dp units. Examples are QVGA low-density and VGA high density.

> * normal: Screens that are of similar size to a medium-density HVGA screen. The minimum layout size for a normal screen is approximately 320x470 dp units. Examples of such screens a WQVGA low-density, HVGA medium-density, WVGA high-density.

> * large: Screens that are of similar size to a medium-density VGA screen. The minimum layout size for a large screen is approximately 480x640 dp units. Examples are VGA and WVGA medium-density screens.  
> 	
> * xlarge: Screens that are considerably larger than the traditional medium-density HVGA screen. The minimum layout size for an xlarge screen is approximately 720x960 dp units. In most cases, devices with extra-large screens would be too large to carry in a pocket and would most likely be tablet-style devices. Added in API level 9.

从上表的数据可知，目前 android 设备的主流尺寸分布是从 normal-hdpi 这个分类开始，根据 google 对 normal 的解释，它的大小相当于 medium-density HVGA 屏幕上的 `320x470 dp`，换算成设备像素就是 `480x705 px`。

我们知道宽屏比窄屏能显示更多内容，如果我们以宽屏为其准尺寸设计，那么在窄屏上就可能出现控件放不下、文字截断的情况。反过来，如果我们以窄屏为基准设计，那么在宽屏上布局时会容易处理，控件的宽度增加或者间隔增加就可以了。高度和宽度存在同样的问题，所以也应该选高度小的作为基准。设计时扣除固定元素高度之和后分配给可滚动区域，这样方便界面的元素布局能够动态响应，开发更好做屏幕适配。

所以选择基准尺寸和我们想支持的设备紧密相关，这需要基于多方面的因素考虑。 例如，如果我们希望支持尽可能多的设备，就越有可能获取更多用户，但开发的兼容工作量就相应增加，很多新特性就可能不适合作为应用的主要功能，而只适合作为增强功能。通常可以考虑覆盖 90% 以上，团队资金和人员比较充足的话可以考虑覆盖 95%，98% 甚至更多。  

以覆盖 90% 以上为例，如果我们同时支持 iOS 和 android，或只支持 android 时，应该选 `480x705 px`作为基准尺寸，而如果只支持 iOS ， 我们应该选 `750 x 1334 px` 作为基准尺寸。

iOS 的设备像素比主要分布在2和3，而 android 这边设备像素比主要分布在 1.5(hdpi), 2(xhdpi)和 3(xxhdpi)，所以 iOS 需要输出`@2x` 和 `@3x` 两套切图； android 需要输出 hdpi, xhdpi 和 xxhdpi 三套切图。


##总结

现在我们知道，设计基准尺寸的选择以及切图的输出是和我们想支持的设备紧密相关，写作本文时：

###基准尺寸

* 仅支持 iOS ， 应该选 `750 x 1334 px` 作为基准尺寸
* 仅支持 android 时，应该选 `480x705 px` 作为基准尺寸
* 支持 iOS 和 android，应该选 `480x705 px` 作为基准尺寸

### 切图

* 支持 iOS 需要输出`@2x` 和 `@3x` 两套切图
* 支持 android 需要输出 hdpi, xhdpi 和 xxhdpi 三套切图

我们需要需要注意，随着设备的更新换代，我们的基准尺寸和切图会发生变化，就像以前我们可能需要为 android 提供 mdpi 的切图。  

另外想说一下，设计师在设计之初就要把屏幕适配这事放在心上，将界面的元素看成水流一样，尽量让它们能自由流动，这样开发者就能更好地也更容易地做屏幕适配。Apple 在屏幕适配这块提出了 auto layout 的解决方案，这是一个设计师视角的解决方案，也是我们日常的生活中的视角，用界面元素的之间的约束来表达布局，推荐设计师用约束这种方式去做设计并最终输出。可以看到 google 实际上也很认可 auto layout 用约束来布局的想法，在新版本的 android 开发中默认的根布局容器就是 ConstraintLayout，它就是用约束来表达布局。最后我们再看 web 开发布局这边，css 布局的核心就是流，为支持屏幕适配，目前的主流方案是响应式布局，而这种布局的核心我认为仍然是约束。可以看到在屏幕适配这块，各平台最终的想法其实是一样的。

#Reference  

* [CSS Length Explained](https://hacks.mozilla.org/2013/09/css-length-explained/)  
* [Image Size and Resolution](https://developer.apple.com/design/human-interface-guidelines/ios/icons-and-images/image-size-and-resolution/)  
* [Support different pixel densities](https://developer.android.google.cn/training/multiscreen/screendensities)  
* [ScreenSizeQualifier](https://developer.android.google.cn/guide/topics/resources/providing-resources#ScreenSizeQualifier)  
* [Support different screen sizes](https://developer.android.google.cn/training/multiscreen/screensizes)  
* [Screen sizes and densities](https://developer.android.com/about/dashboards/index.html#Screens)  
* [The Ultimate Guide To iPhone Resolutions](https://www.paintcodeapp.com/news/ultimate-guide-to-iphone-resolutions)  

