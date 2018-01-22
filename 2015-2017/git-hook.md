---
title: 使用 git hook 自动部署项目
date: 2016-09-28 22:48:42
tags:
---

### 前言

说起部署，就是一个头痛的问题。每次改动文件，都要上传到服务器，以前通过 FTP 来上传，但十分低效。后来试过写脚本来上传，但也觉得麻烦。

今天搭建好了 Hexo 的博客之后，查看了一下 Hexo 的部署 plugin，提供了部署到 Git、Heroku 等平台的部署插件，以及 FTP 插件。而我的 Hexo 博客是部署在自己的服务器上面，又不打算打开 FTP （鄙视 FTP 党）功能，所以最后选择了 Git Hook

### Git Hooks

Git Hooks 就是一些触发特定事件的脚本。比如 commit、push、merge 等等，也区分本地 Hooks 和服务端 Hooks。

> Git 本身可以调用自定义的挂钩脚本，其中有两组：客户端和服务器端。客户端挂钩用于客户端的操作，如提交和合并。服务器端挂钩用于 Git 服务器端的操作，如接收被推送的提交。

<!-- more -->

Git Hooks 支持的事件有：

- applypatch-msg
- post-update
- pre-rebase
- commit-msg
- pre-applypatch
- update
- post-commit
- pre-commit
- post-receive
- prepare-commit-msg

要实现自动部署，我们只需要 `post-recieve`。

### 大致流程为
1. 本地执行 `git push`；
2. 服务器的 git bare 仓库接受到推送，执行 `post-recieve` 脚本；
3. 把更新的文件移动到工作目录。

### 部署

#### 1. 安装 Git
我使用的服务器 Ubuntu，安装 Git 很简单：

```
apt-get install git
```

#### 2. 创建 git 用户，用来运行 git 服务
```
adduser git
```

#### 3. 创建远程仓库

假设我的博客放在：`/web/www/blog`
Git 仓库放在：`/web/git/`

那么：

```
cd /web/git

# 初始化仓库
git init --bare blog.git

# 给 git 用户添加权限
chown -R git:git blog.git
```

#### 4. 修改 post-receive 脚本

```
cd /web/git/blog.git/hooks

# 如果没有这个文件，手动创建一个
vi post-receive
```

`post-receive` 文件的内容是：

```
#!/bin/sh
GIT_WORK_TREE=/web/www/blog/public git checkout -f
```

#### 5. 给 post-receive 文件添加执行权限

```
chmod +x post-receive
```

#### 6. clone 仓库到本地

```
git clone git@your-server:/web/git/blog.git
```

克隆后，进入 Hexo 博客的 `public` 目录，随便修改点东西，提交，然后 `git-push` 试试。

**Enjoy it.**







