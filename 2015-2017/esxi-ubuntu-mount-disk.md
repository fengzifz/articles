---
title: ESXi 5.5 Ubuntu 添加、挂载硬盘
date: 2017-03-28 21:01:03
tags:
---

今天在 ESXi 里面装了一台 Ubuntu，用来做邮箱服务器，安装 Ubuntu 时，添加的系统盘只有 20G，所以还需要额外添加一个硬盘专门用来装邮箱的数据。

**相关环境：**
- VMware ESXi 5.5
- 操作系统：Ubuntu 12.04.2 LTS
- vSphere Client 5.5

## 1. 通过 vSphere Client 添加硬盘

**注意：因我已经操作完成，所以以下截图仅作为步骤参考。**

#### 1.1 关闭虚拟机，然后编辑虚拟机设置

<!-- more -->

点击下图中的『添加』。

![](https://www.fengzifz.com/media/14906610186512/14906887798343.jpg)


#### 1.2 选择硬盘

![](https://www.fengzifz.com/media/14906610186512/14906890548389.jpg)

#### 1.3 选择『创建新的虚拟磁盘』

![](https://www.fengzifz.com/media/14906610186512/14906891270397.jpg)

#### 1.4 选择『厚置备延迟置零』

这里简单解释一下这三个概念：

- 置备延迟置零（zeroed thick）：以默认的厚格式创建虚拟磁盘。创建过程中为虚拟磁盘分配所需空间。创建时不会擦除物理设备上保留的任何数据，但是以后从虚拟机首次执行写操作时会按需要将其置零。**也就是说立刻分配指定大小的空间，空间内数据暂时不清空，以后按需清空。**
- 厚置备置零（eager zeroed thick）：创建支持群集功能（如 Fault Tolerance）的厚磁盘。在创建时为虚拟磁盘分配所需的空间。与平面格式相反，在创建过程中会将物理设备上保留的数据置零。创建这种格式的磁盘所需的时间可能会比创建其他类型的磁盘长。**也就是说立刻分配指定大小的空间，并将该空间内所有数据清空。**
- 精简置备（Thin）：使用精简置备格式。最初，精简置备的磁盘只使用该磁盘最初所需要的数据存储空间。如果以后精简磁盘需要更多空间，则它可以增长到为其分配的最大容量。**也就是说是为该磁盘文件指定增长的最大空间，需要增长的时候检查是否超过限额。**

![](https://www.fengzifz.com/media/14906610186512/14906891644428.jpg)


#### 1.5 选择虚拟设备节点

这里按默认就可以了。虚拟设备节点，会按照系统的硬盘数量，自动递增下去。如系统已经有两块硬盘，你再为其新增一块时，它的虚拟节点就是：SCSI(0:2)，那么，三块硬盘的节点依次是：

- SCSI(0:0)
- SCSI(0:1)
- SCSI(0:2)

![](https://www.fengzifz.com/media/14906610186512/14906894950440.jpg)

#### 1.6 信息预览，点击『完成』即可

![](https://www.fengzifz.com/media/14906610186512/14906896241493.jpg)

## 2. 在虚拟机里面挂在硬盘

通过 vSphere Client 添加硬盘后，然后我们进入虚拟机里面操作。

#### 2.1 查询系统的硬盘情况

在终端运行 `fdisk -l` 来查看硬盘情况：

```bash

Disk /dev/sda: 21.5 GB, 21474836480 bytes
255 heads, 63 sectors/track, 2610 cylinders, total 41943040 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x00011d48

   Device Boot      Start         End      Blocks   Id  System
/dev/sda1   *        2048      499711      248832   83  Linux
/dev/sda2          501758    41940991    20719617    5  Extended
/dev/sda5          501760    41940991    20719616   8e  Linux LVM

Disk /dev/sdb: 214.7 GB, 214748364800 bytes
86 heads, 25 sectors/track, 195083 cylinders, total 419430400 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x0c32528f

Disk /dev/sdb doesn't contain a valid partition table

```

可以看到 `Disk /dev/sdb doesn't contain a valid partition table`，说明了系统已经检测到 `/dev/sdb` 这块硬盘，但并没有挂载。

顺带说明一下 Linux 系列的系统的硬盘情况：

> 2.6 kernel以后,linux会将识别到的硬件设备,在/dev/下建立相应的设备文件.
如:
sda 表示第1块SCSI硬盘，第二块是sdb，以此类推
hda 表示第1块IDE硬盘(即连接在第1个IDE接口的Master口上)
scd0 表示第1个USB光驱.
当添加了新硬盘后,在/dev目录下会有相应的设备文件产生.cciss的硬盘是个例外,它的
设备文件在/dev/cciss/目录下

然后我们在终端运行如下命令：

```
brw-rw---- 1 root disk 8,  0 Mar 28 13:18 /dev/sda
brw-rw---- 1 root disk 8,  1 Mar 28 13:18 /dev/sda1
brw-rw---- 1 root disk 8,  2 Mar 28 13:18 /dev/sda2
brw-rw---- 1 root disk 8,  5 Mar 28 13:18 /dev/sda5
brw-rw---- 1 root disk 8, 16 Mar 28 13:18 /dev/sdb
brw-rw---- 1 root disk 8, 17 Mar 28 13:18 /dev/sdb1
```

说明该系统一共有两个硬盘，分别是：sda 和 sdb。而 sdx（x 代表数字）表示该磁盘的分区。

#### 2.2 创建未被挂载的磁盘分区

```bash
fdisk /dev/sdb
```

此时会进入 command 命令状态，我们可以按照提示进行操作：

- `m` 查看帮助
- `p` 查看分区信息
- `n` 创建分区
- `w` 保存并退出

以我这里为例，我用 `/dev/sdb` 作为数据盘，所以，这块硬盘，除了主分区外，其他容量的都分到第一个分区。那么我使用 `fdisk /dev/sdb` 操作步骤是：

1. 按 `n`；
2. 然后按 `p`，先创建主分区；
3. 然后直接按 enter，因为我只需要创建一个分区，所以直接按照默认（最大）的容量分配；
4. 按 `w` 保存并退出。

完成后，运行 `fdisk -l` 就可以看到 `sdb` 的硬盘信息了：

```bash
Disk /dev/sda: 21.5 GB, 21474836480 bytes
255 heads, 63 sectors/track, 2610 cylinders, total 41943040 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x00011d48

   Device Boot      Start         End      Blocks   Id  System
/dev/sda1   *        2048      499711      248832   83  Linux
/dev/sda2          501758    41940991    20719617    5  Extended
/dev/sda5          501760    41940991    20719616   8e  Linux LVM

Disk /dev/sdb: 214.7 GB, 214748364800 bytes
86 heads, 25 sectors/track, 195083 cylinders, total 419430400 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x0c32528f

   Device Boot      Start         End      Blocks   Id  System
/dev/sdb1            2048   419430399   209714176   83  Linux
```

#### 2.3 格式化未挂在的硬盘

```
mkfs.ext3 /dev/sdb1
```

完成后，看在终端最下面看到如下信息（只截取部分）：

```bash
Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (xxxxx blocks): done
Writing superblocks and filesystem accounting information: done 
```

#### 2.4 创建目录并挂在

```
# 1. 在根目录创建一个 data 目录
mkdir /data

# 2. 挂在硬盘
mount /dev/sdb1 /data
```

大功告成，运行 `df -h` 就可以看到已经挂在好的硬盘：

```bash
/dev/mapper/mail-root   19G  802M   17G   5% /
udev                   494M  4.0K  494M   1% /dev
tmpfs                  201M  256K  201M   1% /run
none                   5.0M     0  5.0M   0% /run/lock
none                   502M     0  502M   0% /run/shm
/dev/sda1              228M   26M  191M  12% /boot
/dev/sdb1              197G  188M  187G   1% /data
```

#### 2.5 开机自动挂载

因为 `mount` 在重启系统后会失效，所以我们把它写到 `/etc/fstab` 文件里面，让它永久挂载：

```
# 1. 编辑 /etc/fstab 文件
vi /etc/fstab

# 2. 在文件最下面加上
/dev/sdb1       /data       ext3       defaults    0    0
```

完成。

#### 2.6 验证 mount

```
mount -a
```

运行上面命令，看系统有无报错，如果报错，那么就是上一步编辑文件出错，需要检查一下。如果没有报错，重启一下系统，然后再次运行 `df -h` 查看一下有无自动挂载。



