---
title: 把本地的 git 推送到不同的远程仓库
date: 2016-11-21 09:43:57
tags: git
---

**这是一个奇葩的需求。**

假设我们已经从 github 那里克隆了一个仓库到本机，然后我们进入这个仓库。

再假设我们在 coding.net 上面有另外一个空的仓库，我们现在要把本机这个从 github 克隆下来的仓库，也备份到 coding.net 的这个空仓库上面，具体操作是：

```
// 1. 进入本地仓库
cd rep_from_github 

// 2. 使用 remote add 命令来添加一个远程仓库
git remote add coding git@git.coding.net:xxx/xxx.git

// 3. 直接推送
git push coding master
```

Enjoy it.
