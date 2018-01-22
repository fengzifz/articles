---
title: php7 安装出现 "locale - Cannot set LC_CTYPE to default locale" 错误
date: 2016-10-20 09:17:26
tags: php7 Ubuntu
---

在安装 PHP7 时，如果服务器的语言包缺失，那么就会出现下面类似的错误：

```bash
locale: Cannot set LC_CTYPE to default locale: No such file or directory
locale: Cannot set LC_MESSAGES to default locale: No such file or directory
locale: Cannot set LC_ALL to default locale: No such file or directory
```

### 服务器环境
Ubuntu 14-64bit

### 解决办法

#### 情况一
如果只是出现以下两个错误：

```bash
locale: Cannot set LC_CTYPE to default locale: No such file or directory
locale: Cannot set LC_MESSAGES to default locale: No such file or directory
```
<!-- more -->

那么运行如下命令来解决：

```bash
sudo locale-gen en_GB.UTF-8
sudo dpkg-reconfigure locales

# 然后可以运行 locale 命令，看是否还有其他错误
```

#### 情况二
如果 local 缺失包含了 `LC_ALL` 的错误：

```bash
locale: Cannot set LC_ALL to default locale: No such file or directory
```

那么可以重装 language 包：

```bash
export LC_ALL="en_GB.UTF-8"
apt-get install --reinstall language-pack-en-base
```

再运行 `locale` 命令，查看是否还有其他类似的 `locale: xxx No such file or directory` 的错误

