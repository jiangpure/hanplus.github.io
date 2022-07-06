---
layout:     post
title:      "『Android』 将 GitHub 项目发布为远程依赖"
subtitle:   "可以依赖自己的远程库了。"
date:       2018-09-10 12:00:00
author:     "purejiang"
header-img: "img/post-bg-1.jpg"
catalog: true
tags:
    - Android
---

由于新的小项目想依赖自己以前写的库，老是本地复制粘贴感觉很麻烦，然后学习了一下发布远程依赖，在此记录一下，也提供一些经验避免踩坑吧~
***
## 一、发布 GitHub 项目
*如果已经了解发布 GitHub 项目请直接跳过这一步。*


这里说一下 Android Studio 上传项目到 GitHub：
### 1. 下载并安装 [Git](https://git-scm.com/) 。

### 2. 在 Android Studio 上配置 Git：

*File* -> *Settings* -> *Version Control* -> *Git*。

![配置Git](/img/android/remote_implement/0.jpg)

### 3. 配置 GitHub 账号：

*File* -> *Settings* -> *Version Control* -> *GitHub*。

![配置GitHub账](/img/android/remote_implement/1.jpg)

### 4. 选定项目，创建本地代码仓库：

*VCS* -> *Import into Version Control* -> *Create Git Repository...*

![创建本地代码仓库](/img/android/remote_implement/2.jpg)

### 5. 项目上右键，添加文件到本地仓库，如需添加单个文件可在文件上右键然后 Add 即可：

![添加文件到本地仓库](/img/android/remote_implement/3.jpg)

### 6. 创建 GitHub 远程仓库：

![上传本地仓库到GitHub](/img/android/remote_implement/4.jpg)

- 输入仓库名和是否私有以及仓库描述等：

![输入仓库名](/img/android/remote_implement/5.jpg)

### 7. 提交文件到本地仓库并同步到 GitHub：

![提交文件或者文件夹](/img/android/remote_implement/6.jpg)

- 选择需要提交同步的文件并输入提交信息：

![选择提交文件](/img/android/remote_implement/7.jpg)

-  *Commit* 只会上传到本地仓库选择，*Commit and Push* 会在提交到本地仓库的同时同步到 GitHub，然后就可以在 GitHub 上看到项目了。

![Commit/Commit and Push](/img/android/remote_implement/8.jpg)

关于命令行上传，可以百度或者看这里：[命令行上传本地项目到GitHub)](https://purejiang.gitee.io/2019/02/12/github_push_cmd/)。

***

## 二、发布 GitHub 项目的版本
-  当项目上传完成后，需要在 GitHub 上发布版本：

![查看版本](/img/android/remote_implement/9.jpg)

![创建版本](/img/android/remote_implement/10.jpg)

- 发布版本，并填写相关信息即可。

![发布版本](/img/android/remote_implement/11.jpg)

- 发布后可查看改项目所有的发布版本。

![查看版本](/img/android/remote_implement/12.jpg)

***

## 三、发布 GitHub 版本到 JitPack
[JitPack](https://jitpack.io/) 是一个远程仓库，将项目版本同步到 JitPack，之后无需审核即可远程依赖。

- 进入 JitPack，使用 GitHub 账号登录。

![GitHub账号登录](/img/android/remote_implement/13.jpg)

- 登陆之后可以看到已有的 GitHub 项目，右边是已有的版本，点击 **Get it** 则 JitPack 将会开始编译项目。

![开始编译项目](/img/android/remote_implement/14.jpg)

- 是否编译成功可通过 Log 的颜色判断，红色则为失败，绿色为通过，当 Log 为红色的时候，通过远程依赖是找不到的，可以通过点击 Log 图标进行查看编译日志，排查失败的原因。

![是否编译成功](/img/android/remote_implement/15.jpg)

- 编译成功之后，在 Android Studio 项目的根 `build.gradle` 中添加 maven 路径，然后在添加依赖即可。

![添加依赖](/img/android/remote_implement/16.jpg)


* 如果 Gradle 工具版本大于等于4.6。

1) 在根 build.gradle 添加：
```groovy
buildscript { 
  dependencies {
    classpath 'com.github.dcendents:android-maven-gradle-plugin:2.1' // Add this line
  }
}
```

2) 在 library 的 build.gradle 中添加，`${YourUsername}` 是远程依赖的项目名：
```groovy
apply plugin: 'com.github.dcendents.android-maven'  

group = 'com.github.${YourUsername}'
```

3) 重新提交 GitHub 并发行版本，且同步到 JitPack 即可。
