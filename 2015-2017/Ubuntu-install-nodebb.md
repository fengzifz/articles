---
title: Ubuntu 安装 NodeBB 完整教程
date: 2017-07-13 09:09:49
tags: Nodejs NodeBB
---

最近无意看见 NodeBB 这个论坛挺漂亮的，于是就折腾一下，自己搭建 NodeBB 玩一下。

---

### 环境
1. 服务器环境：Ubuntu 14.04 x64

2. NodeBB 需要的环境：
	- Node.js：v4+
	- Redis：v2.8.9+
	- MongoDB：v2.6+
	- Nginx：v1.3.13+（如果需要做反向代理才需要）

### 安装 Node.js

我们安装 Node.js v7 版本：

```bash
# Install Node.js 7.x repository
curl -sL https://deb.nodesource.com/setup_7.x | bash -

# Install Node.js and npm
apt-get install -y nodejs
```

期间遇到报错的话，直接 google 错误的提示即可，还是比较好解决的。

<!-- more -->

### 安装 MongoDB

以下几条命令直接复制官方的安装步骤：

```bash
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv EA312927
```

```
echo "deb http://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/3.2 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.2.list
```

```
sudo apt update && sudo apt install -y mongodb-org
```

### 安装 Redis

先 update 以下 package：

```bash
apt-get update
```

然后直接安装最新版 redis：

```bash
apt-get install redis-server -y
```

安装期间出错，可以自行 google 解决之。

### 安装 Nginx

这个不再哆嗦了，网上已经一大堆教程了。

### 安装 NodeBB

首先，先安装 git 和 build-essential。

```bash
sudo apt-get install -y git build-essential
```

然后，把 NodeBB 复制下来。

```bash
git clone -b v1.5.x https://github.com/NodeBB/NodeBB.git $HOME/nodebb
```

最后进入 nodebb 目录，运行下面命令进行安装。

```bash
npm install --production

./nodebb setup
```

安装成功后，在本机网络访问 `http://localhost:4567` 即可访问 NodeBB。

### NodeBB - Login 403 forbidden 报错

安装好 NodeBB 后，点击登录时，可能会出现 `403 forbidden` 的问题，这个问题是我在折腾过程中，耗时最多的问题，然后我们需要知道问题的根本原因所在，才能解决这个问题。

经过一番研究，主要有以下两种情况及其解决办法。

#### 1. socket.io 和 socket.io-client 版本问题

在 NodeBB v1.3.x 版本中，如果安装好 NodeBB 后，出现 login 403 forbidden 的问题，这是因为 NodeBB v1.3.x 中，socket.io-client 的版本是 1.7，在这个版本中，socket.io 的文件路径改变了，见 [socketio/socket.io#2766](https://github.com/socketio/socket.io/pull/2766)，导致 socket.io 不可用，所以我们需要同时修改 `socket.io` 和 `socket.io-client` 的版本：

打开 NodeBB 的根目录，编辑 `package.json`，修改 `socket.io` 和 `socket.io-client` 的版本：

```json
"dependencies": {
    .....
    "socket.io": "1.4.8",
    "socket.io-client": "1.4.0",
   ....
}
```

然后重新运行：

```bash
npm install
```

然后重启 NodeBB。

#### 2. socket.io 访问控制源的问题（Access Control Allow Origin）

在 NodeBB v1.5.x 中，socket.io 和 socket.io-client 的版本是：

```json
"socket.io": "2.0.1",
"socket.io-client": "2.0.1",
```

但如果仍然出现 login 403 forbidden 的问题，这次的问题跟 socket.io 和 socket.io-client 的版本无关了，我们需要设置 socket.io 允许访问的域名，打开根目录的 `config.json`，然后加入：

```json
"socket.io": {
    "origins": "domain_1:domain_2"
}
```

请按照自己的需求，修改 `domain_2` 和 `domain_2`。
如果你在测试环境，那可以直接修改成：

```json
"socket.io": {
    "origins": "*:*"
}
```

---

**参考资料**

- [CORS problem (No 'Access-Control-Allow-Origin') on client](https://github.com/socketio/socket.io/issues/2294)
- [socket.io-client 1.7.0+ dist path change breaks NodeBB](https://github.com/NodeBB/NodeBB/issues/5241)
- [NodeBB Github](https://github.com/NodeBB/NodeBB)


