---
title: 微信扫码支付开发
date: 2016-01-18 11:42:01
tags:
---

不少开发者看完微信的开发文档之后，都已经哭了。最近做一个微信支付的功能，踩了不少坑，有些坑爬了很久才出来，趁现在还记得，赶紧记录下来。

## 微信扫码支付相关文档

- [扫码支付 - 模式一](https://pay.weixin.qq.com/wiki/doc/api/native.php?chapter=6_1)
- [扫码支付 - 模式二](https://pay.weixin.qq.com/wiki/doc/api/native.php?chapter=6_5)
- [API 列表统一下单](https://pay.weixin.qq.com/wiki/doc/api/native.php?chapter=9_1)

## 微信扫码支付之模式二
### 流程
需要了解微信扫码支付模式二的支付流程，可以去到[这里](https://pay.weixin.qq.com/wiki/doc/api/native.php?chapter=6_5)了解，这里不再解释。

<!-- more -->

### 准备
1. 微信支付需要已经认证过的服务号才能开通支付；
2. 开通支付权限后，登陆[微信公众后台](https://mp.weixin.qq.com)，点击左菜单的“微信支付”（如果没有，请先添加插件功能），然后点击“开发配置”；
3. 在“扫码支付”一栏，下面填入支付回调 url，例如：`http://www.xxx.com/wechat/callback/notify.php`（这里可以根据具体情况再修改）；
4. 登陆[微信商户平台](https://pay.weixin.qq.com)（如果没有商户账号，请先登陆[微信公众后台](https://mp.weixin.qq.com)，左菜单 -> 微信支付 -> 支付申请，申请通过后，微信团队会发邮件给你，里面包含了商户账号和密码）；
5. 登陆微信商户平台后，点击左菜单账户设置下的“操作证书”，点击安装操作证书；然后点击左菜单的“API安全”，在“API密钥”一栏下面，设置密钥；

上述完成后，我们有如下资料：
- 微信公众号 app ID
- 微信公众号 app secret
- 微信商户号 商户 ID
- 微信商户号 KEY
- 支付回调 url 地址

然后进入开发阶段。

### 开发
扫码支付模式二可以简化成两个开发阶段：

1. 生成二维码；
2. 等待微信后台通知回调进行订单操作。

然后我们逐个击破。

#### 生成二维码
这里所说的二维码，是指支付二维码，就是用户使用微信扫码二维码后，然后使用微信进行付款。我们向微信申请的是一个协议为 `weixin://` 的支付链接，然后我们自己使用这个链接来生成二维码。

那么，申请支付链接需要哪些东西呢？这里可以查看微信开发文档[统一下单 API](https://pay.weixin.qq.com/wiki/doc/api/native.php?chapter=9_1)。

在开发初期，我们尽可能地把开发和调试的难度降到最低（特别像微信这么坑的开发文档 -_-|||），那么，从微信开发文档[统一下单 API](https://pay.weixin.qq.com/wiki/doc/api/native.php?chapter=9_1)可以看到，有如下必填项：

**生成支付支付链接必要变量**

| 字段名        | 变量名         | 类型           | 值 |
| ------------ | ------------- | ------------- | ------------- |
| 公众账号ID | appid | String(32) | 在准备阶段已经获得 |
| 商户号 | mch_id | String(32) | 在准备阶段已经获得 |
| 随机字符串 | nonce_str | String(32) | 自定义 |
| 签名 | sign | String(32) |  |
| 商品描述 | body | String(128) | 自定义 |
| 商户订单号 | out_trade_no | String(32) | 自定义（建议使用你的系统的订单号） |
| 总金额 | total_fee | Int | 单位是“分”，提交给微信时需要乘以 100 |
| 终端IP | spbill_create_ip | String(16) | 自行获取 |
| 通知地址 | notify_url | String(256) | 在准备阶段已经获得 |
| 交易类型 | trade_type | String(16) | 模式二的交易类型是“NATIVE” |

- 公众账号 ID -> appid -> 在准备阶段已经获得
- 商户号 -> mch_id -> 在准备阶段已经获得
- 随机字符串 -> nonce_str -> 自定义
- 签名 -> sign -> ???
- 商品描述 -> body -> 自定义
- 商户订单号 -> out_trade_no -> 自定义（建议使用你的系统的订单号）
- 总金额 -> total_fee -> 单位是“分”，提交给微信时需要乘以 100
- 终端 IP -> spbill_create_ip -> 自行获取
- 通知地址 -> notify_url -> 在准备阶段已经获得
- 交易类型 -> trade_type -> 模式二的交易类型是“NATIVE”

那么，在上述获取生成支付二维码的必填字段中，我们现在就只剩下“签名”了。

参阅微信开发文档[安全规范](https://pay.weixin.qq.com/wiki/doc/api/native.php?chapter=4_3)的第 1 点：签名算法。

我们使用如下参数来生成签名：

```
appid: wx1db1c82222222222
mch_id: 1301111111
spbill_create_ip: $_SERVER['REMOTE_ADDR'] // PHP，例如：192.168.1.109
nonce_str: 1234567890 // Max langth: 32
body: Test
total_fee: 1
notify_url: http://www.xxx.com/wechat/callback/notify.php // 注意需要 urlencode
trade_type: NATIVE
```

第一，参数名按照 ASCII 码从小到大排序（字典序），然后进行拼接：

```
// 这是第一个字符串
first_string="appid=wx1db1c82222222222&body=Test&mch_id=1301111111nonce_str=1234567890&notify_url=http%3A%2F%2Fwww.xxx.com%2Fwechat%2Fcallback%2Fnotify.php&spbill_create_ip=192.168.1.109&total_fee=1&trade_type=NATIVE"
```

第二，拼接 API 密钥（就是我们在准备阶段得到的微信商户 KEY），假设我们的 KEY 值是 `11111111112222222222333333333344`，然后进行拼接：

```
// 这是第二个字符串
second_string=first_string . "&key=11111111112222222222333333333344"

// 然后生成签名 sign
// 如下是 PHP 代码
MD5(second_string).toUpperCase()

// 假设得到的值是，那么，这个就是我们得到的签名了
9A0A8659F005D6984697E2CA0A9CF3B7
```

那么，生成支付支付链接必要变量全部准备好，然后我们把这些必要变量整理成 `xml` 格式。不会的同学请自行查阅相关语言的开发文档。

最终得到我们用来申请支付链接的数据如下：

```
<xml>
<appid>wx1db1c82222222222</appid>
<mch_id>1301111111</mch_id>
<body>Test</body>
<total_fee>1</total_fee>
<spbill_create_ip>192.168.1.109</spbill_create_ip>
<nonce_str>1234567890</nonce_str>
<trade_type>NATIVE</trade_type>
<notify_url>http://www.xxx.com/wechat/callback/notify.php</notify_url>
<sign>9A0A8659F005D6984697E2CA0A9CF3B7</sign>
<xml>
```

然后我们把这些数据 post 到微信这个接口 `https://api.mch.weixin.qq.com/pay/unifiedorder`，就会得到支付二维码：

```php
// 以下是 PHP 示例代码

// 需要发送的数据
$xml = '<xml>
<appid>wx1db1c82222222222</appid>
<mch_id>1301111111</mch_id>
<body>Test</body>
<total_fee>1</total_fee>
<spbill_create_ip>192.168.1.109</spbill_create_ip>
<nonce_str>1234567890</nonce_str>
<trade_type>NATIVE</trade_type>
<notify_url>http://www.xxx.com/wechat/callback/notify.php</notify_url>
<sign>9A0A8659F005D6984697E2CA0A9CF3B7</sign>
<xml>';

// 向微信接口发起请求
$ch = curl_init();
curl_setopt($ch, CURLOPT_URL, "https://api.mch.weixin.qq.com/pay/unifiedorder");
curl_setopt($ch, CURLOPT_POST, true);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
curl_setopt($ch, CURLOPT_POSTFIELDS, $xml);
$output = curl_exec($ch);
curl_close($ch);

// 结果
$outputXml = simplexml_load_string($output);
$outputArr = array($xml->getName() => XML2Array($outputXml));
$qrcodeUrl = $outputArr['xml']['code_url']; // 最终得到的支付二维码，例如：weixin://wxpay/s/An4baqw

/**
 * 以下函数是把 XML 转为 Array
 */
function XML2Array(SimpleXMLElement $parent) {
        $array = array();

        foreach ($parent as $name => $element) {
            ($node = & $array[$name])
            && (1 === count($node) ? $node = array($node) : 1)
            && $node = & $node[];

            $node = $element->count() ? XML2Array($element) : trim($element);
        }

        return $array;
    }
```

假设我们的到的支付链接是：

```
weixin://wxpay/s/An4baqw
```

那么，现在我们剩下的就是把这个链接转成二维码。

这里可以使用第三方的生成二维码工具 [PHP Qrcode](phpqrcode.sourceforge.net)。

下载后直接引入这个文件：`phpqrcode.php`，然后：

```php
QRcode::png('weixin://wxpay/s/An4baqw', 'img_pay_qrcode/pay_qrcode.png', 10, 6); // 10 表示生成的二维码图片的大小，6 表示二维码图片的留白区域
```

那么，到这里为止，生成二维码步骤已经完成。

