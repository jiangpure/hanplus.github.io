---
layout:     post
title:      "『Android』 Google Play 上架流程（二）：Firebase 和 Adjust 的接入"
subtitle:   "数据上报 SDK 的接入工作。"
date:       2021-12-25 10:00:00
author:     "purejiang"
header-img: "img/post-bg-1.jpg"
catalog: true
tags:
    - Android
---

>上文说到了 **Google 结算服务**的接入工作，接下来进行 `Adjust` 和 `Firebase` 的接入工作。

应用一般会自带数据上报或者接入第三方数据上报 SDK ，以得到有关应用和用户的行为数据，然后会根据数据针对应用的投放和功能等进行相应调整，所以这个还是挺重要的。

国内一般是像`友盟统计`之类的。

海外比较常用的就是 `Adjust` 和来自 Google 的 `Firebase`。

---
### 一、Firebase 的接入

#### 1. 新建项目

首先登录 [Firebase 控制台](https://console.firebase.google.com/u/0/)，开始新建项目。\
![创建项目](/img/android/googleplay_shalf_2_3/1.png)

输入项目名称，这里没有特殊限制。\
![输入项目名](/img/android/googleplay_shalf_2_3/2.png)

可以自行选择是否开启 Google Analytics（建议开启，后期看数据方便）。\
![选择开启](/img/android/googleplay_shalf_2_3/3.png)

新建或者选择已有的分析账号（可以每个分析账号对应一个应用或者几个同类型应用，看具体需要）。\
![选择账号](/img/android/googleplay_shalf_2_3/4.png)

到这里新建项目已经是完成，接下来就是关联应用。

#### 关联应用

首先打开 Firebase 控制台，转到项目设置，并在底部添加应用，`Android`/`Web`/`Ios`，这里我们是新建 Android 应用。\
![项目设置](/img/android/googleplay_shalf_2_3/5.png)

其次添加应用配置，如果有接入 Google 登录的话，包名和 `SHA1` 必须和 Google 登录的配置保持一致。\
![添加应用](/img/android/googleplay_shalf_2_3/6.png)

然后下载配置文件 `google-services.json`，这个文件，在后面的 Firebase SDK 接入中是必需的。\
![下载配置文件](/img/android/googleplay_shalf_2_3/7.png)

最后都选下一步即可，创建完成之后就可以在项目设置里面看到应用了。\
![项目设置里面的应用](/img/android/googleplay_shalf_2_3/8.png)

#### 接入 SDK 和事件上报

将下载的 `google-services.json` 文件放在应用级目录（也就是 app 目录）下，并做好插件相关配置。

- 在项目级的 `build.gradle` 的 `buildscript/dependencies` 下添加：

```groovy
buildscript {
    repositories {
        google()  // Google's Maven repository
    }
    dependencies {
        classpath 'com.google.gms:google-services:4.3.5'
    }
}
```
- 在应用级的 `build.gradle` 中添加插件和依赖：

```groovy
apply plugin: 'com.android.application'
// add this line
apply plugin: 'com.google.gms.google-services'

android {
    // ...
}

dependencies {
    // Firebase
    implementation platform('com.google.firebase:firebase-bom:26.8.0')
    implementation 'com.google.firebase:firebase-analytics'
}
```

事件上报分为 Firebase 自动收集事件和自定义事件：

- 自动收集事件是不需要我们主动去上报，Firebase 内部会自动进行采集上报的，如安装、卸载、首次打开等等。具体事件可参考： [[GA4] 自动收集的事件 - Google Analytics（分析）帮助](https://support.google.com/analytics/answer/9234069?hl=zh-Hans&ref_topic=9756175)。

- 运营或者市场等需要针对更细致的场景和行为数据进行收集，可以通过 `FirebaseAnalytics.logEvent()` 在指定位置进行自定义事件的上报。

```java
public class MyActivity extends Activity {
    private FirebaseAnalytics mFirebaseAnalytics;
   
    mFirebaseAnalytics = FirebaseAnalytics.getInstance(this);
}
```

Bundle 里面可以不带参数，也可以带参数上报。事件名和参数名可以使用自定义字符串，也可以使用 `FirebaseAnalytics.Event` 和 `FirebaseAnalytics.Param` 提供的固定字符。
```java
    Bundle bundle = new Bundle();
    mFirebaseAnalytics.logEvent("test", bundle);
    
    Bundle bundle = new Bundle();
    bundle.putString(FirebaseAnalytics.Param.ITEM_ID, id);
    mFirebaseAnalytics.logEvent(FirebaseAnalytics.Event.SELECT_CONTENT, bundle);
```

当然，除了我们自己需要的事件，Firebase 也推荐了一些事件，可视情况上报，建议的事件可参考： [[GA4] 推荐的事件 - Google Analytics（分析）帮助](https://support.google.com/analytics/answer/9267735?hl=zh-Hans&ref_topic=9756175)。

#### 上报测试
接入后不可能直接上线进行上报测试，而且数据的查看不是实时的，一般都是统计前一天的数据。所以，要确定事件是否全部接好，我们需要使用 Firebase 的 `DebugView` 功能。

![DebugView功能](/img/android/googleplay_shalf_2_3/9.png)

1. 首先需要 Android 手机连接 `USB 调试`，命令行执行如下：

    `adb shell setprop debug.firebase.analytics.app <包名>`
    
2. 执行命令之后，就可以实时的在 DebugView 界面看到事件上报了。
![事件上报](/img/android/googleplay_shalf_2_3/10.png)

3. 测试完毕之后，要关闭 `debug` 模式，命令行执行如下：
    
    ```adb shell setprop debug.firebase.analytics.app .none.```
	
测试过程中，如果出现没有收到事件上报的情况，可从以下方面进行排查：
- 项目配置参数是否正确
- 事件上报代码有无问题
- 手机网络连接是否正常
- 手机是否开启 Firebase 的 `debug` 模式
- 手机是否支持谷歌（例如华为）

### Adjust


Adjust的接入。
