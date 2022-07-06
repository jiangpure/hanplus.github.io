---
layout:     post
title:      "『Android』 Google Play 上架流程（一）"
subtitle:   "上架的前期准备工作。"
date:       2021-09-17 13:14:00
author:     "purejiang"
header-img: "img/post-bg-1.jpg"
catalog: true
tags:
    - Android
---

>最近公司要做海外发行业务，所以接入了 Google Play 的 SDK，在此记录一下接入经验，避免以后踩坑。
上架谷歌商店主要分三大部分：

[一、**前期准备**](https://purejiang.gitee.io/2021/09/17/googleplay_shelf_1/)

[二、**SDK接入**](https://purejiang.gitee.io/2021/12/21/googleplay_shelf_2_1/)

[三、**提审上架**](https://purejiang.gitee.io/2021/12/27/googleplay_shelf_3)


 本文内容为前期准备工作，主要分为**准备参数**、**Google API 后台新建项目**、**Google Play 控制中心新建应用**三部分，接入和提审内容请移步至后续文章。
 
---
### 准备参数
- 提前定好应用包名
- 提前准备好应用签名的 `SHA1` 值，由于 Google Play 上架规则的调整原因，2021 年 8 月起，新应用需要在 Google Play 后台签名，所以最好自己新建签名文件上传或者在 Google Play 后台创建签名。
Android Studio 中获取 `SHA1` 值可参考[百度地图 SDK 的简单接入]。(https://www.jianshu.com/p/f1d447760fbf)

### 在 Google API 后台新建项目
- 需要使用谷歌账号登录，参考 [Google 登录官方文档](https://developers.google.cn/identity/sign-in/android/start-integrating)，通过页面中的配置项目可以快速新建项目。

![快捷配置项目](/img/android/googleplay_shalf_1/1.png)

- 保存好相关参数，可下载携带参数的 `.json` 文件。

![保存 google 参数](/img/android/googleplay_shalf_1/2.png)

- 也可以在Google API 后台创建新项目，然后在凭证页面新建凭证，一般是新建 `OAuth 客户端 ID`。

![创建凭据](/img/android/googleplay_shalf_1/3.png)

![新建 OAuth 客户端](/img/android/googleplay_shalf_1/4.png)

- 进入[ Google API 后台](https://console.cloud.google.com/)，可以查看自己的项目配置，一般会使用到 *Web Client 的参数*。

![API服务](/img/android/googleplay_shalf_1/5.png)

### 在 Google Play 管理中心新建应用
- 注册 Google Play 开发者账号，并使用支持海外支付的信用卡支付 25 美元。
- 需要使用到 Google Play 结算服务，所以要在 [Google Play 管理中心](https://developer.android.google.cn/distribute/console)中，根据[官方文档](https://support.google.com/googleplay/android-developer/answer/9859152?hl=zh-Hans)新建应用。

![新建应用](/img/android/googleplay_shalf_1/6.png)

- 编辑版本，需要上传 `aab` 包，因为新应用不能再采用 `apk`+ `obb` 的方式上传了，根据 Google Play 上架的规则调整， 2021 年 8 月起，新建应用需要上传 `aab` 包。

![新建版本](/img/android/googleplay_shalf_1/7.png)

- 根据运营或者市场提供的内购表，配置商品，`产品 ID` 就是在查询商品信息的时候需要的 `sku`。

![配置谷歌内购](/img/android/googleplay_shalf_1/8.png)

- 至此基本的准备工作就已经完成了，如果有接入 `Firebase` 和 `Adjust` 等数据上报 SDK 需求的，请移步至 **SDK 接入** 部分。