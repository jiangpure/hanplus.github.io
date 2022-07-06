---
layout:     post
title:      "『Android』 能在 Android 开发中用到的 Github 项目"
subtitle:   "持续更新 Github 上有用的项目。"
date:       2018-08-29 13:00:00
author:     "purejiang"
header-img: "img/post-bg-1.jpg"
tags:
    - Android
---

> 说明：有关这些第三方库的最新依赖，可以自行到 GitHub 上去搜索，添加相应版本即可。

**[Okhttp](https://github.com/square/okhttp)**：一个处理网络请求的开源项目，是安卓端最火热的轻量级网络框架，由 Square 贡献。

简介：用于替代`HttpUrlConnection`和`Apache HttpClient`(android API23  6.0里已移除 `HttpClient`)。

`implementation 'com.squareup.okhttp3:okhttp:3.9.1'`
	
***

**[Retrofit](https://github.com/square/retrofit)**：时下非常火的网络请求框架，也由 Square 贡献。

简介：该库基于 `Http` 协议，封装了 `okHttp` 的网络请求框架，与 `RxJava` 完美结合，现在是 2.0 版本，也就是大家常说的 `Retrofit2`。

`implementation 'com.squareup.retrofit2:retrofit:2.5.0'`

***

**[OkGo](https://github.com/jeasonlzy/okhttp-OkGo)**：一个基于 `okhttp` 的标准 RESTful 风格的网络框架，借鉴了 `Retrofit` 的思想。

简介：该库基于 `Http` 协议，封装了 `okHttp ` 的网络请求框架，支持 `RxJava`，`RxJava2`，支持自定义缓存，支持批量断点下载管理和批量上传管理功能，虽然借鉴了 `retrofit`，但是也有突出的地方，比如说有 `retrofit` 所没有的下载进度监听等。

`implementation 'com.lzy.net:okgo:3.0.4'`

***

**[CircleImageView](https://github.com/hdodenhof/CircleImageView)**：A fast circular ImageView perfect for profile images.

简介：可以用来实现开发中常使用到的圆形用户头像，一个自定义快速圆形 `ImageView`，非常实用。

`implementation 'de.hdodenhof:circleimageview:2.2.0'`

***

**[RxJava](https://github.com/ReactiveX/RxJava)**：`RxJava` 在 GitHub 主页上的自我介绍是 “A library for composing asynchronous and event-based programs using observable sequences for the Java VM.”。

简介：一个在 `Java VM` 上使用可观测的序列来组成异步的、基于事件的程序的库，现在是 2.0 版本，也就是大家常说的 `RxJava2`。

`implementation 'io.reactivex.rxjava2:rxjava:2.x.y'`

***

**[Gson](https://github.com/google/gson)**：提供 `JSON` 数据转换的库，由 Google 开源。

简介：`Gson` 是一个 `Java` 库，可以用来将 `Java` 对象转换成他们的 `JSON` 表示。它还可以用于将一个 `JSON` 字符串转换为一个等价的 `Java` 对象。`Gson` 可以使用任意的 `Java` 对象，包括那些你没有源代码的预先存在的对象。

`implementation 'com.google.code.gson:gson:2.8.2'`

***

**[LitePal](https://github.com/LitePalFramework/LitePal)**：一款封装完善的 `SQLite` 框架。

简介：`LitePal`郭神力荐，一款非常方便的 `SQLite` 框架，采用了对象关系映射 `(ORM)` 的模式，将平时开发时最常用的一些数据库功能进行了封装，使得开发者不用编写一行 `SQL` 语句就可以完成各种建表、増删改查的操作。并且 `LitePal` 很“轻”，`jar` 包大小不到 100k，而且近乎零配置。

`implementation 'org.litepal.android:core:1.6.1'`

***

**[Android-PickerView](https://github.com/Bigkoo/Android-PickerView)**：一个仿 `ios` 风格的多项滑动选择器。
简介：`Android-PickerView`支持时间和自定义选项，支持三级联动和自定义颜色字体间距等等，使用简单。

`implementation 'com.contrarywind:Android-PickerView:4.1.7'`

***

**[Picasso](https://github.com/square/picasso)**：`Picasso` 是 Square 开源的 `Android` 端的图片加载和缓存框架。
简介：一个适用于 Android 的强大图像下载和缓存库，在使用最小的内存 来做复杂的静态图片变换方面强无敌，但是不支持 `GIF` 是硬伤，上手简单易用。

`implementation 'com.squareup.picasso:picasso:2.71828'`

***

**[Glide](https://github.com/bumptech/glide)**：`Glide` 是 Google 官方推荐的图片加载框架，其更注重加载的流畅性。

简介：和 `Picasso` 一样为图片加载而生，但是 `Glide` 是一个快速高效的 Android 图片加载库，更注重于平滑的滚动，与 `Picasso` 各有千秋吧，图片加载速度以及内存占比还是更胜一筹的。

`implementation 'com.github.bumptech.glide:glide:4.9.0'`

***

**[Fresco](https://github.com/facebook/fresco)**：由 Facebook 维护的图片加载框架。
简介：其特点是支持框架渐进式 `JPEG` 流式传输、显示动画 `GIF` 和 `WebP`、广泛的图像加载和显示定制，流畅性与 `Glide` 相近，但是使用难度略高，加载中的 `OOM` 概率明显低于 `Glide`。

`implementation 'com.facebook.fresco:fresco:2.0.0'`

***

**[AndroidUtilCode](https://github.com/Blankj/AndroidUtilCode)**：AndroidUtilCode is a powerful & easy to use library for Android.

简介：`AndroidUtilCode` 是一个工具类的集合，其中 `utilcode` 的工具类都是开发中常用到的，极大地避免了我们重复造轮子。

`木有远程依赖~`

***

**[JiaoZiVideoPlayer](https://github.com/lipangit/JiaoZiVideoPlayer)**：节操播放器，高度自定义的 `Android` 播放框架.

简介：可以完全自定义 UI 和任何功能，一行代码切换播放引擎，支持的视频格式和协议取决于播放引擎，支持大小窗播放等等。

`implementation 'cn.jzvd:jiaozivideoplayer:7.0.5'`

***

**[IJKPlayer](https://github.com/bilibili/ijkplayer)**： Bilibili 开源的基于 `FFmpeg` 的播放器框架。

简介：实现了跨平台功能，API 易于集成，编译配置可自定义，方便控制安装包大小和各种需要。总之还是特别好用的，有兴趣的话可以看看我的基于 `IjkPlayer` 的开源项目：[Jvideoview](https://github.com/nicejiang/Jvideoview)。

```
# 这两个依赖项可以支持大多数设备，如果有其他需求请去 github 上自取
implementation 'tv.danmaku.ijk.media:ijkplayer-java:0.8.8'
implementation 'tv.danmaku.ijk.media:ijkplayer-armv7a:0.8.8'
```

***

**[zxing](https://github.com/zxing/zxing)**：zxing 是 Google 开源的多格式 `1D / 2D` 条码图像处理库，以 `Java` 实现，并且可以支持其他语言。
简介：总之在 `Android` 上对条形码和二维码进行处理肯定少不了这个强大的库，很多市面上的应用都是用 `zxing` 进行二维码处理的，zxing 的接入可以看看这位老哥的文章：[zxing 的集成...](https://www.jianshu.com/p/a4ba10da4231)。

```
implementation 'com.google.zxing:android-core:3.3.0'
implementation 'com.google.zxing:core:3.3.2'
```

***

**[ShimmerRecyclerView](https://github.com/sharish/ShimmerRecyclerView)**：`ShimmerRecyclerView` 是一个仿支付宝加载数据的时候显示闪烁效果的 `RecyclerView`。

简介：这个 `RecyclerView` 可以通过以灰白的条或者块展示正在加载的视图。而且它提供了两种方法控制演示的视图和实际的元素之间的切换。

```
repositories {
    jcenter()
    maven { url "https://jitpack.io" }
}

dependencies {
    implementation 'com.github.sharish:ShimmerRecyclerView:v1.3'
}
```

***

#### *持续更新中...*