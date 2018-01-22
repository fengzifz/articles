---
title: Ubuntu 修改静态 IP 详细步骤
date: 2017-03-29 15:01:43
tags: Ubuntu
---

之前新安装了一台 Ubuntu 服务器，因公司所有的上网设备都需要绑定 IP + MAC 地址，而在路由器绑定了之后，发现 Ubuntu 无法上网。于是 Google 了一番，终于搞掂，记录一下。

在路由器上面，把 Ubuntu 设定成为固定 IP 后，而 Ubuntu 还是使用 DHCP 来获取 IP，而检查后，发现其 DNS 失效了。

```
cat /etc/resolv.conf
```

发现里面空空如也，问题出在这里，但具体为什么，我没有细究，下面来一步步操作，如何把 Ubuntu 设置成静态 IP。

<!-- more -->

## 1. 设置静态 IP

网络配置文件的位置在 `/etc/network/interfaces`。

```
vi /etc/network/interfaces
```

内容如下：

```bash
auto eth0 

# 1. 把 dhcp 改成 static
# iface eth0 inet dhcp
iface eth0 inet static

# 2. 然后添加下面信息
address 192.168.1.100
netmask 255.255.255.0
gateway 192.168.1.1 
```

## 2. 设置 DNS

DNS 配置文件在 `/etc/resolv.conf`。

```
vi /etc/resolv.conf
```

内容如下：

```bash
nameserver 192.168.1.1 # 主 dns
nameserver 8.8.8.8 # 备用 dns
```

## 3. 重启网络服务

```
/etc/init.d/networdking restart
```

如果看到下面信息，证明配置成功：

```bash
* Running /etc/init.d/networking restart is deprecated because it may not enable again som interfaces
* Reconfiguring network interfaces...
ssh stop/waiting
ssh start/running, process 1418
                                 [OK]
```

这样，Ubuntu 的静态 IP 设置就完成了。

## 4. Ubuntu reboot 后无法上网问题

#### 4.1 DNS 设置失效

Ubuntu 在重启后，在 `/etc/resolv.conf` 里面的设置会丢失了，`/etc/resolv.conf` 是动态创建的，重启后会被覆盖掉。

**解决办法：**

在 `/etc/resolvconf/resolv.conf.d/base` 中添加 DNS 配置：

```bash
nameserver 192.168.1.1
nameserver 8.8.8.8
```

现在，重启服务器后，DNS 配置也不会失效了。

全部完成。
