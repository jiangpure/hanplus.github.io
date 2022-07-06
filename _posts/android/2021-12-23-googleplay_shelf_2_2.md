---
layout:     post
title:      "『Android』 Google Play 上架流程（二）：Google Play 结算服务"
subtitle:   "Google Play 结算系统的接入工作。"
date:       2021-12-23 14:22:11
author:     "purejiang"
header-img: "img/post-bg-1.jpg"
catalog: true
tags:
    - Android
---

>上文说到了 **Google 登录服务**的接入工作，接下来进行 **Google Play 结算服务**的接入工作。

## Google Play 结算服务
Google Play 结算服务主要是应用 / 游戏使用谷歌的支付系统，其销售的内容一般来说分为两种：
- *一次性物品* （一次性的非定期付费购买的商品，例如 xx 游戏中购买50点券）
- *订阅物品* （定期使用内容的商品，例如订阅一个月的 xx 大会员）

本文针对应用内一次性物品的购买，不涉及订阅物品，如有需要请参阅官方文档 [Google Play 结算系统](https://developer.android.google.cn/google/play/billing)。

话不多说，接下来接入 Google Play Billing，开始~

### 一、接入前的准备

在结构较为复杂的项目或者多个项目中使用时，可以新建一个 **Library**，然后新建工具类用来实现所有的支付逻辑，包括谷歌支付的初始化，拉起支付，下单，补单等，最后通过提供接口供外部调用。

谷歌结算服务的版本，可在官网查看，本文编写时的版本为 `4.0.0`。
```groovy
dependencies {
    // billing的版本
    def billing_version = "4.0.0"
    implementation "com.android.billingclient:billing:$billing_version"
}
```
### 二、接入 Google Play 结算服务 SDK

- 新建 `GooglePayManager` 类，创建 `BillingClient` 实例，因为 `BillingClient` 包含了很多支付相关方法，所以放前面作为全局变量。

```java
mBillingClient = BillingClient.newBuilder(context)
        .setListener(mPurchaseUpdateListener)
        .enablePendingPurchases()
        .build();          
```

- 定义如下接口，用来供外部调用，当然有其他的回调也可以加进来，像连接谷歌商店失败等。

```java
public interface GooglePayCallback {
	/**
	 * 支付成功
	 */
    void paySuccess(Purchase purchase);
    
    /**
	 * 支付失败
	 */
    void payFailure(int code, String msg);
    
    /**
     * 补单发货
     */
    void wendPurchase(Purchase purchase);
}
```

- `PurchasesUpdatedListener()` 是支付信息的监听，在这里可以对支付结果 `BillingResult` 进行处理，`BillingResult` 中有两个方法：

1. `getResponseCode()` 方法获取结算 API 返回的支付状态码，`BillingClient.BillingResponseCode.OK` 表示**支付成功**，`BillingClient.BillingResponseCode.USER_CANCELED` 表示**用户取消支付**，其他状态码表示**支付出现错误**。

2. `billingResult.getDebugMessage()` 方法可以获取结算 API 返回的相关消息。

```java
mPurchaseUpdateListener = new PurchasesUpdatedListener() {
            @Override
            public void onPurchasesUpdated(@NonNull BillingResult billingResult, List<Purchase> purchases) {
                // 根据返回的状态码做相应回调
                if (billingResult.getResponseCode() == BillingClient.BillingResponseCode.OK && purchases != null) {
                    // 成功
                    for (Purchase purchase : purchases) {
                        if (mCallBack != null) {
                            mCallBack.success(purchase);
                        }
                    } 
                } else if (billingResult.getResponseCode() == BillingClient.BillingResponseCode.USER_CANCELED) {
                    // 取消
                    if (mCallBack != null) {
                        mCallBack.payFailure(PAY_CANCEL, billingResult.getDebugMessage());
                    }
                } else {
                    // 失败
                     if (mCallBack != null) {
                        mCallBack.payFailure(PAY_FAIL, billingResult.getDebugMessage());
                    }
                }
            }
        }
```

- 通过 `BillingClient.startConnection()` 连接谷歌商店，在 `onBillingServiceDisconnected()` 里可以尝试重新连接或者做好失败的处理，连接成功后才可进行后面的操作。

```java
mBillingClient.startConnection(new BillingClientStateListener() {
    @Override
    public void onBillingSetupFinished(BillingResult billingResult) {
        if (billingResult.getResponseCode() == BillingClient.BillingResponseCode.OK) {
            isConnected = true;
        }
    }

    @Override
    public void onBillingServiceDisconnected() {
        // 强烈建议实现自己的连接重试逻辑并替换 onBillingServiceDisconnected() 方法。请确保在执行任何方法时都与 BillingClient 保持连接。
        if (mCallBack != null) {
            mCallBack.payFailure(CONNECT_FAILED, "Unable to connect Google Play.");
        }
    }
});
```

- 通过 `sku` 查询商品信息，`sku` 是在 Google Play 后台配置的商品标识 `id`，`SkuDetailsParams` 只能传 `sku` 的列表，所以单次购买一件，也要传 List。

由于我们用的是一次性物品，所以设置的 type 是 `SkuType.INAPP`，订阅物品则是 `SkuType.SUBS`。

`SkuDetails` 是商品信息，一般情况我们原封不动拿到然后去下单即可。

```java
List<String> skuList = new ArrayList<>();
skuList.add(sku);
SkuDetailsParams.Builder params = SkuDetailsParams.newBuilder();
params.setSkusList(skuList).setType(BillingClient.SkuType.INAPP);
// 查询商品列表
mBillingClient.querySkuDetailsAsync(params.build(), new SkuDetailsResponseListener() {
            @Override
            public void onSkuDetailsResponse(BillingResult billingResult, List<SkuDetails> skuDetailsList) {
                mSkuDetailsList = skuDetailsList;
                if (billingResult.getResponseCode() == BillingClient.BillingResponseCode.OK) {
                    // 查询商品成功
                    for (SkuDetails skuDetails : skuDetailsList) {
                        // 发起下单请求
                    }
                } else {
                    if (mCallBack != null) {
                        mCallBack.payFailure(QUERY_SKU_FAIL, billingResult.getDebugMessage());
                    }
                }
            }
});
```

- 通过 `BillingClient.launchBillingFlow()` 发起购买请求进行下单。

先要新建一个下单请求的参数 `BillingFlowParams`，携带商品信息和额外信息。

以前接入的 `AIDL` 方式，是提供了透传参数 `developerPayload` 的，方便开发者补单；现在的 `Google Billing` 是没有这个参数，取而代之的是在下单请求参数里面的关联字符串。

建议 `setObfuscatedAccountId()` 传入应用的用户 `id`，`setObfuscatedProfileId` 传入应用本次下单的 `orderId`（不是谷歌的 `orderId`，是应用自己的 `orderId`），这样就可以关联到对应的应用 `orderId` 进行补单了。

```java
BillingFlowParams billingFlowParams = BillingFlowParams.newBuilder()
        .setSkuDetails(skuDetails)
        .setObfuscatedAccountId(userId)
        .setObfuscatedProfileId(orderId)
        .build();
        
BillingResult billingResult = mBillingClient.launchBillingFlow(activity, billingFlowParams);

if (billingResult.getResponseCode() != BillingClient.BillingResponseCode.OK) {
    // 下单失败
    mCallBack.payFailure(LAUNCH_BILLING_FAIL, billingResult.getDebugMessage());
}
```

- 下单成功后会在 `PurchasesUpdatedListener` 的 `onPurchasesUpdated()` 中回调成功，所以在此处进行**发货**。

发货成功之后调用 `BillingClient.consumeAsync()` 对下单的商品进行消耗（确认发货），如果客户端收到发货成功的通知不准确，也可以在服务端侧进行消耗。**无论是客户端还是服务端，每次发货成功都一定要调用消耗，确保用户能对该商品进行下一次的购买。**

在客户端这里，我们将消耗逻辑封装在 `handleConsumePurchase()` 方法里，供其他地方调用。

```java
/**
 * 消耗（确认发货）
 */
public void handleConsumePurchase(Purchase purchase) {
    ConsumeParams consumeParams = ConsumeParams.newBuilder()
            .setPurchaseToken(purchase.getPurchaseToken())
            .build()
    ConsumeResponseListener listener = new ConsumeResponseListener() {
        @Override
        public void onConsumeResponse(BillingResult billingResult, String purchaseToken) {
            if (billingResult.getResponseCode() == BillingClient.BillingResponseCode.OK) {
                // 消耗成功
            }
        }
    };
    mBillingClient.consumeAsync(consumeParams, listener);
}
```

- 在整个支付过程中，出现以下几种情况需要重新查询没有消耗的订单，然后进行补单：

1. 支付成功但是没有在 `onPurchasesUpdated()` 回调中收到支付成功。
2. 收到支付成功但是没有进行发货（一般是客户端通知服务端发货）。
3. 发货成功但是没有进行商品的消耗。
4. ~~在应用外进行的购买~~（本文是针对应用内支付，所以不考虑该情况）。

查询订单可通过 `BillingClient.queryPurchasesAsync()` 方法进行，一般在 `onCreate()` 和 `onResume()` 中调用，也可以在应用的初始化或者购买商品处调用。

在查询消费失败的订单回调中，可以从 `purchase` 的附带信息中拿到下单时存的应用 `orderId` 和 `userId`，进行补单操作。

```java
/**
 * 查询订单
 */
public void queryPurchases() {
    mBillingClient.queryPurchasesAsync(BillingClient.SkuType.INAPP, new PurchasesResponseListener() {
        @Override
        public void onQueryPurchasesResponse(@NonNull BillingResult billingResult, @NonNull List<Purchase> list) {
            // google billing 消耗失败的补单
            for (Purchase purchase : list) {
                if (mCallBack != null) {
                    // 补单
                    mCallBack.wendPurchase(purchase);
                }
            }
        }
    });
}
```

### 三、避坑指南
- 当通过 `sku`查询商品信息，返回错误码 `= -1`，错误信息 `：Service connection is disconnected` 时，可检查当前登录的测试账号，尽量不要是开发者账号。同时也可以进入 Google Play，检查是否有显示付费项目，如果有付费项目一般就没有问题。

- 谷歌掉单的情况下，最好是客户端通知服务端发货，发货成功后服务端再对谷歌订单进行消耗。

- 谷歌结算服务的测试步骤：
1. 上传 `aab` 包体到 Google Play
2. 将本地账号加入应用的测试账号列表
3. 从测试链接中下载商店包进行支付的测试

![测试账号列表](/img/android/googleplay_shalf_2_2/1.png)