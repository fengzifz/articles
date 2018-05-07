---
title: Ubuntu 或 Debian 下，MySQL 5.7 忘记 root 密码的重置方法
date: 2018-04-11 09:59:41
tags: mysql ubuntu debian
---

也许因为各种原因，我们有时候会忘记了 MySQL 的 root 的密码，网上很多教程，都教你直接 `mysql -u root -p` 登陆 mysql 后修改，但前提是，我忘记了 root 的密码，如何登陆？

密码错误的提示：

```
ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: YES)
```

### debian-sys-maint 用户

MySQL 安装之后，会默认生成几个用户，其中一个就是 `debian-sys-maint` 了，我们可以使用这个用户登陆来修改 `root` 的密码。

> debian-sys-maint 中 Debian 系统对 MySQL 维护用的，可以理解为通过系统的某个“非常规”程序对 MySQL 进行备份恢复等行为时，改程序所使用的登录 MySQL 的账户。

那么，`debian-sys-maint` 的用户密码是多少呢？

在 Ubuntu 下面，可以打开这个文件查看：

```
vi /etc/mysql/debian.cnf
```

然后会看到这个配置文件里面，有个 `password` 的字段：

```
# Automatically generated for Debian scripts. DO NOT TOUCH!
[client]
host     = localhost
user     = debian-sys-maint
password = xxxxxxxxxx
socket   = /var/run/mysqld/mysqld.sock
[mysql_upgrade]
host     = localhost
user     = debian-sys-maint
password = xxxxxxxxxx
socket   = /var/run/mysqld/mysqld.sock
```

把 `password` 记下来，然后在终端使用 `debian-sys-maint` 来登陆：

```
mysql -u debian-sys-maint -p
# 输入密码之后就可以登陆成功
```

登陆到 mysql 之后，用下面命令来重置 `root` 的密码：

```
update mysql.user set authentication_string=PASSWORD('xxxxxx') where user='root';
```

最后刷新一下权限即可：

```
flush privileges;
```


