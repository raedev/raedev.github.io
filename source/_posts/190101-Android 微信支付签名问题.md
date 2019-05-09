---
title: Android 如何集成微信支付？
date: 2017-06-08 10:01:20
tags: 
- Android
- 微信
- 支付
---

1、微信调用统一订单后会返回下面结果
```json
{
    "return_code": "SUCCESS",
    "return_msg": "OK",
    "appid": "APPID",
    "mch_id": "商户号",
    "nonce_str": "随机字符串",
    "sign": "081675D3A89B1A735613CF2D777E6F06",
    "prepay_id": "wx201706052018103dd047b0880123350695",
    "result_code": "FAIL",
    "err_code": "ORDERPAID",
    "err_code_des": "该订单已支付"
}
```

<!-- more -->

2、Android 需要服务端再次签名返回，因为如果在APP端做校验的话会暴露appkey，对APP的安全性有影响

3、对下面的字符进行拼接之后签名产生sign字段，签名格式[查看官方签名](https://pay.weixin.qq.com/wiki/doc/api/app/app.php?chapter=4_3)

`注意：这里的key为商户的key，而不是AppSecret；timestamp为当前的时间戳`

```java
"appid=" + appid + "&noncestr=" + nonce_str + "&package=Sign=WXPay" + "&partnerid=" + mch_id + "&prepayid=" + prepayid + "&timestamp=" + timeStamp + "&key=" + key;
```

4、最后返回的下面的字段：

```json
{
    "appid": "第1步的APPID",
    "mch_id": "第1步的mch_id",
    "prepay_id": "第1步的prepay_id",
    "nonce_str": "第1步的nonce_str",
    "sign": "第3步的签名，不是第1步的签名"，
    "timestamp":"第3步的时间戳"
}
```

