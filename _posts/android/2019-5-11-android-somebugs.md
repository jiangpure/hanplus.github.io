---
layout:     post
title:      "『Android』 开发中遇到的一些 Bug"
subtitle:   "这可不是我写的 bug！"
date:       2019-05-11 16:05:20
author:     "purejiang"
header-img: "img/post-bg-1.jpg"
catalog: true
tags:
    - Android
---

> 一些在 Android 开发中会遇到的典型的坑，在这里记录一下。
---

**问题简介：**

`EditText` 设置 `type` 为 `textPassword` 之后，字体间距会变宽。

**问题信息：**

![字体间距变宽](/img/android/some_bugs/0.png)

**解决办法：**

在代码中重新设置成默认字体。
```java
passwordText.setTypeface(Typeface.DEFAULT);
```
**效果：**

![Screenshot_20220324_162207.jpg](/img/android/some_bugs/1.jpg)

---
**问题简介：**

`targetVersion` 大于等于 `30` 时，使用 `V1` 签名过的 apk，在 `Android 11` 上会提示安装失败。

**问题信息：**

![安装失败](/img/android/some_bugs/2.png)

**解决办法：**

解决这个问题有如下两个方法。

- 打包的的时候将 `V1` 签名改为 `V2` 签名。

![V2 签名](/img/android/some_bugs/3.png)

- 降低包体的 `targetVersion` ，小于 `30` 即可。

**效果：**

![Screenshot_20220324_162207.jpg](/img/android/some_bugs/1.jpg)

---
持续更新中...




