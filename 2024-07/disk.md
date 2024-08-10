# 通过替换 ZFS 镜像池中的磁盘来扩容

- 原文链接：<https://qiita.com/belgianbeer/items/8df197588462cd7f6b45>

* [FreeBSD](https://qiita.com/tags/freebsd)
* [RAID](https://qiita.com/tags/raid)
* [ 硬盘驱动器](https://qiita.com/tags/hdd)
* [ZFS](https://qiita.com/tags/zfs)
* [ZFSonLinux](https://qiita.com/tags/zfsonlinux)

上次更新于 2023 年 07 月 16 日发布于 2023 年 06 月 24 日

## ZFS 镜像池的磁盘更换与容量扩展

## 开始

自家服务器的硬盘使用量即将超过 80%^ 1^，再加上使用已经超过了 10 年，因此判断还是该更换硬盘了。于是我购买了新的硬盘并进行了更换。本文记录了实际进行硬盘更换的过程。

尽管本文作者的服务器操作系统是 FreeBSD，但是即使在 Linux 上使用 ZFS 的情况下，ZFS 相关操作也是类似的（作者也在 Linux 上使用 ZFS）。

## 服务器存储配置

目标服务器的操作系统是 FreeBSD，作为系统存储有一台 256GB 的 SSD，作为数据存储有两台 2TB 的硬盘，全部使用 ZFS 配置。为了数据保护，两台硬盘配置为 ZFS 镜像，因此尽管有两台硬盘，但容量上等同于一台。本次将两台 2TB 的硬盘更换为各自 6TB 的硬盘。

## 更換操作前的檢查

在實際進行更換操作之前，請確認當前狀況。使用 zpool list 來確認硬碟的使用量等信息如下所示。

```
$ zpool list
NAME    SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP  HEALTH  ALTROOT
zroot   228G  16.8G   211G        -         -    12%     7%  1.00x  ONLINE  -
zvol0  1.81T  1.49T   333G        -         -     5%    82%  1.00x  ONLINE  -
$
```

zroot 是用於系統的 SSD。而 zvol0 是此次更換的硬碟目標，可以看到 ZFS 鏡像配置的使用量達到了 82%。

配置的详细信息可以在 zpool list -v 和 zpool status 中查看。

```
$ zpool list -v zvol0
NAME             SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  
zvol0           1.81T  1.49T   333G        -         -     5%    82%  1.00x    ONLINE  -
  mirror-0      1.81T  1.49T   333G        -         -     5%  82.1%      -    ONLINE
    gpt/ndisk2      -      -      -        -         -      -      -      -    ONLINE
    gpt/ndisk1      -      -      -        -         -      -      -      -    ONLINE
$
```

```
$ zpool status zvol0
  pool: zvol0
 state: ONLINE
  scan: scrub repaired 0B in 05:36:58 with 0 errors on Tue Jan 17 08:40:57 2023
config:

        NAME            STATE     READ WRITE CKSUM
        zvol0           ONLINE       0     0     0
          mirror-0      ONLINE       0     0     0
            gpt/ndisk2  ONLINE       0     0     0
            gpt/ndisk1  ONLINE       0     0     0

errors: No known data errors
$
```

从这些信息我们可以看出，ndisk1 和 ndisk2 的磁盘分区已经设置成了镜像。实际上，这 3 个存储设备分别是 ada0（固态硬盘）、ada1（硬盘）、ada2（硬盘），硬盘分区的配置如下。

```
$ gpart show ada1 ada2
=>        40  3907029088  ada1  GPT  (1.8T)
          40  3907029088     1  freebsd-zfs  (1.8T)

=>        40  3907029088  ada2  GPT  (1.8T)
          40  3907029088     1  freebsd-zfs  (1.8T)

$ gpart show -l ada1 ada2
=>        40  3907029088  ada1  GPT  (1.8T)
          40  3907029088     1  ndisk1  (1.8T)

=>        40  3907029088  ada2  GPT  (1.8T)
          40  3907029088     1  ndisk2  (1.8T)

$
```

可以看出 ada1 和 ada2 都是采用 GPT 格式，ada1 的用于 ZFS 的分区命名为 ndisk1，ada2 的分区则命名为 ndisk2。

## 使用镜像配置更换硬盘的步骤

不仅限于 ZFS，对于镜像（RAID 1）配置的两个硬盘都需要更换的步骤，大概有以下两种方法。

* 使用新硬盘准备镜像，然后从现有镜像复制数据，复制完成后再进行更换。或者先更换再从原始硬盘复制数据。
* 更换一侧硬盘，使用镜像功能同步数据，完成后再更换另一侧硬盘重新同步数据

使用前一种方法，数据复制只需一次，但同时需要连接 3 台或 4 台硬盘，还需要考虑 SATA 接口数量等。在这种情况下，虽然硬盘可以使用 USB 连接，但由于 USB 2.0 传输速度慢，复制所需时间较长，因此最好使用 USB 3.0。

作者选择了后一种逐一更换的方法。这种方法在更换硬盘时可能会有些影响，但即使在同步期间（性能会有所降低），也能像平常一样继续使用服务器，这是其优点。

## 硬盘更换和数据同步

### 注意：在 ZFS 镜像中更换驱动器时

在更换 ZFS 镜像中的驱动器时，请注意在执行任何操作（如 zpool detach zvol0 gpt/ndisk2 ）之前最好不要从 ZFS 池中移除 ndisk2。因为一旦使用 zpool detach 命令将 ndisk2 从池中移除，ndisk2 中的数据将被视为已清除。如果在数据同步到新硬盘时发生 ndisk1 读取错误，并且需要用 ndisk2 重新进行同步，那么一旦 ndisk2 被移除，就无法再次使用它进行同步了。如果只是关闭电源并移除 ndisk2，那么 ndisk2 中的数据将保持与 ndisk1 相同，这样即使发生意外，也可以使用 ndisk2 进行重新同步。

### 更换第一台硬盘

首先更换第一台硬盘，将包含 ndisk2 分区的硬盘取出进行更换。关闭服务器并切断电源，取下旧硬盘，连接新硬盘后通电。

 启动后确认 zvol0 的状态。

```
$ zpool list
NAME    SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT
zroot   228G  16.8G   211G        -         -    12%     7%  1.00x    ONLINE  -
zvol0  1.81T  1.49T   333G        -         -     5%    82%  1.00x  DEGRADED  -
$
```

```
$ zpool status zvol0
  pool: zvol0
 state: DEGRADED
status: One or more devices could not be opened.  Sufficient replicas exist for
        the pool to continue functioning in a degraded state.
action: Attach the missing device and online it using 'zpool online'.
   see: https://openzfs.github.io/openzfs-docs/msg/ZFS-8000-2Q
  scan: scrub repaired 0B in 05:36:58 with 0 errors on Tue Jan 17 08:40:57 2023
config:

        NAME                      STATE     READ WRITE CKSUM
        zvol0                     DEGRADED     0     0     0
          mirror-0                DEGRADED     0     0     0
            12897545936258916783  UNAVAIL      0     0     0  was /dev/gpt/ndisk2
            gpt/ndisk1            ONLINE       0     0     0

errors: No known data errors
```

两个硬盘应该构成镜像，但其中一个 gpt / ndisk2 不见了，只剩下一个，因此在 HEALTH 或 STATE 方面显示为 DEGRADED。另外，在 zpool status 中，显示了 gpt / ndisk2 所在位置的 config：部分中不见了，而是显示为“12897545936258916783”，并以“was /dev/gpt/ndisk2”结尾。稍后将使用此 ID。

### 创建备用分区

为备用硬盘准备工作，要进行 GPT 初始化，并准备用于 ZFS 的分区。

 首先，使用 GPT 初始化硬盘。

```
$ gpart show ada1
gpart: No such geom: ada1.
$ gpart create -s gpt ada1
ada1 created
$ gpart show ada1
=>         40  11721045088  ada1  GPT  (5.5T)
           40  11721045088        - free -  (5.5T)

$
```

然后为 ZFS 分配分区。我将新分区命名为 sdisk1 以区别于旧分区。

```
$ gpart add -t freebsd-zfs -a 4k -l sdisk1 ada1
ada1p1 added
$ gpart show ada1
=>         40  11721045088  ada1  GPT  (5.5T)
           40  11721045088     1  freebsd-zfs  (5.5T)

$ gpart show -l ada1
=>         40  11721045088  ada1  GPT  (5.5T)
           40  11721045088     1  sdisk1  (5.5T)

$
```

### 同步第一台数据

由于已准备好新分区，因此将用 sdisk1 替换旧 ndisk2。为此操作使用 zpool replace 。

通常情况下，您需要指定构成池的设备作为 zpool replace 的源设备，但是在这种情况下，由于设备已被取下，因此没有这样的设备。在这种情况下，请指定显示的 ID，即 zpool status zvol0 。

```
$ zpool replace zvol0 12897545936258916783 gpt/sdisk1
$
```

这将开始同步来自 ndisk1 的镜像到 sdisk1。

同期在后台进行，如前所述，在此期间通常可以正常使用服务器。由于数据量大约为 1.5TB，因此同步需要相当长的时间，而在此期间如果无法了解进展情况，可能会感到有些不安。在 ZFS 中可以通过 zpool status 确认同步进度，从而获得结束时间的大致参考。

```
$ zpool status zvol0
  pool: zvol0
 state: DEGRADED
status: One or more devices is currently being resilvered.  The pool will
        continue to function, possibly in a degraded state.
action: Wait for the resilver to complete.
  scan: resilver in progress since Sat Jan 28 14:25:44 2023
        12.8G scanned at 596M/s, 600K issued at 27.3K/s, 1.49T total
        0B resilvered, 0.00% done, no estimated completion time
config:

        NAME                        STATE     READ WRITE CKSUM
        zvol0                       DEGRADED     0     0     0
          mirror-0                  DEGRADED     0     0     0
            replacing-0             DEGRADED     0     0     0
              12897545936258916783  UNAVAIL      0     0     0  was /dev/gpt/ndisk2
              gpt/sdisk1            ONLINE       0     0     0
            gpt/ndisk1              ONLINE       0     0     0

errors: No known data errors
$
```

在 status: 下显示 One or more devices is currently being resilvered 和 resilver^ 3^ 中（同步中）的状态。在 action: 下显示指示等待 resilver 完成。而在 scan: 中显示处理量和预计时间，但正如 no estimated completion time 所示，同步刚开始时由于无法预知时间，因此不会显示。

过一段时间，将会显示预计完成时间，但根据经验，在这个阶段这些时间通常不可靠。

```
$ zpool status zvol0
===== <省略> =====
  scan: resilver in progress since Sat Jan 28 14:25:44 2023
        265G scanned at 664M/s, 6.51G issued at 16.3M/s, 1.49T total
        6.51G resilvered, 0.43% done, 1 days 02:27:43 to go
===== <省略> =====
$ 
```

进一步确认后，显示需要大约 14 小时才能完成。

```
$ zpool status zvol0
===== <省略> =====
  scan: resilver in progress since Sat Jan 28 14:25:44 2023
        289G scanned at 488M/s, 18.6G issued at 31.4M/s, 1.49T total
        18.6G resilvered, 1.22% done, 13:38:30 to go
===== <省略> =====
$
```

一小时后再次确认时，显示时间已减少到约 6 小时，但第一台镜像的同步最终花了约 15 个半小时。以下是第一台同步完成时检查到的 zpool list 和 zpool status 的结果。

```
$ zpool list -v zvol0
NAME             SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT
zvol0           1.81T  1.49T   333G        -         -     5%    82%  1.00x    ONLINE  -
  mirror-0      1.81T  1.49T   333G        -         -     5%  82.1%      -    ONLINE
    gpt/sdisk1      -      -      -        -         -      -      -      -    ONLINE
    gpt/ndisk1      -      -      -        -         -      -      -      -    ONLINE
$
```

```
$ zpool status zvol0
  pool: zvol0
 state: ONLINE
  scan: resilvered 1.49T in 15:34:45 with 0 errors on Sun Jan 29 06:00:29 2023
config:

        NAME            STATE     READ WRITE CKSUM
        zvol0           ONLINE       0     0     0
          mirror-0      ONLINE       0     0     0
            gpt/sdisk1  ONLINE       0     0     0
            gpt/ndisk1  ONLINE       0     0     0

errors: No known data errors
$
```

HEALTH 和 STATE 均为 ONLINE，无错误发生，第一台同步顺利完成。

### 第二个硬盘的同步

由于第二个硬盘的更换和同步步骤与第一个完全相同，因此只记录命令的执行和结果。

 更换第二个硬盘并启动。

```
$ zpool status zvol0
  pool: zvol0
 state: DEGRADED
status: One or more devices could not be opened.  Sufficient replicas exist for
        the pool to continue functioning in a degraded state.
action: Attach the missing device and online it using 'zpool online'.
   see: https://openzfs.github.io/openzfs-docs/msg/ZFS-8000-2Q
  scan: resilvered 1.49T in 15:34:45 with 0 errors on Sun Jan 29 06:00:29 2023
config:

        NAME                      STATE     READ WRITE CKSUM
        zvol0                     DEGRADED     0     0     0
          mirror-0                DEGRADED     0     0     0
            gpt/sdisk1            ONLINE       0     0     0
            13953654332474917058  UNAVAIL      0     0     0  was /dev/gpt/ndisk1

errors: No known data errors
$
```

 镜子用隔板准备

```
# gpart create -s gpt ada2
ada2 created
# gpart show ada2
=>         40  11721045088  ada2  GPT  (5.5T)
           40  11721045088        - free -  (5.5T)

# gpart add -t freebsd-zfs -a 4k -l sdisk2 ada2
ada2p1 added
# gpart show ada2
=>         40  11721045088  ada2  GPT  (5.5T)
           40  11721045088     1  freebsd-zfs  (5.5T)

# gpart show -l ada2
=>         40  11721045088  ada2  GPT  (5.5T)
           40  11721045088     1  sdisk2  (5.5T)

#
```

 启动第二台同步

```
$ zpool replace zvol0 13953654332474917058 gpt/sdisk2
$ zpool status zvol0
  pool: zvol0
 state: DEGRADED
status: One or more devices is currently being resilvered.  The pool will
        continue to function, possibly in a degraded state.
action: Wait for the resilver to complete.
  scan: resilver in progress since Sun Jan 29 13:38:15 2023
        12.8G scanned at 692M/s, 492K issued at 25.9K/s, 1.49T total
        0B resilvered, 0.00% done, no estimated completion time
config:

        NAME                        STATE     READ WRITE CKSUM
        zvol0                       DEGRADED     0     0     0
          mirror-0                  DEGRADED     0     0     0
            gpt/sdisk1              ONLINE       0     0     0
            replacing-1             DEGRADED     0     0     0
              13953654332474917058  UNAVAIL      0     0     0  was /dev/gpt/ndisk1
              gpt/sdisk2            ONLINE       0     0     0

errors: No known data errors
```

 第二台同步大约需要 14 小时。

```
$ zpool status zvol0
  pool: zvol0
 state: ONLINE
  scan: resilvered 1.49T in 14:11:00 with 0 errors on Mon Jan 30 03:49:15 2023
config:

        NAME            STATE     READ WRITE CKSUM
        zvol0           ONLINE       0     0     0
          mirror-0      ONLINE       0     0     0
            gpt/sdisk1  ONLINE       0     0     0
            gpt/sdisk2  ONLINE       0     0     0

errors: No known data errors
$
```

## 容量扩展

交换和数据同步已顺利完成，但仍有一些工作要做。因为仅仅更换了硬盘，2TB→6TB 的容量扩展还没有生效。

再次确认 zvol0 的容量时，SIZE 显示为 1.81T，与更换前相同。但仔细观察到 EXPANSZ 显示为 3.62T，可知有 3.62TB 的扩展空间。

```
$ zpool list zvol0
NAME    SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT
zvol0  1.81T  1.49T   333G        -     3.62T     5%    82%  1.00x    ONLINE  -
$
```

这个容量扩展的设置是 ZFS 池的 autoexpand 属性。首先检查当前的 autoexpand 值，通常默认为 off。

```
$ zpool get autoexpand zvol0
NAME   PROPERTY    VALUE   SOURCE
zvol0  autoexpand  off     default
$
```

 尝试将 autoexpand 设置为 on。

```
$ zpool set autoexpand=on zvol0
$ zpool get autoexpand zvol0
NAME   PROPERTY    VALUE   SOURCE
zvol0  autoexpand  on      local
$
```

但仅仅将 autoexpand 设置为 on 并不会改变容量。

```
$ zpool list zvol0
NAME    SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT
zvol0  1.81T  1.49T   333G        -     3.62T     5%    82%  1.00x    ONLINE  -
$
```

为了扩展容量，您需要设置 autoexpand=on 并执行 zpool online -e 。

```
$ zpool online -e zvol0
missing device name
usage:
        online [-e] <pool> <device> ...
$
```

由于忘记指定设备名称导致错误😅，但在正常情况下，指定设备名称是必需的。当使用 -e 选项时，好像可以不指定设备名称。无论如何，在 zpool online -e 中，只需指定池内的任何一个设备名称即可解决问题。

如果重新指定设备名称并执行，容量增加到了 5.45TB^ 4^。

```
$ zpool online -e zvol0 gpt/sdisk1
$ zpool list zvol0
NAME    SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT
zvol0  5.45T  1.49T  3.97T        -         -     1%    27%  1.00x    ONLINE  -
$
```

这次在更换硬盘后，我们修改了 autoexpand 属性，如果在更换之前设置好，那么在第二台同步完成后容量将会自动扩展。此外，autoexpand 不仅在镜像中可以使用，在 raidz 等配置中也同样适用。

2023 年 07 月 16 日添加：收到关于使用 zpool online -e 进行扩展时不需要设置 autoexpand 属性的指示。autoexpand 属性被用于自动扩展容量的设置。

## 更换完成后

由于新硬盘的传输速度提高，我能感受到整体性能有所提升。虽然 CPU 已经使用了 10 年以上，但主要用途是文件服务器，所以似乎可以继续使用。

1. 因为在 ZFS 中采用写时复制的方式进行数据更新，当剩余空间不足时，与其他文件系统相比性能会急剧下降(在硬盘上更为明显)，因此不建议使用到容量的极限。使用到什么程度会导致性能下降取决于工作集的配合，因此不能简单地说清空容量是否会对 ZFS 的性能造成影响，就算是其他文件系统中不受影响的剩余空间，在 ZFS 中也可能导致性能下降。
2. 即使将 ndisk2 分离，使用 zpool import -D 可以恢复数据，但我没有尝试过。
3. 「resilver」这个词是我开始使用 ZFS 时了解到的，原意是指擦拭银饰品，但在 ZFS 中表示 RAID 正在进行恢复。
4. 硬盘的 TB 容量以十进制表示，所以 6TB 相当于 6,000,000,000,000 字节，而以二进制表示的 1TB 是 1,099,511,627,776 字节，因此是 5.45TB。
