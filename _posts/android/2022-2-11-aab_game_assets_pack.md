---
layout:     post
title:      "『Android』 游戏 aab 包上传谷歌，提示超过 150 MB 的处理"
subtitle:   "处理游戏 aab。"
date:       2022-01-28 09:11:22
author:     "purejiang"
header-img: "img/post-bg-1.jpg"
catalog: true
tags:
    - Android
---

>游戏 aab 包上传谷歌，提示超过 150 MB 时需要进行应用的资源分发，所以在重打包前要做好 Play Asset Delivery（游戏资源分发）的处理  。

#### 一、Play Asset Delivery
关于 [Play Asset Delivery](https://developer.android.google.cn/guide/app-bundle/asset-delivery) 的详细介绍可以看官网文档，这里不再赘述。

#### 二、资源分发的处理
由于接触的业务是 Unity 游戏，所以是 Unity 游戏 + Gradle 打包的方式；这里以 Android Studio 为打包工具。
- 首先新建一个或多个 Android Library。
![新建 Android Library](/img/android/aab_game_assets_pack/1.png)
- 将游戏的资源文件，也就是 `assets` 中的 `AssetBundles` 文件夹，放入 `src/main/assets` 中，如果有多个 Library 需要注意分割好资源，**注意不要将游戏配置文件放进去**。

- 其次配置存放资源的 Android Library 的 build.gradle 文件如下，**packName** 则是 Android Library 的名称，**deliveryType** 则是资源分发模式。

```groovy
apply plugin: 'com.android.asset-pack'

assetPack {
    packName = "assetsPackGameRes" // Directory name for the asset pack
    dynamicDelivery {
        deliveryType = "install-time"
    }
}
```

资源分发模式分为三种：

`install-time` 资源包在用户安装应用时分发。如果资源是进入游戏必要的，最好使用这种模式。
`fast-follow` 资源包在安装应用后自动下载。
`on-demand`  资源包在应用运行时下载

这里由于分发资源都是进入游戏所必须的，所以使用 `install-time`（用户安装时分发），其他分发模式请参考官方文档。

- 然后在 app 的 build.gradle 中添加资源包配置，**assetsPacks** 是存放资源的 Android Library 的名称列表，可以配置多个资源包，**注意分号别漏了**。

```groovy
plugins {
    //...
}

android {
   //...
    defaultConfig {
        //...
       assetPacks = [':assetsPackGameRes', ':assetsRes2']
    }
}

dependencies {
  //...
}
```

- 然后进行 aab 的打包、安装和测试，该步骤可以参考[《aab打包、安装测试以及注意事项》](https://purejiang.gitee.io/2022/01/25/aab_package/)。

- 最后，通过对比配置资源分发后的 aab 和原来的 aab ，aab 的总体积无变化，不是说要小于150M 吗？

别急，先解包可以发现，配置分发后的 aab 中多了资源包的文件夹，而且资源包占了绝大部分的体积 。本来体积很大的 base 现在只有不到100M，**只要 base 不大于150M 就可以顺利提交审核了**。

![未做资源分发的 aab 结构](/img/android/aab_game_assets_pack/2.png)

![配置资源分发后的 aab 结构](/img/android/aab_game_assets_pack/3.png)

#### 三、注意事项
- 审核提示超过 150 MB，是指当用户下载应用时，安装应用所需的压缩 APK（例如，基本 APK + 配置 APK）的总大小不得超过 150 MB；配置了资源分发后，Asset Pack 也就是 **资源包** 的大小不计入其中。
- Unity 游戏在 aab 安装后出现黑屏或者 **not enough storage space to install** 的提示，需要注意要将游戏启动相关的配置文件放在 app 目录下，而不是放在 **资源包** 中，例如 Unity 游戏的 `assets` 中的 `bin` 文件夹和 config 配置等。




