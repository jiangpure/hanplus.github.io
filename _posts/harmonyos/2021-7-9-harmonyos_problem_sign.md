---
layout:     post
title:      "『HarmonyOS』 HAP 的签名调试问题"
subtitle:   "鸿蒙打包签名问题。"
date:       2021-07-09 13:14:00
author:     "purejiang"
header-img: "img/post-bg-1.jpg"
catalog: true
tags:
    - HarmonyOS
---

> 最近刚开始学习鸿蒙开发，发现和安卓开发还是有点不同的...

---
建完项目之后想要在真机上运行，发现提示：
![没签名不能运行](/img/harmonyos/harmonyos_problem_sign/1.png)
随即点链接进入到华为的文档：[编译构建生成 HAP](https://developer.harmonyos.com/cn/docs/documentation/doc-guides/build_hap-0000001053342418#section7901222142216)

按照步骤到自动签名步骤发现，和教程上一样的操作却生成不了签名，提示：
![自动签名界面提示](/img/harmonyos/harmonyos_problem_sign/2.png)

看提示是说没有找到 HarmonyOS 设备，但是我的真机是 HarmonyOS 设备，而且已经开启 `USB 调试`连上 ADB 了，这让我百思不得其解。看它文档上又是 *[申请发布证书和 Profile 文件](https://developer.harmonyos.com/cn/docs/documentation/doc-guides/publish_app-0000001053223745#section178461193713)* 这些东西，我就只想运行下 Demo 而已...

经过我的一番查找，还是找到了原因
- 如果是使用自动签名的步骤，在 ADB 调试时必须只能有 HarmonyOS 设备，如果有其他的设备也在连接，则会出现上面读不到 HarmonyOS 设备的情况。

最后附上开发工具的指南：[HUAWEI DevEco Studio 使用指南](https://developer.harmonyos.com/cn/docs/documentation/doc-guides/tools_overview-0000001053582387)




