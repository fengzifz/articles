---
title: 配置免费 HTTPS - letsencrypt ssl
date: 2016-10-21 11:30:11
tags: HTTPS SSL letsencrypt
---

### 前言

随着 HTTPS 的逐步普及，像我们这种用来装逼的个人博客，有必要也弄一个 HTTPS。但是，大牌厂商的 SSL 证书很贵，作为一个个人网站，没必要花这笔钱。

而在去年，为了加快 HTTPS 的普及，Cisco、Mozilla、EFF 和 CoreOS 等联合创办了一个组织，叫[Internet Security Research Group](https://letsencrypt.org/about/)，简称 ISRG。

ISRG 提供了免费、自动化和开放的 certificate authority (CA) 服务，纵使每三个月就要运行一次命令来 renew，但穷逼们还是可以去尝试一下的。

### 为什么个人博客要使用 HTTPS

1. **逼格高**（这个是重点）；
2. 防止国内运营商劫持；
3. 数据传输安全；
4. 免费；
5. Let's encrypt 容易部署；

### 配置 Let's Encrypt 证书

配置 Let's Encrypt 证书分两个大的步骤：

1. 生成证书；
2. 配置证书（本文以 Nginx 为例）。

<!-- more -->

#### 1. 生成证书
官方推荐使用 [Certbot](https://certbot.eff.org/#ubuntutrusty-nginx) 这套自动化工具来实现，使用十分简单，具体有 2 个步骤：

（1）在服务器下载安装 certbot；
（2）执行命令生成证书；

##### （1）下载安装 certbot

```
wget https://dl.eff.org/certbot-auto
chmod a+x certbot-auto
```

##### （2）执行命令生成证书
```
./certbot-auto certonly --webroot -w /data/www/www.fengzifz.com/public -d fengzifz.com -d www.fengzifz.com
```

期间需要输入如下信息：
- 域名
- 邮箱

操作成功后，终端会提示：

```
IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at
   /etc/letsencrypt/live/fengzifz.com/fullchain.pem. Your cert will
   expire on 2017-01-19. To obtain a new or tweaked version of this
   certificate in the future, simply run certbot-auto again. To
   non-interactively renew *all* of your certificates, run
   "certbot-auto renew"
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
```

证书文件都存放在这个类似的路径下面：

```
/etc/letsencrypt/live/fengzifz.com/
```

#### 在 Nginx 上配置证书

Mozilla 提供了一个 HTTPS 自动生成配置文件的工具：[Mozilla SSL Configuration Generator](https://mozilla.github.io/server-side-tls/ssl-config-generator/)，

主要加上下面这些配置：

```
server {
        listen 80;
        server_name fengzifz.com www.fengzifz.com;
        rewrite ^(.*)$  https://$host$1 permanent;
}

server {
    listen 443 ssl http2;

    server_name fengzifz.com www.fengzifz.com;

    ssl_certificate /etc/letsencrypt/live/fengzifz.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/fengzifz.com/privkey.pem;
    ssl_dhparam /etc/nginx/ssl/dhparam.pem;
    ssl_trusted_certificate /etc/letsencrypt/live/fengzifz.com/root_ca_cert_plus_intermediates;
    resolver <这里填你的 VPS 的 DNS 服务器地址>;
```

注意，上述配置项当中，如下两项配置还没有:

- dhparam.pem
- root_ca_cert_plus_intermediates

`dhparam.pem` 添加方法是：

```
mkdir /etc/nginx/ssl

# 以下命令可能要花费 10 分钟
openssl dhparam -out /etc/nginx/ssl/dhparam.pem 2048
```

`root_ca_cert_plus_intermediates` 添加方法是:
```
cd /etc/letsencrypt/live/fengzifz.com
wget https://letsencrypt.org/certs/isrgrootx1.pem
mv isrgrootx1.pem root.pem
cat root.pem chain.pem > root_ca_cert_plus_intermediates
```


完成后，重启 Nginx，刷新网址，就变成 HTTPS 了。


