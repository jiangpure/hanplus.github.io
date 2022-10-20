---
layout:     post
title:      "『Android』 Google Play 上架流程（二）：Google 登录"
subtitle:   "Google登录服务的接入工作。"
date:       2021-12-21 17:19:00
author:     "purejiang"
header-img: "img/post-bg-1.jpg"
catalog: true
tags:
    - Android
---


>上文说到了接入前的准备工作，接下来主要分为 **Google 登录服务**、 **Google Play 结算服务**、**数据统计服务**三部分进行接入，本文内容为 **Google 登录服务**的接入工作。


## Google 登录服务
本文主要内容是 Google 普通登录和 Google 一键登录的接入。

### Google 普通登录
- 在项目的顶级 `build.gradle` 文件中添加谷歌仓库。

```groovy
buildscript {
    repositories {
        google()
    }
    // ...
}
allprojects {
    repositories {
        google()
    }
    // ...
}
```

- 在应用级的 `build.gradle` 文件中添加依赖，本文编写时的版本为 `20.0.0`。

```groovy
implementation 'com.google.android.gms:play-services-auth:20.0.0'
```

- 创建 `GoogleSignInClient` 对象，设置 `server_client_id`，`server_client_id` 即在谷歌 API 后台获取的 `Web Client Id`。

```java
// 谷歌登录配置
GoogleSignInOptions gso = new GoogleSignInOptions.Builder(GoogleSignInOptions.DEFAULT_SIGN_IN)
        .requestIdToken(getString(R.string.server_client_id))
        .requestEmail()
        .build();
// 传入 Activity
mGoogleSignInClient = GoogleSignIn.getClient(this, gso);
```

- 获取当前登录账号，不为 `null` 时，可以**直接进行登录**或者**调用登出再调登录**。

```java
// 获取的 account 不为 null 则表示当前已有登录的账号，为 null 则无任何登录账号
GoogleSignInAccount account = GoogleSignIn.getLastSignedInAccount(this);
```

- 通过 `GoogleSignInClient.getSignInIntent()` 获取登录的 `Intent` 对象，通过 `startActivityForResult()` 拉起谷歌登录界面。

```java
// RC_SIGN_IN 为自定义的请求码
Intent signInIntent = mGoogleSignInClient.getSignInIntent();
startActivityForResult(signInIntent, RC_SIGN_IN);
```

- 登录完成后，在 `onActivityResult()` 中获取登录结果和账号信息，一般拿到 `id` 和 `token` 即可，`email` 是可变更的，所以不能作为用户的唯一标识。

```java
@Override
public void onActivityResult(int requestCode, int resultCode, Intent data) {
    super.onActivityResult(requestCode, resultCode, data);

    if (requestCode == RC_SIGN_IN) {
        Task<GoogleSignInAccount> task = GoogleSignIn.getSignedInAccountFromIntent(data);
        handleSignInResult(task);
    }
}

private void handleSignInResult(Task<GoogleSignInAccount> completedTask) {
    try {
        // 登录成功，获取账号信息
        GoogleSignInAccount account = completedTask.getResult(ApiException.class);
        String id = account.getId();
        String token = account.getIdToken();
        String email = account.getEmail();
        String name = account.getDisplayName();
        Toast.makeText(GoogleLoginActivity.this, "登陆成功", Toast.LENGTH_SHORT).show();
    } catch (ApiException e) {
      // 登录失败
      Log.e(TAG, "signInResult:failed code=" + e.getStatusCode()+",msg:"+e.getMessage());
      Toast.makeText(GoogleLoginActivity.this, "登陆失败", Toast.LENGTH_SHORT).show();
    }
}
```

- 通过 `GoogleSignInClient.signOut()` 可以登出谷歌账号，切换账号可以以 *登出 -> 登录* 方式实现。

```java
mGoogleSignInClient.signOut().addOnCompleteListener(new OnCompleteListener<Void>() {
    @Override
    public void onComplete(@NonNull Task<Void> task) {
        // 登出账号成功
    }
});
```
- 以下便是谷歌普通登录的界面，选择测试账号登录即可。
![Google 普通登录](/img/android/googleplay_shalf_2_1/1.png)
### 谷歌一键登录
谷歌一键登录


### 注意事项

- 当登陆失败时，请检查 `Web Client Id` 与 [Google API 后台](https://console.cloud.google.com/) 配置的是否一致。
- 当登陆失败时，请检查是否能连接 **Google Play 商店**。
- 一般来说，在 Google API 后台创建凭证时，会创建**包名**和上传签名的 `SHA1`，所以当谷歌登录失败，错误码 `= 10` 时，请检查**包名**和 `SHA1` 值。

以上就是 Google 登录服务的接入过程，更详细的接入内容请参考官方文档[《使用谷歌登录》](https://developers.google.cn/identity/sign-in/android/start-integrating)。

