---
title: 清理Docker，删除没用的文件
date: 2017-03-27 18:28:01
tags:
---

Docker 在前两年的时候就已经火得很，因为它极大地简化了部署项目的流程，并且可以保证生产环境和开发环境的一致性，很多人也走上了 docker 的这条不归路。

我并不是一个 docker 的忠实粉丝，我更加主张按照企业的实际情况，来选择合适的方式来部署、维护、管理项目。

很多历史原因，公司仍在在使用 Docker，它的确好用，新建、重启 container 等都很方便。就算 container 出了问题，我们可以直接把它删掉，用对应的 image 重新建一个 container。

而在最近，发现了 docker 的一个问题，就是它的某些文件，在不知不觉间占用了硬盘，这些文件是什么呢？它可能是没用的 container、image 或者 volumn 等。

就算你很熟悉 docker，但你查看它的 container、image 或者 volumn 目录时，你看到的是：

![](https://www.fengzifz.com/media/14904351284416/14904351561990.jpg)

所以，我们并不知道哪些文件是没用的。既然像我这种低端用户都遇到的问题，那么高级玩家肯定早就遇到了。那么，究竟如何删除 docker 没用的 container、image 或者 volumn 呢？

重点提醒：下面有几条命令可以删除 docker 没用的文件。拿 container 来说，它会删除状态是『exited』的 container，在使用命令之前，请确保你的『exited』的 container 是没有用的。

<!-- more -->

## 1. 删除 exited container

在终端运行 `docker ps -a` 时，可以看到所有的 container，包括 exited 的 container。当然，清除没有用的 container 时，我们可以用 `docker rm <container id>` 一个个删除，但数量太多时，就有点麻烦了。所以，可以使用如下命令，删除所有状态是 exited 的 container：

```
docker rm -v $(docker ps -a -q -f status=exited)
```


参数 `-v` 表示删除与之关联的 volumn。可以在终端运行 `docker rm --help` 查看帮助。

## 2. 删除没用的 image

相比起 container，没用的 images 就更加占用硬盘空间。用 `docker images` 可以查看 images 列表，我们会看到一些 `<none>` 的 images，它们常常是在某个 image 更新后，留下来的上一个版本的 image。


```
<none>              <none>              b94becc70264        19 months ago       757.6 MB
ubuntu              latest              44ae5d2a191e        19 months ago       188.3 MB
<none>              <none>              3f7da654931a        20 months ago       132.8 MB
```

用下面命令把它们干掉：


```
docker rmi $(docker images -f "dangling=true" -q)
```

`dangling=true` 按照中文来翻译的话，意思是指『悬空』的 image，我理解成『没有被使用的 image』。

## 3. 删除没用的 volumn

在第 1 点提及的删除 container 命令中，我们加上 `-v` 可以把对应的没用的 volumn 也删除，但如果你之前很多次使用 `docker rm <container_id>` 来删除 container 时，这个命令并没有删除对应 volumn，那么我们可以使用以下命令来删除已经没用的 volumn：


```
docker volume rm $(docker volume ls -qf dangling=true)
```

现在看看你的服务器是不是已经腾出了很多的空间出来了？



