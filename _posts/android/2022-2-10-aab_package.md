---
layout:     post
title:      "『Android』 aab 打包、安装测试以及注意事项"
subtitle:   "全面实行，谷歌 aab 包。"
date:       2022-01-25 14:12:00
author:     "purejiang"
header-img: "img/post-bg-1.jpg"
catalog: true
tags:
    - Android
---

> 由于 Google Play 上架的新规，所以现在新应用的上架都需要用 aab ，在此记录一下 aab 相关的经验。

本文主要分为以下部分：
 - aab 打包
 
 - 安装测试
 
 - 注意事项

#### 一、aab 打包
Android App Bundle（aab） 是谷歌新的安卓安装文件，其实也就是根据 cpu 架构和语言等，切分多个 apk 以减少包体体积，aab 打包有以下两种方式。
- AS 打包
Android Studio 打包，类型直接选择 Android App Bundle，然后选择签名等步骤，即可打包 aab。
![构建签名 bundle 或者 apk](/img/android/aab_package/1.png)
![构建 aab](/img/android/aab_package/2.png)

- 命令行打包
Gradle 加入环境变量，在 app 的 `build.gradle` 文件中配置好签名，通过命令行  ` gradle bundle ` 或者 `gradlew bundle`  进行 aab 打包。

#### 二、安装测试
aab 是不能直接安装的，需要上传到 Google Play 后台，通过商店下载安装测试，不过其本质还是安装 apk。我们也可以通过谷歌提供的 [bundletool](https://developer.android.google.cn/studio/command-line/bundletool) 进行 aab 的本地安装测试，而不需要上传到 Google Play 后台。
- 首先[在此处下载](https://github.com/google/bundletool/releases) `bundletool`。
![bundletool的github仓库](/img/android/aab_package/3.png)

- 然后通过 `bundletool` 将 aab 转为一组 apk，也就是 apks，签名配置可不填，不填则使用默认的 debug 签名。

```
java -jar [ bundletool 文件] build-apks --bundle [ aab 文件] --output [ apks 文件]
 --ks=[签名文件]
 --ks-pass=[签名密码]
 --ks-key-alias=[别名]
 --key-pass=[别名密码]
```

**例：**`java -jar bundletool.jar build-apks --bundle app-realease.aab --output app-output.apks --ks=d:\test.keystore --ks-pass=123456 --ks-key-alias=com.test.app --key-pass=123456`

- 再通过 `bundletool` 将 apks 安装到真机。

```
java -jar [ bundletool 文件] install-apks --apks [ apks 文件]
```

**例：**`java -jar bundletool.jar install-apks --apks app-output.apks`

- 最后等待应用安装完成即可。
#### 三、注意事项

- 测试安装时尽量只使用一台手机连接 usb 调试。

- 谷歌规定 aab 里面的 base 文件夹不能超过 150 MB 大小，超过 150 MB 需要进行应用的资源分发，游戏的 aab 的 base 文件夹一般都超过了 150 MB，所以在打包前要做好资源分发的处理，资源分发的处理可见[《游戏 aab 包上传谷歌，提示超过 150 MB 的处理》](https://purejiang.gitee.io/2022/01/28/aab_game_assets_pack/)。
