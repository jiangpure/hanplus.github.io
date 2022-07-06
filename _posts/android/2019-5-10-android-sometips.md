---
layout:     post
title:      "『Android』 开发中可以用到的技巧合集"
subtitle:   "整合一下，以备不时之需。"
date:       2021-05-10 18:05:20
author:     "purejiang"
header-img: "img/post-bg-1.jpg"
catalog: true
tags:
    - Android
---
> 一些在 Android 开发中可能会有用的技巧，整合一下，免得以后到处找。

---

**需求描述：**

APP 提审应用商店，需要去掉某些权限，但是引用的第三方库里包含了这些权限，又不好逐条删除，需要在打包后自动去掉这些权限。

**方法：**

- 方法 1：在 `AndroidManifest.xml` 中设置权限的属性为 `remove`，要记得添加 `tools` 的命名空间。

```xml
<uses-permission android:name="android.permission.CAMERA" tools:node="remove" />
```

- 方法 2：在 app 下的 `build.gradle` 中添加如下代码，通过 `gradle` 在打包过程中对 `AndroidManifest.xml` 进行修改。

```groovy
project.afterEvaluate {
    project.android.applicationVariants.all { variant ->
        variant.outputs.each { output ->
            output.processResources.doFirst { pm->
                String manifestPath = output.processResources.manifestFile;
                def manifestContent = file(manifestPath).getText()
                manifestContent = manifestContent.replace('<uses-permission android:name="android.permission.RECORD_AUDIO" />', '')
		manifestContent = manifestContent.replace('<uses-permission android:name="android.permission.READ_PHONE_STATE" />', '')
                file(manifestPath).write(manifestContent)
            }
        }
    }
}
```

**成果：**

- 方法 1：打包后检查权限，确认权限都被删除了。（推荐）

- 方法 2：会出现删除不了权限的情况。

---

**需求描述：**

希望打包后能够将 `apk/aab` 以指定的名称输出到指定的文件夹下。

**方法：**

在 app 下的 `build.gradle` 的 `android` 中添加如下代码，适用于新版的 Gradle。

```groovy
android.applicationVariants.all { variant ->
    variant.outputs.all {
        // 此处指定生成的apk文件名
        outputFileName = "${defaultConfig.applicationId}_${(new Date()).getTime()}.apk"
        // 此处指定生成 apk/aab 的路径
        variant.getPackageApplication().outputDirectory = new File("F:\apk")
    }
}
```

`outputFileName` 指定的就是输出的 `apk/aab` 名称。

`outputDirectory` 指定的就是输出的目录了。

PS：注意不能直接输出在盘符的路径下，只能输出在下级目录。

**成果：**

![输出apk/aab](/img/android/some_tips/0.png)

---

持续更新中...





