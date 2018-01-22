---
title: 微信公众号支付
date: 2016-01-20 11:45:09
tags:
---

现在越来越多的网店已经支持微信公众号支付，下面为大家介绍一下微信公众号支付的开发。

## 准备

- 微信公众号必须是认证的服务号
- 微信公众号需要申请微信支付接口功能
- 域名必须已经备案
- 你已经能够获取微信用户的 openid

## 配置
在开发之前，需要先到[微信公众号平台](https://mp.weixin.qq.com)进行必要的配置。

<!-- more -->

#### 第一，添加授权回调域名
登陆微信公众号平台，点击左菜单的『接口权限』，然后找到『网页账号』，点击『修改』，填入你的域名，例如：`www.xxx.com`。

#### 第二，设置支付授权目录
点击左菜单的『微信支付』，然后点击『开发配置』，在『公众号支付』一栏下面，添加支付授权目录和测试授权目录。

例如，我们的微信公众号支付逻辑文件位置是：`www.xxx.com/pay/pay.php`，那么，支付授权目录就是：

```
// 注意最后需要加上斜杠：/
www.xxx.com/pay/
```

## 开发

### 流程
详细请看[微信公众号开发流程](https://pay.weixin.qq.com/wiki/doc/api/jsapi.php?chapter=7_4)，这里不再重复多说。

查看流程后，我们把开发步骤分成两个主要步骤：

1. 生成预付订单（prepay_id）
2. 生成支付参数并签名
3. 微信用户支付

#### 生成预付订单
预付订单，所使用的 API 跟微信扫码支付是一样的，也是使用[统一下单 API](https://pay.weixin.qq.com/wiki/doc/api/jsapi.php?chapter=9_1)，需要的参数如下：


| 字段名        | 变量名         | 类型           | 值 |
| ------------ | ------------- | ------------- | ------------- |
| 公众账号ID | appid | String(32) | 在准备阶段已经获得 |
| 商户号 | mch_id | String(32) | 在准备阶段已经获得 |
| 随机字符串 | nonce_str | String(32) | 自定义 |
| 签名 | sign | String(32) | 请参考：[微信扫码支付](http://www.chenzifeng.net/index.php/2016/01/23/%E5%BE%AE%E4%BF%A1%E6%89%AB%E7%A0%81%E6%94%AF%E4%BB%98%E5%BC%80%E5%8F%91/) |
| 商品描述 | body | String(128) | 自定义 |
| 商户订单号 | out_trade_no | String(32) | 自定义（建议使用你的系统的订单号） |
| 总金额 | total_fee | Int | 单位是“分”，提交给微信时需要乘以 100 |
| 终端IP | spbill_create_ip | String(16) | 自行获取 |
| 通知地址 | notify_url | String(256) | 在准备阶段已经获得 |
| 交易类型 | trade_type | String(16) | 公众号支付的交易类型是“JSAPI” |

生成预付订单成功后，微信服务器会返回一个 xml 格式的数据：

 ```
 <xml>
   <return_code><![CDATA[SUCCESS]]></return_code>
   <return_msg><![CDATA[OK]]></return_msg>
   <appid><![CDATA[wx2421b1c4370ec43b]]></appid>
   <mch_id><![CDATA[10000100]]></mch_id>
   <nonce_str><![CDATA[IITRi8Iabbblz1Jc]]></nonce_str>
   <sign><![CDATA[7921E432F65EB8ED0CE9755F0E86D72F]]></sign>
   <result_code><![CDATA[SUCCESS]]></result_code>
   <prepay_id><![CDATA[wx201411101639507cbf6ffd8b0779950874]]></prepay_id>
   <trade_type><![CDATA[JSAPI]]></trade_type>
</xml>
 ```

我们把中间的 `prepay_id` 提取出来。

#### 生成支付参数并签名

支付参数是返回给用户，用户支付时，把这些参数直接提交到微信服务器进行验证并支付的，查看[网页调起支付 API](https://pay.weixin.qq.com/wiki/doc/api/jsapi.php?chapter=7_7&index=6#)，可以看到需要的参数如下：

- 公众号id
- 时间戳
- 随机字符串
- 订单详情扩展字符串（prepay_id）
- 签名方式（新版接口是`MD5`）
- 签名

上述字段中，除了`签名`需要额外处理以外，其他参数都已经知道。具体签名算法请参考[微信签名算法](https://pay.weixin.qq.com/wiki/doc/api/jsapi.php?chapter=4_3)，只要按着步骤做就可以了。

#### 微信用户支付
我们把需要的参数准备好后，前端使用 JSSDK 来调起支付。

```
// 注意：在头部引入微信 JSSDK：<script src="http://res.wx.qq.com/open/js/jweixin-1.0.0.js"></script>

// 检查配置是否正确
wx.config({
   debug: true, // 调试开关
   appId: "xxx",
   timestamp: "xxx",
   nonceStr: "xxx",
       signature: "xxx",
       jsApiList: [
           'checkJsApi',
           'chooseWXPay'
       ]
});

wx.ready(function () {
   document.querySelector('#chooseWXPay').onclick = function () {
       wx.chooseWXPay({
           timestamp: "xxx",
           nonceStr: "xxx",
           package: "xxx",
           signType: "MD5", // 新版支付接口使用 MD5 加密
           paySign: "xxx",
           success: function () {
               alert('支付成功,');
               // Some operations here...
           }
       });
   };
});
    
wx.error(function (res) {
   alert('验证失败:' + res.errMsg);
   // Add Your Code Here If You Need
});
```

到此，微信公众号支付已经基本完成。后续的业务逻辑需要根据自己项目需求进行开发。

