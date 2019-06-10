---
title: Java对接iOS内购以及各个渠道微信支付
date: 2019-05-29 23:11:29
tags: pay
category: pay 
---

![](jvm-Java对接iOS内购以及各个渠道微信支付/pay.png)

最近因为公司业务需要，要对接支付这一块，我也是第一次对接第三方支付相关，因为是微信，iOS 缘故，对接前感觉可能会比较容易，毕竟这两家的东西用的人肯定很多，但是在具体的对接过程中还是碰到了很多的问题。

<!-- more -->

##### iOS 内购

iOS 对接相对来说比较容易点，客户端 SDK 已经集成了支付相关的很多流程，这里服务端只是做一个校验，保存相关数据和业务操作就可以。

###### 购买流程：
```
1. app 购买成功之后，将 receipt-data 提交给自己的应用服务器
2. 应用服务器拿到数据之后解析后拿着数据去苹果服务器验证。
3. 苹果返回验证结果给应用服务器，应用服务器返回给 app。
```
####### 问题
```
1. 应用服务器到苹果服务器验证数时会有两个环境，沙盒测试环境和正式环境，这里在保存数据的时候要注意标记支付环境，方便后续统计数据。
2. 苹果服务器访问是真慢。

```
###### 相关代码

> 
1. 内购沙盒地址：SANDBOX_CERTIFICATE_URL = https://sandbox.itunes.apple.com/verifyReceipt
2. 正式地址：    BUY_CERTIFICATE_URL = https://buy.itunes.apple.com/verifyReceipt

``` java
/**
 * 
 * @param receiptData app 端传来的交易数据
 * @param userId 用户ID
 * @return 
*/
public Order iosInnerBuy(String receiptData, String userId) {
        String payChannel = "sandbox";
        String result = OkHttpUtil.sendHttpPost(SANDBOX_CERTIFICATE_URL, "{\"receipt-data\":\"" + receiptData + "\"}");
        JSONObject obj = JSONObject.parseObject(result);
        // 如果沙箱环境没有成功，调用正式环境处理
        if (obj.getInteger("status") != 0) {
            payChannel = "formal";
            result = OkHttpUtil.sendHttpPost(BUY_CERTIFICATE_URL, "{\"receipt-data\":\"" + receiptData + "\"}");
        }
        String productId = "";
        JSONObject jsonArry = JSONObject.parseObject(result);
        if (jsonArry == null) {
            throw new ParamsException(Code.FAILED, "参数有误");
        }
        JSONObject map = (JSONObject) jsonArry.get("receipt");
        if (map == null) {
            throw new ParamsException(Code.FAILED, "参数有误");
        }
        JSONArray inApp = (JSONArray) map.get("in_app");
        if (inApp == null) {
            throw new ParamsException(Code.FAILED, "参数有误");
        }
        if (inApp.size() > 0) {
            for (int i = 0; i < inApp.size(); i++) {
                JSONObject json = inApp.getJSONObject(i);
                productId = (String) json.get("product_id");
            }
        }
        if (StringUtils.isBlank(productId)) {
            log.info("product_id 为空，返回结果为:{}", result);
            throw new ParamsException(Code.FAILED, "product_id 不能为空");
        }
        // TODO 具体的业务逻辑...
        return order;
    }

```

###### 微信 app 支付

微信支付我们在生活中使用也比较多，流程也相对比较清晰，并且官方提供的文档都是中文的，所以比较容易一点。
