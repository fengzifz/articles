---
title: Mac 使用 tar 压缩生成 "._" 文件的解决办法
date: 2017-03-20 17:51:15
tags: Mac tar
---

在 Mac 下使用 tar 命令来压缩文件，当在其他平台解压后，会发现每一个文件都多了一份 `._` 开头的文件副本，例如，我们的正常文件里面是这样的：

```bash
html
  | style.css
  | test.js
```

在 Mac 下使用如下命令对其进行压缩：

```bash
tar -cvf html.tar.gz html
```

<!-- more -->

然后把 `html.tar.gz` 复制到其他平台，例如 Linux（**注意：我只测试过 Linux 平台，其他如 Windows 未进行验证**），然后进行解压，会发现里面的文件是：

```bash
html
  | style.css
  | test.js
  | ._style.css
  | ._test.js
```
  
## 原因

解释见：[Why do I get files like ._foo in my tarball on OS X?](https://superuser.com/questions/61185/why-do-i-get-files-like-foo-in-my-tarball-on-os-x) 最高票的答案。

不想打开链接看的话，可以看下面一段话：

> OS X's tar also knows how to convert the ._ members back to native formats, but the ._ files are usually kept when archives are extracted on other platforms. 

简单翻译是：

> Mac OSX 系统的 tar 知道如何把 ._ 文件转换回本来的文件格式，但是，其他系统往往会把 ._ 这些文件保留下来。

其实好像也没有把根本原因解释出来，大概就是 Mac 下的 tar 机制问题，这个我也不深究了，有兴趣的朋友可以探讨一下。我们来说说解决办法：

## 使用 COPYFILE\_DISABLE 来防止 Mac 的 tar 生成 ._ 文件

```bash
COPYFILE_DISABLE=1 tar -cvf html.tar.gz html
```

## 如何批量删除 ._ 文件

我们往往压缩的文件，都不像上面例子里面的机构那么简单，那么假如不行压缩一个目录结构负责、文件众多的目录后，移到其他平台发现里面很多 `._` 的文件，如何批量删除呢？

**使用 `find` 命令并配合 `xargs` 来删除。** `xargs` 可以把 `find` 找到的文件作为参数传递过来，并对其执行对应的命令：

```bash
find . -name "._*" | xargs rm -f 
```

