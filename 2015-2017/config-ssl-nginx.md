---
title: Nginx 配置 GeoTrust SSL 证书
date: 2015-11-16 11:48:27
tags: SSL, GeoTrust, Nginx
---

### 创建 CSR
在目标服务器上面，使用如下命令创建密钥对：

```bash
openssl req -new -newkey rsa:2048 -nodes -keyout yeshm.com.key -out yeshm.com.csr
```
然后服务器会提示输入如下信息：

- Common Name (the domain name certificate should be issued for): **Common-Name**
- Country: **CN**
- State (or province): **GuangDong**
- Locality (or city): **ZhongShan**
- Organization: **Your-company**
- Organizational Unit (Department): **Org-Name**
- E-mail address: **domains@xxx.com**

完成之后，会生成两个文件：
- xxx.com.csr
- xxx.com.key

<!-- more -->

### 申请 CRT
使用如下 3 个步骤申请 CRT：
第一，登录之后，打开如下链接申请 CRT： https://security-center.geotrust.com/process/retail/console_home?application_locale=GEOTRUST_US

第二，找到之前生成的 `xxx.com.csr`，复制里面的内容

```bash
cat xxx.com.csr
```

第三，把它粘贴到第一步打开链接的 text 框里面，点击提交就可以了。

### Nginx 配置 HTTPS
第一，下载 SSL Certificate & Intermediate CA Certificate，申请了 CRT 之后，就可以登录到 [GeoTrust 登录](https://security-center.geotrust.com/process/retail/console_login?application_locale=GEOTRUST_US)，然后点击页面下方的 **Download certificate**，下载下来是一个 .zip 压缩包

第二，把压缩包上传到服务器，解压之后，创建一个叫 `xxx.com.crt` 的文件，然后把 ssl_certificate.crt 和 IntermediateCA.crt 的内容全部复制进去：

```bash
cat ssl_certificate.crt IntermediateCA.crt >> yeshm.com.crt
```

第三，配置 Nginx。在需要配置 HTTPS 的 nginx 的配置文件里面，在 `server` 的上下文里面，添加如下配置：

```bash
server {
	listen 443;

	ssl on;
	ssl_certificate /etc/ssl/xxx.com.crt;
	ssl_certificate_key /etc/ssl/xxx.com.key; 
}
```

第四，重启 nginx 就会生效

---

参考文献：
- [SSL Certificate Installation in Nginx server](https://knowledge.geotrust.com/support/knowledge-base/index?page=content&id=SO25680)
- [Generating CSR on Apache + OpenSSL/ModSSL/Nginx](https://www.namecheap.com/support/knowledgebase/article.aspx/9446/0/apache-opensslmodsslnginx)

