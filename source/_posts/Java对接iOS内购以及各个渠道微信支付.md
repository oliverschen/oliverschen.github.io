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

1. app 购买成功之后，将 receipt-data 提交给自己的应用服务器
2. 应用服务器拿到数据之后解析后拿着数据去苹果服务器验证。
3. 苹果返回验证结果给应用服务器，应用服务器返回给 app。

####### 问题

1. 应用服务器到苹果服务器验证数时会有两个环境，沙盒测试环境和正式环境，这里在保存数据的时候要注意标记支付环境，方便后续统计数据。
2. 苹果服务器访问是真慢。
3. 这个问题是代码上线一段时间后在生产暴露出来的，被刷单。

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
        // 用于记录是沙盒测试支付还是实际用户购买支付，可用于以后对账
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

###### 被恶意攻击
1. 起因

因为版本时间比较紧张，各个开发和测试的时间都很短，导致在开发和测试环境没有暴露出来被刷单的问题。今天数据分析师突然说有异常充值订单，（和钱有关的还是要进行代码审核，多方确认才能上生产的）进行排查后发现是因为有人用支付成功的 receipt-data 来进行多次请求（应该是写了脚本进行连续访问）因为当时代码没有加入苹果支付返回的唯一ID transaction_id（这里确实是代码问题，正常情况是必须加上这些参数的）。导致重复充值问题。当时已经下班了才发现的（肯定回不去了），立马进行了修复，异常订单问题还没有讨论。

2. 解决

```
1. 首先给充值接口添加 redis 分布式锁，防止脚本恶意频繁刷单。接口的加密校验也做了处理。
2. 保存 Apple 验证返回的 transaction_id，校验获取到的这个值是唯一的，和 product_id 在同一层数据结构中。
3. 对异常用户做一些处理，纠正被刷的订单。
```

> 这个解决也都相对简单，就是在创建订单前先用 transaction_id 检查以下有没有订单，如果有的话就直接抛出相应的异常告知调用方。
> 如果没有的话，则创建订单，保存对应的 transaction_id，进行数据校验。

##### 微信 app 支付

> 推荐参考 https://github.com/binarywang

微信支付我们在生活中使用也比较多，流程也相对比较清晰，并且官方提供的文档都是中文的，所以比较容易一点。微信支付一般涉及到 APP 拉起支付，h5 支付，JSAPI 支付，Native 支付这几种，具体描述![官网](https://pay.weixin.qq.com/wiki/doc/api/index.html)看起来比较清晰，也更加详细，这里先看下 ![APP 支付](https://pay.weixin.qq.com/wiki/doc/api/app/app.php?chapter=8_1)，在自己 APP 中集成微信支付。

###### 流程

1. 选择要购买的商品，调用 APP 自己的服务生成订单和支付数据，加签后返回给客户端。
2. 客户端校验服务端返回的参数，拉起微信支付。
3. 输入密码进行支付，生成支付信息。

###### 问题

1. 前后端验签的时候特别容易出错，这里要仔细看文档。

###### 相关代码

``` java
 /**
     * 统一预下单接口
     *
     * @param jsonParam
     * @param request
     * @return
     * @throws WxPayException
     */
    @RequestMapping("/unifiedOrder")
    @ResponseBody
    public ResponseMsg unifiedOrder(@RequestBody JSONObject jsonParam,
                                    HttpServletRequest request) throws WxPayException {
        ResponseMsg responseMsg = new ResponseMsg(Code.SUCCESSED, Constants.SUCCESS);
        // 1. 根据自己业务校验参数
        // 2. 创建订单
        WxPayUnifiedOrderRequest orderRequest = new WxPayUnifiedOrderRequest();
        //商品描述
        orderRequest.setBody("some desc");
        //商户订单号
        orderRequest.setOutTradeNo(order.getOrderId());
        //订单总金额，单位为分
        orderRequest.setTotalFee(Item.getPrice());
        //终端IP
        orderRequest.setSpbillCreateIp(request.getRemoteAddr());
        //指定支付方式 no_credit--可限制用户不能使用信用卡支付
        orderRequest.setLimitPay("no_credit");
        //交易类型
        orderRequest.setTradeType("APP");
        //用户的 openid
        orderRequest.setOpenid(user.getOpenidApp());
       
        WxPayUnifiedOrderResult result = this.wxAppPayService.unifiedOrder(orderRequest);
        HashMap<String, String> param = new HashMap<>(8);
        param.put("appid", result.getAppid());
        param.put("partnerid", result.getMchId());
        param.put("prepayid", result.getPrepayId());
        param.put("package", "Sign=WXPay");
        param.put("noncestr", result.getNonceStr());
        param.put("timestamp", String.valueOf(System.currentTimeMillis() / 1000));
        final WxPayAppOrderResult pay = WxPayAppOrderResult.builder()
                .sign(SignUtils.createSign(param, null, properties.getMchKey(), new String[]{}))
                .prepayId(result.getPrepayId())
                .partnerId(result.getMchId())
                .appId(result.getAppid())
                .timeStamp(param.get("timestamp"))
                .nonceStr(result.getNonceStr())
                .packageValue("Sign=WXPay")
                .rechargeOrderId(order.getRechargeOrderId())
                .build();
        responseMsg.setData(pay);
        return responseMsg;
    }

```
加签方法：在 https://github.com/binarywang 加到本地后可以直接使用
```java

/**
   * 微信支付签名算法(详见:https://pay.weixin.qq.com/wiki/doc/api/tools/cash_coupon.php?chapter=4_3).
   *
   * @param params        参数信息
   * @param signType      签名类型，如果为空，则默认为MD5
   * @param signKey       签名Key
   * @param ignoredParams 签名时需要忽略的特殊参数
   * @return 签名字符串 string
   */
  public static String createSign(Map<String, String> params, String signType, String signKey, String[] ignoredParams) {
    SortedMap<String, String> sortedMap = new TreeMap<>(params);

    StringBuilder toSign = new StringBuilder();
    for (String key : sortedMap.keySet()) {
      String value = params.get(key);
      boolean shouldSign = false;
      if (StringUtils.isNotEmpty(value) && !ArrayUtils.contains(ignoredParams, key)
        && !Lists.newArrayList("sign", "key", "xmlString", "xmlDoc", "couponList").contains(key)) {
        shouldSign = true;
      }

      if (shouldSign) {
        toSign.append(key).append("=").append(value).append("&");
      }
    }

    toSign.append("key=").append(signKey);
    if (WxPayConstants.SignType.HMAC_SHA256.equals(signType)) {
      return me.chanjar.weixin.common.util.SignUtils.createHmacSha256Sign(toSign.toString(), signKey);
    } else {
      return DigestUtils.md5Hex(toSign.toString()).toUpperCase();
    }
  }

```

加签返回给 APP 后进行验证拉起支付，这里得特别小心加签前后的参数大小写，很容易出现问题。以上就是微信 APP 支付。

