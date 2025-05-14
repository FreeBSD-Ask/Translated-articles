# 通过将 ZFS 池构建为镜像结构，消除异常数据的影响

- 原文：[ZFSプールをミラーで構成することで、不正データの影響を無くす](https://qiita.com/belgianbeer/items/0e69cf3c3f0fc3c89adc)
- 作者：みんみん
- 上次更新于 2022-12-26

## 开始之前

在上一篇文章 [试着破坏了 ZFS 池](https://qiita.com/belgianbeer/items/477de8ddc64787442c0b)中，我们确认了 ZFS 能够正确地检测出磁盘上的非法数据。

能够检测出非法数据这一点，比起那些无法检测到非法数据而可能在读取时引发错误行为的文件系统，要好得多。但既然能检测到非法数据，自然而然就会想到：那是否可以将其恢复呢？

在 ZFS 中，如果构建了具备冗余性的池，即使存在非法数据，也可以获取到正确的数据。具体来说，可以使用镜像（mirror）、RAID-Z 或 RAID-Z2 等池结构。

本文将基于与[试着破坏了 ZFS 池](https://qiita.com/belgianbeer/items/477de8ddc64787442c0b)相同的实验，在镜像结构的 ZFS 池上进行测试。

**2022 年 12 月 26 日补充了执行 scrub 后的磁盘状态**

## 准备 ZFS 镜像池

为了实验，准备了 2 台通过 USB 连接的 HDD，并在每台硬盘上创建了 4GB 的分区，用于构建镜像结构的 ZFS 池。

[![DSCF2553.JPG](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F130638%2F789a1ff3-ea8a-ab11-3f7b-f3db1c5364cb.jpeg?ixlib=rb-4.0.0\&auto=format\&gif-q=60\&q=75\&s=5c42c540ecb2ec6b7b329fb3bb1e565b)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F130638%2F789a1ff3-ea8a-ab11-3f7b-f3db1c5364cb.jpeg?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=5c42c540ecb2ec6b7b329fb3bb1e565b)

这两块 HDD 分别是 `/dev/da1` 和 `/dev/da2`，使用 GPT 创建分区表，并在每块硬盘上创建了带有 tt1 和 tt2 标签的分区。

```sh
$ gpart create -s gpt da1
da1 created
$ gpart create -s gpt da2
da2 created
$ gpart add -t freebsd-zfs -a 4k -s 4g -l tt1 da1
da1p1 added
$ gpart add -t freebsd-zfs -a 4k -s 4g -l tt2 da2
da2p1 added
$
$ gpart show da1
=>       40  488397088  da1  GPT  (233G)
         40    8388608    1  freebsd-zfs  (4.0G)
    8388648  480008480       - free -  (229G)

$ gpart show da2
=>       40  312581728  da2  GPT  (149G)
         40    8388608    1  freebsd-zfs  (4.0G)
    8388648  304193120       - free -  (145G)

$ gpart show -l da1
=>       40  488397088  da1  GPT  (233G)
         40    8388608    1  tt1  (4.0G)
    8388648  480008480       - free -  (229G)

$ gpart show -l da2
=>       40  312581728  da2  GPT  (149G)
         40    8388608    1  tt2  (4.0G)
    8388648  304193120       - free -  (145G)

$
```

至此，镜像所需的两个 HDD 各自的分区已创建完成。

接下来，将这两个分区组合成一个 ZFS 镜像池。当然和[上次的实验](https://qiita.com/belgianbeer/items/477de8ddc64787442c0b)一样，不启用 ZFS 压缩。

```sh
$ zpool create -O atime=off ztest mirror gpt/tt1 gpt/tt2
$
```

如下所示，镜像结构的 ZFS 池已创建完成。

```sh
$ zpool status ztest
  pool: ztest
 state: ONLINE
config:

	NAME	     STATE     READ WRITE CKSUM
	ztest	     ONLINE	  0	0     0
	  mirror-0   ONLINE	  0	0     0
	    gpt/tt1  ONLINE	  0	0     0
	    gpt/tt2  ONLINE	  0	0     0

errors: No known data errors

$ zpool list -v ztest
NAME          SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT
ztest        3.75G  3.62G   128M        -         -    72%    96%  1.00x    ONLINE  -
  mirror-0   3.75G  3.62G   128M        -         -    72%  96.7%      -    ONLINE
    gpt/tt1      -      -      -        -         -      -      -      -    ONLINE
    gpt/tt2      -      -      -        -         -      -      -      -    ONLINE

$ zfs list ztest
NAME	USED  AVAIL	REFER  MOUNTPOINT
ztest	420K  3.62G	  96K  /ztest

$ df /ztest/
Filesystem 512-blocks Used   Avail Capacity  Mounted on
ztest         7601576  192 7601384     0%    /ztest
```

## 写入伪数据

那么就像上次实验一样，使用 `dd` 命令用零数据填满文件系统。

```sh
$ dd if=/dev/zero of=/ztest/dummy bs=1m
dd: /ztest/dummy: No space left on device
3711+0 records in
3710+1 records out
3891134464 bytes transferred in 151.249101 secs (25726662 bytes/sec)

$ df -h /ztest/
Filesystem    Size    Used   Avail Capacity  Mounted on
ztest         3.6G    3.6G      0B   100%    /ztest

$ zfs list -o space ztest
NAME   AVAIL   USED  USEDSNAP  USEDDS  USEDREFRESERV  USEDCHILD
ztest     0B  3.63G        0B   3.62G             0B       672K

$ ls -l /ztest/dummy
-rw-r--r--  1 root  wheel  3891134464 Dec 22 17:43 /ztest/dummy
```

## 破坏写入的数据

接着像上次一样，强行写入垃圾数据，使其在 ZFS 中触发错误。由于是为了验证镜像冗余性和 ZFS 的修复功能，因此只向 da1 这一侧写入垃圾数据。

通常，从构成镜像的磁盘中读取数据时，为了分散负载和提升性能，会交替访问两个磁盘。ZFS 的镜像结构也有类似行为。因此，如果像上次一样仅破坏一个区块，读取时可能会正好绕过被破坏的区块，访问正常的磁盘。因此，这次将写入几十 MB 的数据，以制造更大范围的错误。刚好内核有将近 30MB，就用它来破坏数据。

```sh
$ zpool export ztest
$ ls -l /boot/kernel/kernel
-r-xr-xr-x  2 root  wheel  29343392 Nov  4 10:27 /boot/kernel/kernel
$ dd if=/boot/kernel/kernel of=/dev/gpt/tt1 oseek=1000000
dd: /dev/gpt/tt1: Invalid argument
57311+1 records in
57311+0 records out
29343232 bytes transferred in 21.647581 secs (1355497 bytes/sec)
$
```

## 读取被破坏的数据部分

为了确认是否发生错误，仅使用 da1 这一侧进行导入。因为是 USB 接口的 HDD，把 da2 实体移除是最简单的方法。

[![DSCF2555.JPG](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F130638%2Fbd2d7cd7-fc2f-2fd1-4389-d554d26c438d.jpeg?ixlib=rb-4.0.0\&auto=format\&gif-q=60\&q=75\&s=ba0b5066b66b10ea7f44067b1ef43c30)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F130638%2Fbd2d7cd7-fc2f-2fd1-4389-d554d26c438d.jpeg?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=ba0b5066b66b10ea7f44067b1ef43c30)

```sh
$ zpool import
   pool: ztest
     id: 6304409293823647838
  state: DEGRADED
status: One or more devices are missing from the system.
 action: The pool can be imported despite missing or damaged devices.  The
        fault tolerance of the pool may be compromised if imported.
   see: https://openzfs.github.io/openzfs-docs/msg/ZFS-8000-2Q
 config:

        ztest        DEGRADED
          mirror-0   DEGRADED
            gpt/tt1  ONLINE
            gpt/tt2  UNAVAIL  cannot open
```

如上所示，无法访问 da2 侧的 `gpt/tt2`。就这样将 ztest 导入。

```sh
$ zpool import ztest
$ zpool status ztest
  pool: ztest
 state: DEGRADED
status: One or more devices could not be opened.  Sufficient replicas exist for
        the pool to continue functioning in a degraded state.
action: Attach the missing device and online it using 'zpool online'.
   see: https://openzfs.github.io/openzfs-docs/msg/ZFS-8000-2Q
config:

        NAME                      STATE     READ WRITE CKSUM
        ztest                     DEGRADED     0     0     0
          mirror-0                DEGRADED     0     0     0
            gpt/tt1               ONLINE       0     0     0
            13953643250226400519  UNAVAIL      0     0     0  was /dev/gpt/tt2

errors: No known data errors

$ df /ztest
Filesystem 512-blocks    Used Avail Capacity  Mounted on
ztest         7601264 7601024   240   100%    /ztest
$ ls /ztest
dummy
```

虽然只有一侧的驱动器，但 `/ztest/dummy` 确实存在。那么现在尝试实际读取它的内容。为了确认数据内容，这里使用 `hd` 命令进行读取。

```
$ hd /ztest/dummy
00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
*
hd: /ztest/dummy: Input/output error
9e1a0000
$
```

无法读取文件，发生了错误。

## 镜像结构下的读取

现在重新恢复镜像结构，再次进行同样的操作。

```sh
$ zpool export ztest
```

此时，重新连接之前拔下的 da2 侧 USB 硬盘。

```sh
$ zpool import
   pool: ztest
     id: 6304409293823647838
  state: ONLINE
 action: The pool can be imported using its name or numeric identifier.
 config:

        ztest        ONLINE
          mirror-0   ONLINE
            gpt/tt1  ONLINE
            gpt/tt2  ONLINE
```

可以看到，`gpt/tt2` 一侧的状态已经是 ONLINE。那么就将 ztest 导入。

```sh
$ zpool import ztest
$ zpool status ztest
  pool: ztest
 state: ONLINE
config:

        NAME         STATE     READ WRITE CKSUM
        ztest        ONLINE       0     0     0
          mirror-0   ONLINE       0     0     0
            gpt/tt1  ONLINE       0     0     0
            gpt/tt2  ONLINE       0     0     0

errors: No known data errors
$
```

成功地导入了 ztest。接下来就像之前一样，尝试读取 `/ztest/dummy` 文件。

```sh
$ hd /ztest/dummy
00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
*
e7ee0000
$
```

正如所见，文件已被正常读取，并且可以确认其内容全部为 0。

接着确认一下 ztest 的状态，可以看到一切正常，没有问题。

```sh
$ zpool status ztest
  pool: ztest
 state: ONLINE
  scan: resilvered 432K in 00:00:01 with 0 errors on Fri Dec 23 11:14:23 2022
config:

        NAME         STATE     READ WRITE CKSUM
        ztest        ONLINE       0     0     0
          mirror-0   ONLINE       0     0     0
            gpt/tt1  ONLINE       0     0     0
            gpt/tt2  ONLINE       0     0     0

errors: No known data errors
$
```

# 通过 scrub 检查磁盘一致性

ZFS 理论上应该存在无法读取的损坏块，因此执行了 `scrub` 操作来进行一致性检查。

```sh
$ zpool scrub ztest
$ zpool status ztest
  pool: ztest
 state: ONLINE
  scan: scrub in progress since Fri Dec 23 11:18:02 2022
        3.62G scanned at 530M/s, 378M issued at 54.0M/s, 3.62G total
        0B repaired, 10.18% done, 00:01:01 to go
config:

        NAME         STATE     READ WRITE CKSUM
        ztest        ONLINE       0     0     0
          mirror-0   ONLINE       0     0     0
            gpt/tt1  ONLINE       0     0     0
            gpt/tt2  ONLINE       0     0     0

errors: No known data errors
```

经过一段时间后，确认 scrub 完成后的状态如下：

```sh
$ zpool status ztest
  pool: ztest
 state: ONLINE
status: One or more devices has experienced an unrecoverable error.  An
        attempt was made to correct the error.  Applications are unaffected.
action: Determine if the device needs to be replaced, and clear the errors
        using 'zpool clear' or replace the device with 'zpool replace'.
   see: https://openzfs.github.io/openzfs-docs/msg/ZFS-8000-9P
  scan: scrub repaired 15.8M in 00:01:06 with 0 errors on Fri Dec 23 11:19:08 2022
config:

        NAME         STATE     READ WRITE CKSUM
        ztest        ONLINE       0     0     0
          mirror-0   ONLINE       0     0     0
            gpt/tt1  ONLINE       0     0   130
            gpt/tt2  ONLINE       0     0     0

errors: No known data errors
```

从该结果可以看出，`gpt/tt1`（da1 一侧）发生了 130 次校验和错误。但由于镜像（mirror）结构，从健康的 `gpt/tt2` 一侧获取了正确的数据，并在 scrub 过程中修复了损坏的数据。


## 重新读取数据

为了排除操作系统磁盘缓存的影响，先将池导出再重新导入，并重新进行读取操作：

```sh
$ zpool export ztest
$ zpool import ztest
$ hd /ztest/dummy
00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
*
e7ee0000
$
```

如上所示，尽管存在发生错误的磁盘，但由于镜像结构的作用，依然成功读取到了正确的数据。

```sh
$ zpool create -O atime=off ztest mirror gpt/tt1 gpt/tt2
$ dd if=/dev/zero of=/ztest/dummy bs=1m
dd: /ztest/dummy: No space left on device
3711+0 records in
3710+1 records out
3891134464 bytes transferred in 153.170356 secs (25403966 bytes/sec)
$ zpool export ztest
$ dd if=/boot/kernel/kernel of=/dev/gpt/tt1 oseek=1000000
dd: /dev/gpt/tt1: Invalid argument
57311+1 records in
57311+0 records out
29343232 bytes transferred in 21.643263 secs (1355767 bytes/sec)
$ zpool import ztest
$ zpool scrub ztest
$ zpool status ztest
  pool: ztest
 state: ONLINE
status: One or more devices has experienced an unrecoverable error.  An
        attempt was made to correct the error.  Applications are unaffected.
action: Determine if the device needs to be replaced, and clear the errors
        using 'zpool clear' or replace the device with 'zpool replace'.
   see: https://openzfs.github.io/openzfs-docs/msg/ZFS-8000-9P
  scan: scrub repaired 27.9M in 00:01:06 with 0 errors on Mon Dec 26 10:55:49 2022
config:

        NAME         STATE     READ WRITE CKSUM
        ztest        ONLINE       0     0     0
          mirror-0   ONLINE       0     0     0
            gpt/tt1  ONLINE       0     0   227
            gpt/tt2  ONLINE       0     0     0

errors: No known data errors
$ zpool export ztest
```

已经成功导出，因此现在物理上移除 da2 一侧，只使用 da1 一侧进行导入并尝试读取数据。

```sh
$ zpool import
   pool: ztest
     id: 11275383091719095959
  state: DEGRADED
status: One or more devices are missing from the system.
 action: The pool can be imported despite missing or damaged devices.  The
        fault tolerance of the pool may be compromised if imported.
   see: https://openzfs.github.io/openzfs-docs/msg/ZFS-8000-2Q
 config:

        ztest        DEGRADED
          mirror-0   DEGRADED
            gpt/tt1  ONLINE
            gpt/tt2  UNAVAIL  cannot open
$ zpool import ztest
$ hd /ztest/dummy
00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
*
e7ee0000
$ zpool export ztest
```

成功读取了损坏侧 da1 上的所有数据，完全没有问题。也就是说，通过 scrub 操作，数据被写回并得到了修复。而且实际在导入之后通过 `status` 确认，CKSUM 的值也确实是 0（可惜没来得及截图……）。

接下来，再次为已经导出的 ztest 连接上 da2，并尝试导入。

```sh
$ zpool import ztest
$ zpool status ztest
  pool: ztest
 state: ONLINE
  scan: scrub repaired 27.9M in 00:01:06 with 0 errors on Mon Dec 26 10:55:49 2022
config:

        NAME         STATE     READ WRITE CKSUM
        ztest        ONLINE       0     0     0
          mirror-0   ONLINE       0     0     0
            gpt/tt1  ONLINE       0     0     0
            gpt/tt2  ONLINE       0     0     0

errors: No known data errors
$
```

如上所示，错误已经被清除。

## 总结

通过本次实验可以确认，在 ZFS 中将磁盘配置为镜像或 RAID-Z 结构，可以避免部分数据错误的影响。同时也验证了，对于可以修复的数据，执行 scrub 后能够恢复为原来的状态。

不仅仅是 ZFS，其他的卷管理器或 RAID 系统，如果正确配置了镜像或 RAID，也同样能够避免部分数据错误。而是否能够实现自动修复，则取决于各个系统的具体实现。
