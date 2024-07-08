# 通过为 ZFS 池配置镜像来消除无效数据的影响

* [Linux](https://qiita.com/tags/linux)
* [FreeBSD](https://qiita.com/tags/freebsd)
* [ZFS](https://qiita.com/tags/zfs)

上次更新于 2022-12-26 发布于 2022-12-23

# 开始

在之前发布的“尝试破坏 ZFS 池”的文章中，我们发现 ZFS 能够正确检测到磁盘上的非法数据。

能够检测到非法数据要比读取无法检测到非法数据的文件系统并导致错误操作要好得多。但是如果能够检测到非法数据，那么是否无法恢复它呢？这种想法也是很自然的。

如果配置冗余池，则即使存在非法数据，也可以获得正确的数据。具体来说，使用镜像、RAID-Z、RAID-Z2 等池。

在这里，将尝试使用镜像配置的 ZFS 池进行类似于破坏 ZFS 池的实验。

**在 2022-12-26 进行 scrub 后，补充了磁盘状态。**

# 准备 ZFS 镜像池

为实验准备两台 USB 连接的硬盘，分别创建一个 4GB 的分区，并准备一个镜像配置的 ZFS 池。

![DSCF2553.JPG](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F130638%2F789a1ff3-ea8a-ab11-3f7b-f3db1c5364cb.jpeg?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=5c42c540ecb2ec6b7b329fb3bb1e565b)

硬盘为/dev/da1，/dev/da2，在 GPT 下创建分区，并分别创建标记为 tt1 和 tt2 的分区。

```
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

这样，两个 HDD 上的每一个都创建了一个用于镜像的分区。

将这两个分区组合起来创建一个 ZFS 镜像池。当然，就像上次实验一样，不会进行 ZFS 压缩。

```
$ zpool create -O atime=off ztest mirror gpt/tt1 gpt/tt2
$
```

如下所示，已经完成了镜像配置的 ZFS 池。

```
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

# 写入虚拟数据

现在使用 dd 命令以零数据填充文件系统，就像上次的实验一样。

```
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

# 破坏写入的数据

然后强行写入垃圾数据，故意制造 ZFS 错误。这是为了验证镜像冗余和 ZFS 修复功能，因此仅在 da1 侧写入垃圾数据。

通常从镜像配置的磁盘读取时，为了负载平衡和加速，会交替访问两个磁盘。在 ZFS 镜像配置下也会有类似的行为。因此，与上次一样破坏一个块是无法访问到该块的正常驱动器侧的，而是会访问到另一个正常的驱动器侧。因此，这次我们将写入约数十 MB 的数据，更大的区域数据将会引起错误。正好内核接近 30MB，我们将写入它并试图破坏它。

```
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

# 读取损坏数据部分

为了确认是否发生错误，仅使用 da1 一侧进行导入。由于这是 USB 硬盘，因此物理上拔掉 da2 会更容易。

![DSCF2555.JPG](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F130638%2Fbd2d7cd7-fc2f-2fd1-4389-d554d26c438d.jpeg?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=ba0b5066b66b10ea7f44067b1ef43c30)

```
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

无法访问 da2 侧的 gpt/tt2。将继续导入 ztest。

```
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
#
```

只有一个驱动器，但/ztest/dummy 确实存在。那么让我们实际读取它。为了使数据内容可见，我们将使用 hd 命令进行读取。

```
$ hd /ztest/dummy
00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
*
hd: /ztest/dummy: Input/output error
9e1a0000
$
```

无法读取文件，发生了错误。

# 镜像配置下的读取

尝试重新配置为镜像并进行相同操作。

```
$ zpool export ztest
```

在这里连接已拆下的 DA2 端的 USB HDD。

```
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

GPT / TT2 端目前处于在线状态。现在将导入 ztest。

```
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

导入成功，现在让我们像刚才一样读取/ztest/dummy。

```
$ hd /ztest/dummy
00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
*
e7ee0000
$
```

能够正常读取，并确认所有内容都为 0。检查 ztest 的状态后，似乎没有问题。

```
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

# 使用 scrub 检查磁盘完整性

因为 ZFS 可能存在无法读取的部分，所以执行 scrub 以确认完整性。

```
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
$
```

等待 scrub 完成后，再查看状态。

```
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
$
```

可以看到 da1 侧的 tt1 出现了大量的校验和错误。

# 再次读取数据

最后再次尝试读取数据。为了排除操作系统磁盘缓存的影响，先导出再导入后再执行。

```
$ zpool export ztest
$ zpool import ztest
$ hd /ztest/dummy
00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
*
e7ee0000
$
```

尽管发生了错误，但由于镜像配置的帮助，成功读取了正确的数据。

# 【补充】执行 scrub 后的 da1 状态

文章发布后，我们收到了一个问题：“通过 scrub，da1 侧是否被修复并可读取？”实际上，scrub 执行后显示 scan: scrub repaired ，因此我进行了确认。

尽管实验环境已被销毁，我会重新进行部分内容的省略，破坏 da1 侧数据并进行 scrub，然后尝试读取 da1 侧数据。

从创建 ZFS 镜像池到破坏 da1 侧数据，执行 scrub，然后将池导出。

```
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

由于导出成功，我将物理地移除 da2 端口，尝试导入并读取 da1 端口。

```
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

我成功无误地读取了被破坏的 da1 端口的全部数据。也就是说，通过 Scrub 写回并修复了数据。而且实际导入后检查 status，CKSUM 的值也是 0(截屏失败…)。

再次尝试，将 da2 连接到导出的 ztest 并进行导入。

```
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

 这样错误已被清除。

# 汇总

通过在 ZFS 中将磁盘配置为镜像或 RAID-Z，我们确认可以避免部分数据错误。此外，通过执行 scrub 可以修复数据，我们也发现修复后的数据可以恢复到原始状态。

无论是 ZFS 还是其他卷管理器或 RAID 系统，正确地配置镜像或 RAID 可帮助规避部分数据错误。修复能力可能取决于具体实现。
