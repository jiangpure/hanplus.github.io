---
layout:     post
title:      "『Android』 Google Play 上架流程（二）：Google Play 结算库 5.0"
subtitle:   "Google Play 结算系统的接入工作。"
date:       2022-9-20 14:22:11
author:     "purejiang"
header-img: "img/post-bg-1.jpg"
catalog: true
tags:
    - Android
---

>上文说到了 **Google 登录服务**的接入工作，接下来进行 **Google Play 结算库**的接入工作。

## Google Play 结算库
**Google Play 结算库** 主要是应用 / 游戏使用谷歌的支付系统，其销售的内容一般来说分为两种：
- *一次性物品* （一次性的非定期付费购买的商品，例如 xx 游戏中购买50点券）
- *订阅物品* （定期使用内容的商品，例如订阅一个月的 xx 大会员）

本文针对应用内一次性物品的购买，不涉及订阅物品，如有需要请参阅官方文档 [Google Play 结算库](https://developer.android.google.cn/google/play/billing)。

话不多说，接下来接入 Google Play Billing，开始~

### 一、接入前的准备

在结构较为复杂的项目或者多个项目中使用时，可以新建一个 **Module**，然后新建工具类用来实现所有的支付逻辑，包括谷歌支付的初始化，拉起支付，下单，补单等，最后通过提供接口供外部调用。

结算库的版本，可在官网查看，本文编写时的版本为 `5.0.0`，对比 `4.1.0` ，新版本在接入代码上有改动，所以新接入的话建议直接接 `5.0.0`。

### 二、接入 Google Play 结算服务 SDK

#### 2.1 导入远程依赖
在 **Module** 的 `build.gradle` 下导入远程依赖。
```groovy
dependencies {
    // billing的版本
    def billing_version = "5.0.0"
    implementation "com.android.billingclient:billing:$billing_version"
}
```
#### 2.2 初始化结算库
- 设置支付结果回调 `purchaseUpdateListener`，进行交易成功或者失败后的处理（详见 2.5 小节），`enablePendingPurchases()` 是支持待处理的交易（必须加上）。

```java
mBillingClient = BillingClient.newBuilder(context)
        .setListener(purchaseUpdateListener)
        .enablePendingPurchases()
        .build();          
```

- 然后通过 `BillingClient.startConnection()` 连接谷歌商店，在 `onBillingServiceDisconnected()` 里可以尝试重新连接或者做好失败的处理，连接成功后才可进行后面的操作，或者在调起支付时通过设定的连接状态参数去做重连处理。

```java
mBillingClient.startConnection(new BillingClientStateListener() {
    @Override
    public void onBillingSetupFinished(BillingResult billingResult) {
        if (billingResult.getResponseCode() == BillingClient.BillingResponseCode.OK) {
            // 连接成功
        }
    }

    @Override
    public void onBillingServiceDisconnected() {
        // 强烈建议实现自己的连接重试逻辑并替换 onBillingServiceDisconnected() 方法。请确保在执行任何方法时都与 BillingClient 保持连接。
        // 连接失败，最好重新调用谷歌的连接服务
    }
});
```
#### 2.3 查询商品信息
通过 `sku` 查询商品信息，`sku` 是在 **Google Play后台** 配置的**商品标识**。
\
生成谷歌的交易参数，~~`4.0.0` 使用的是 `SkuDetailsParams`~~，`5.0.0` 改成了 `QueryProductDetailsParams`。\
交易参数只能传 `sku` 的列表，所以单次购买一件商品，也要传 List。\
一次性购买的商品设置的 **type** 是 `SkuType.INAPP`，订阅物品则是 `SkuType.SUBS`。\
\
`SkuDetails` 是商品信息，一般情况我们原封不动拿到然后去下单即可。

```java
// 结算库 4.0
/*
List<String> skuList = new ArrayList<>();
skuList.add(sku);
SkuDetailsParams.Builder params = SkuDetailsParams.newBuilder();
params.setSkusList(skuList).setType(BillingClient.SkuType.INAPP);
*/
//结算库5.0
List<QueryProductDetailsParams.Product> productList = new ArrayList<>();
QueryProductDetailsParams.Product product = QueryProductDetailsParams.Product.newBuilder()
                .setProductId(sku)
                .setProductType(BillingClient.ProductType.INAPP)
                .build();
productList.add(product);
QueryProductDetailsParams queryProductDetailsParams = QueryProductDetailsParams.newBuilder()
                        .setProductList(productList)
                        .build();
// 4.0 查询商品列表
/*
mBillingClient.querySkuDetailsAsync(params.build(), new SkuDetailsResponseListener() {
            @Override
            public void onSkuDetailsResponse(BillingResult billingResult, List<SkuDetails> skuDetailsList) {

            }
});
*/
// 5.0 查询商品列表
mBillingClient.queryProductDetailsAsync(queryProductDetailsParamsnew ProductDetailsResponseListener() {
            @Override
            public void onProductDetailsResponse(BillingResulbillingResult, List<ProductDetails> productDetailsList) {
                // check billingResult
                int code = billingResult.getResponseCode();
                // 查询商品失败
                if (code != BillingClient.BillingResponseCode.OK || productDetailsList == null || productDetailsList.isEmpty()) {
                    String msg = billingResult.getDebugMessage();
                    if (mCallBack != null) {
                        mCallBack.payFailure(QUERY_SKU_FAIL, msg);
                    }
                    return;
                }
                // 查询商品成功
                for (ProductDetails productDetails : productDetailsList) {
                    // 拉起支付页面
                }
        }
    }
);
```
#### 2.4 拉起支付页面
通过 `launchBillingFlow()` 方法可以发起交易请求进行下单。\
\
先要新建一个下单请求的参数 `BillingFlowParams`，携带商品信息和额外信息;`ProductDetailsParams` 需要传入查询到的商品信息 `productDetails`。

- ~~`2.0` 版本~~ 和以前接入的 `AIDL` 方式，是提供了透传参数 `developerPayload` 的，方便开发者补单。
- 现在的 `3.0` 版本往后是没有这个参数，取而代之的是在下单请求参数里面的关联字符串。

建议 `setObfuscatedAccountId()` 传入应用的用户 `id`，`setObfuscatedProfileId` 传入应用本次下单的 `orderId`（不是谷歌的 `orderId`，是应用自己的 `orderId`），这样就可以关联到对应的应用 `orderId` 进行补单。

```java
List<BillingFlowParams.ProductDetailsParams> productDetailsParamsList = new ArrayList<>();
BillingFlowParams.ProductDetailsParams product = BillingFlowParams.ProductDetailsParams.newBuilder()
    .setProductDetails(productDetails)
    // 优惠购买
    //.setOfferToken(selectedOfferToken)
    .build();
productDetailsParamsList.add(product);

BillingFlowParams billingFlowParams = BillingFlowParams.newBuilder()
                .setProductDetailsParamsList(productDetailsParamsList)
                .setObfuscatedAccountId(orderId)
                .setObfuscatedProfileId(orderId)
                    .build();

        // 拉起支付页面
BillingResult billingResult = mBillingClient.launchBillingFlow(activity, billingFlowParams);

int responseCode = billingResult.getResponseCode();
if (responseCode == BillingClient.BillingResponseCode.ITEM_ALREADY_OWNED) {
            // 商品已存在
    return;
}
if (responseCode != BillingClient.BillingResponseCode.OK) {
    String msg = billingResult.getDebugMessage();
    // 拉起支付页面失败
    return;
}
```
#### 2.5 支付结果
自定义成功和失败的接口，用来供外部调用，当然有其他的回调也可以加进来，像支付的步骤回调（用于支付日志打点上报）等。

```java
public interface GooglePayCallback {
    /**
     * 支付成功
     * @param purchase 订单信息
     * @param sku 商品 id
     * @param purchaseToken 订单对应的token
     * @param orderId 订单 id
     * @param isSupplement 是否是补单
     */
    void paySuccess(Purchase purchase, String sku, String purchaseToken, String     orderId, boolean isSupplement);

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

`PurchasesUpdatedListener` 是谷歌支付结果的回调，在 `onPurchasesUpdated()` 中可以对支付结果 `BillingResult` 进行处理，`BillingResult` 中有两个方法：\
\
`getResponseCode()` 方法获取结算 API 返回的支付状态码，`BillingClient.BillingResponseCode.OK` 表示 **支付成功**，`BillingClient.BillingResponseCode.USER_CANCELED` 表示 **用户取消支付**，其他状态码表示 **支付出现错误**。\
\
`billingResult.getDebugMessage()` 方法可以获取结算 API 返回的相关消息，这个方法返回的字符串可能为空。

```java
purchaseUpdateListener = new PurchasesUpdatedListener() {
            @Override
            public void onPurchasesUpdated(@NonNull BillingResult billingResult, List<Purchase> purchases) {
                // 根据返回的状态码做相应回调
                if (billingResult.getResponseCode() == BillingClient.BillingResponseCode.OK && purchases != null) {

                    // 支付成功
                    for (Purchase purchase : purchases) {
                        // 通知发货
                        mCallBack.paySuccess(purchase, purchase.getSkus().get(0), purchase.getPurchaseToken(), orderId, false);
                    } 
                } else if (billingResult.getResponseCode() == BillingClient.BillingResponseCode.USER_CANCELED) {

                    // 支付取消
                    mCallBack.payFailure(PAY_CANCEL, billingResult.getDebugMessage());
                } else {
                
                    // 支付失败
                    mCallBack.payFailure(PAY_FAIL, billingResult.getDebugMessage());
                }
            }
        }
```
#### 2.5 消耗订单（通知谷歌发货成功）
用户支付成功后会在 `PurchasesUpdatedListener` 的 `onPurchasesUpdated()` 中回调结果，所以在`BillingClient.BillingResponseCode.OK` 时进行**发货**。

发货成功之后调用 `BillingClient.consumeAsync()` 对下单的商品进行消耗（确认发货），如果客户端收到发货成功的通知不准确，也可以在服务端侧进行消耗。

**无论是客户端还是服务端，每次发货成功都一定要调用消耗，确保用户能对该商品进行下一次的购买。**

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

#### 2.6 客户端补单
- 在整个支付过程中，出现以下几种情况需要重新查询没有消耗的订单，然后进行补单：\
\
A: 支付成功但是没有在 `onPurchasesUpdated()` 回调中收到支付成功。\
B: 收到支付成功但是没有进行发货（一般是客户端通知服务端发货）。\
C: 发货成功但是没有进行商品的消耗。\
D: ~~在应用外进行的购买~~（本文是针对应用内支付，所以考虑该情况）。\
\
查询订单可通过 `BillingClient.queryPurchasesAsync()` 方法进行，一般在 `onCreate()` 和 `onResume()` 中调用，也可以在应用的初始化或者购买商品处调用。\
\
在查询消费失败的订单回调中，可以从 `purchase` 的附带信息中拿到下单时存的应用 `orderId` 和 `userId`，进行补单操作。

```java
    /**
    * 查询订单，google最近一次的交易（成功失败都会返回）
    */
mBillingClient.queryPurchasesAsync(QueryPurchasesParams.newBuilder().setProductType(BillingClient.ProductType.INAPP).build(), new PurchasesResponseListener() {
    @Override
    public void onQueryPurchasesResponse(BillingResult billingResult, List<Purchase> purchases) {
        // Process the result
        for (Purchase purchase : purchases) {
            handleConsumePurchase(purchase);
        }
    }
});
```
### 三、支付验证
支付完成之后，需要在服务端进行谷歌支付订单的验证。

#### 3.1 获取验证配置

- 在 Google Cloud Platform 中选择应用对应的项目
- 搜索进入“API 和服务”
- 点击“启用 API 和服务”
- 选择并启用这些 API ：\
  Google play android developer API\
  Google Play Developer Reporting API\
  Google Play Custom APP Publishing API
- 创建一个服务账号，并授予 Owner 角色
- 下载对应的 JSON 文件

#### 3.2 获取验证的 Token
- 然后由服务端同事获取用于支付验证的 refresh_token。

### 3.3 进行支付验证
可以通过 [API products](https://developers.google.cn/android-publisher/api-ref/rest/v3/purchases.products) 拿到订单信息，根据购买状态和消耗状态和透传参数中的应用 `orderId` 判断是否发货。

- `purchaseState` （购买状态）：0. 已购买 1. 已取消 2. 待处理
- `consumptionState` （消耗状态）：0. 尚未消耗 1. 已使用
- `obfuscatedExternalProfileId` （可传字符串）：应用的 orderId 或者订单的唯一标识

### 四、避坑指南
- 当通过 `sku` 查询商品信息，返回错误码 `-1`，错误信息 `Service connection is disconnected` 时，可检查当前登录的测试账号，尽量不要是开发者账号；同时也可以进入 Google Play，检查是否有显示付费项目，如果有付费项目一般就没有问题，没有的话需要更换网络地区或者账号。
- 谷歌掉单的情况下，最好是客户端通知服务端发货，支付时一定要记得传透传参数，或者本地储存 应用 orderid 和谷歌 orderid 的映射，这样在后面补单的时候才能对应上应用的订单号（否则会造成丢单），发货成功后服务端再对谷歌订单进行消耗。
- 谷歌结算服务的测试步骤：
  1. 上传 `aab` 包体到 Google Play
  2. 将本地账号加入应用的测试账号列表
  3. 从测试链接中下载商店包进行支付的测试\
\
![测试账号列表](/img/android/googleplay_shalf_2_2/0.png)

- 支付验证出现 `401` 的情况：\
  \
![验证出错](/img/android/googleplay_shalf_2_2/1.png)
1. 检查权限是否都有。\
   \
![权限](/img/android/googleplay_shalf_2_2/2.png)

2. 将新建的 *server_account* 的 *email* 加入到测试组，然后重新保存应用内的商品信息：\
\
![加入到测试组](/img/android/googleplay_shalf_2_2/3.png)
\
![重新保存商品](/img/android/googleplay_shalf_2_2/4.png)