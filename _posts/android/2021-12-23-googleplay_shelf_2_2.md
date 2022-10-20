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

>上文说到了 **Google 登录服务**的接入工作，接下来进行 **Google Play 结算服务**的接入工作，本文主要针对应用内一次性物品的购买，不涉及订阅物品，如有需要请参阅官方文档 [Google Play 结算系统](https://developer.android.google.cn/google/play/billing)。

## Google Play 结算服务
Google Play 结算服务主要是应用 / 游戏使用谷歌的支付系统，其销售的内容一般来说分为两种：
- *一次性物品* （一次性的非定期付费购买的商品，例如 xx 游戏中购买50点券，或者一把武器）
- *订阅物品* （定期使用内容的商品，例如订阅一个月的 xx 大会员）

话不多说，接下来接入 Google Play Billing，开始~

### 一、接入前的准备

在结构较为复杂的项目或者多个项目中使用时，可以新建一个 **Library**，然后新建工具类用来实现所有的支付逻辑，包括谷歌支付的初始化，拉起支付，下单，补单等，最后通过提供接口供外部调用。

在项目的 build.gradle 中导入结算服务的远程依赖，谷歌结算服务的版本，可在官网查看，本文编写时的版本为 `4.0.0`。
```groovy
dependencies {
    // billing的版本
    def billing_version = "4.0.0"
    implementation "com.android.billingclient:billing:$billing_version"
}
```
### 二、接入 Google Play 结算服务 SDK

1. 新建支付方法类\
新建 `GooglePayManager` 作为支付管理工具类，`BillingClient` 包含了很多支付相关方法，所以初始化 `BillingClient` 实例放前面作为全局变量，`setListener()` 设置支付结果的监听（详见第三步）。

```java
mBillingClient = BillingClient.newBuilder(context)
        .setListener(mPurchaseUpdateListener)
        .enablePendingPurchases()
        .build();          
```
2. 定义结算服务相关回调的接口\
定义的接口用来供外部调用，当然有其他的回调也可以加进来，如连接谷歌商店成功/失败、补单成功/失败等。

```java
public interface GooglePayCallback {
    /**
    * 支付成功
     * @param purchase
     * @param isSupplement 是否是补单
     */
    void paySuccess(Purchase purchase, boolean isSupplement);

    /**
     * 支付失败
     *
     * @param code
     * @param msg
     */
    void payFailure(int code, String msg);

    /**
     * 支付步骤
     *
     * @param orderId orderId,有则传,无则传""
     * @param type 类型
     * @param msg 描述信息
     */
    void payStep(String orderId, String type, String msg);
}
```
3. 创建支付结果的监听\
`PurchasesUpdatedListener` 是支付结果的监听类，在初始化 `BillingClient` 的时候传入。\
\
`BillingResult` 是支付结果，`BillingResult` 中有两个方法：

- `getResponseCode()` 可以获取结算 API 返回的支付状态码，`BillingClient.BillingResponseCode.OK` 表示**支付成功**，`BillingClient.BillingResponseCode.USER_CANCELED` 表示**用户取消支付**，其他状态码表示**支付出现错误**。

- `getDebugMessage()` 可以获取结算 API 返回的相关消息。\
\
有了状态码和支付信息就可以进行支付成功、用户取消支付、支付失败和Google Play 服务断线等情况进行处理。
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

4. 连接 Google Play\
通过 `BillingClient.startConnection()` 连接 Google Play，应用和 Google Play 的连接可能会断开，导致查询商品信息失败、支付失败等问题，所以在 `onBillingServiceDisconnected()` 里一定要做好重新连接的处理，连接成功后才可进行后面的操作。

```java
mBillingClient.startConnection(new BillingClientStateListener() {
    @Override
    public void onBillingSetupFinished(BillingResult billingResult) {
        if (billingResult.getResponseCode() == BillingClient.BillingResponseCode.OK) {
            isConnected = true;
            // 这里也可以进行补单
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
5. 查询商品信息\
通过 `querySkuDetailsAsync()` 方法传递 `SkuDetailsParams` 来查询商品信息, `SkuDetailsParams` 只能传 `sku` （在 Google Play 后台配置的 `产品id`）的列表，所以就算只购买一件物品，也要传 List。\
\
由于我们用的是一次性物品，所以设置的 type 是 `SkuType.INAPP`，订阅物品则是 `SkuType.SUBS`。\
\
在 `onSkuDetailsResponse()` 回调中返回的 `SkuDetails` 是商品信息，一般情况我们拿到原封不动然后去进行下单即可。

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
                    // 查询商品失败
                    if (mCallBack != null) {
                        mCallBack.payFailure(QUERY_SKU_FAIL, billingResult.getDebugMessage());
                    }
                }
            }
});
```

6. 下单\
首先要新建一个下单请求的参数 `BillingFlowParams`，携带商品信息和额外信息。\
\
通过 `BillingClient.launchBillingFlow()` 发起购买请求进行下单，此时会拉起Google Play 的支付页面。\
\
以前接入的 `AIDL` 方式，是提供了透传参数 `developerPayload` 的，方便开发者补单；现在的 `Google Billing` 是没有这个参数，取而代之的是在下单请求参数里面的关联字符串。\
\
建议 `setObfuscatedAccountId()` 传入应用的用户 `id`，`setObfuscatedProfileId` 传入应用本次下单的 `orderId`（不是`谷歌 orderId`，是应用自己的 `orderId`），这样在补单的时候就可以关联到对应的 `应用 orderId`。

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

7. 发货\
购买成功后，谷歌结算服务会在 `PurchasesUpdatedListener` 的 `onPurchasesUpdated()` 中回调成功，所以在第二步定义好的成功接口中进行应用/游戏的**发货**。\
\
当发货成功之后，需要调用 `BillingClient.consumeAsync()` 对下单的商品进行消耗（确认发货），如果客户端收到发货成功的通知不准确，也可以在服务端侧进行消耗。\
\
**无论是客户端还是服务端，每次发货成功后都一定要调用消耗，不然用户下一次就会无法购买该商品。**\
\
在客户端侧，我们将消耗逻辑封装在 `handleConsumePurchase()` 方法里，这样可以在多个地方（首次购买成功/补单成功）调用。

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
8. 查询订单\
在整个支付过程中会出现以下几种情况，需要重新查询没有消耗的订单，这时需要查单然后补单：\
\
A: 支付成功但是没有在 `onPurchasesUpdated()` 回调中收到支付成功。\
B: 收到支付成功但是没有进行发货（一般是客户端通知服务端发货）。\
C: 发货成功但是没有进行商品的消耗。\
D: ~~在应用外进行的购买~~（本文是针对应用内支付，所以可以不考虑该情况）。\
\
可以通过 `BillingClient.queryPurchasesAsync()` 方法查询订单，一般在 `onCreate()` 和 `onResume()` 中调用，也可以在应用的初始化或者购买商品处调用。\
\
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
### 三、支付验证
支付完成之后，需要在服务端进行谷歌支付订单的验证。

- 先获取验证配置\
1. 在 Google Cloud Platform 中选择应用对应的项目。\
2. 搜索进入“API 和服务”。\
3. 点击“启用 API 和服务”。\
4. 选择并启用这些 API 库：Google play android developer API, Google Play Developer Reporting API, Google Play Custom APP Publishing API\
5. 创建一个服务账号，并授予 Owner 角色。\
6. 下载对应的 JSON 文件。

- 然后由服务端同事获取用于支付验证的 token。

### 避坑指南
- 谷歌结算服务的测试步骤：
1. 上传 `aab` 包体到 Google Play。
2. 将本地账号加入应用的测试账号列表。
3. 从测试链接中下载商店包进行支付的测试。
![测试账号列表](/img/android/googleplay_shalf_2_2/0.png)

- 当通过 `sku` 查询商品信息，返回错误码 `= -1`，错误信息 `：Service connection is disconnected` 时，可检查当前登录的测试账号，尽量不要是开发者账号。同时也可以进入 Google Play，检查是否有显示付费项目，如果有付费项目一般就没有问题。
- 谷歌掉单的情况下，最好是客户端通知服务端发货，发货成功后服务端再对谷歌订单进行消耗。

- 支付验证出现 `401` 的情况：
![验证出错](/img/android/googleplay_shalf_2_2/1.png)
1. 检查权限是否都有。
![权限](/img/android/googleplay_shalf_2_2/2.png)
2. 将新建的 *server_account* 的 *email* 加入到测试组，然后重新保存应用内的商品信息：\
![加入到测试组](/img/android/googleplay_shalf_2_2/3.png)
\
![重新保存商品](/img/android/googleplay_shalf_2_2/4.png)