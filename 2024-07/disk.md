# 通过替换 ZFS 镜像池中的磁盘进行扩容

- 原文链接：[ZFSミラープールのディスク交換による容量増設](https://qiita.com/belgianbeer/items/8df197588462cd7f6b45)
- 作者：みんみん
- 上次更新于 2023 年 07 月 16 日

## 引言

自家服务器的硬盘使用量已经快超过 80%[1]，同时硬盘也已经使用了 10 多年。出于安全考虑，决定更换硬盘。因此，我购买了新的硬盘并进行了更换。本文记录了实际更换硬盘的过程。

值得一提的是，笔者的服务器操作系统是 FreeBSD，但如果是在 Linux 上使用 ZFS，相关操作也是类似的（笔者也在 Linux 上使用 ZFS）。

## 服务器存储结构

目标服务器的操作系统为 FreeBSD，存储结构如下：用于系统的固态硬盘（256GB）1 块，用于数据存储的硬盘（2TB）2 块，所有硬盘均使用 ZFS 进行配置。这两块硬盘通过 ZFS 配置为镜像（mirror）方式，意味着即使有两块硬盘，容量仅相当于一块硬盘的容量。此次操作将把这两块 2TB 的硬盘分别更换为 6TB 的硬盘。

## 更换前的检查

在实际进行更换操作之前，我们先检查当前的状态。通过执行 `zpool list` 命令，查看硬盘的使用情况，输出如下：

```sh
$ zpool list
NAME    SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP  HEALTH  ALTROOT
zroot   228G  16.8G   211G        -         -    12%     7%  1.00x  ONLINE  -
zvol0  1.81T  1.49T   333G        -         -     5%    82%  1.00x  ONLINE  -
$
```

zroot 是系统用的 SSD，而 zvol0 是这次更换的硬盘，ZFS 镜像结构的使用量为 82%。

可以通过 `zpool list -v` 或 `zpool status` 命令查看更详细的配置：

```sh
$ zpool list -v zvol0
NAME             SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  
zvol0           1.81T  1.49T   333G        -         -     5%    82%  1.00x    ONLINE  -
  mirror-0      1.81T  1.49T   333G        -         -     5%  82.1%      -    ONLINE
    gpt/ndisk2      -      -      -        -         -      -      -      -    ONLINE
    gpt/ndisk1      -      -      -        -         -      -      -      -    ONLINE
$
```

```sh
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

从中可以看到，ndisk1 和 ndisk2 是组成镜像的硬盘分区。三块存储设备分别为 ada0（SSD）、ada1（硬盘）和 ada2（硬盘）。硬盘的分区信息如下：

```sh
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

从中可以看出，ada1 和 ada2 均使用 GPT 分区，ada1 的 ZFS 用分区被命名为 ndisk1，ada2 的则为 ndisk2。

## 镜像结构下的硬盘更换步骤

对于 ZFS 或其他 RAID 1 镜像（Mirror）结构的硬盘，更换两块硬盘的步骤大致有两种方法：

* 使用新的硬盘构建镜像，然后从现有镜像中复制数据，复制完成后再更换硬盘。或者先更换硬盘，再从原硬盘复制数据。
* 更换其中一块硬盘，利用镜像功能同步数据，完成后再更换另一块硬盘并重新同步数据。

如果采用第一种方法，数据复制仅需进行一次，但需要同时连接 3 或 4 台硬盘，这可能会受到 SATA 接口数量等限制。虽然硬盘可以通过 USB 连接，但如果使用 USB 2.0 接口，传输速度较慢，复制时间较长，因此建议使用 USB 3.0 接口。

笔者选择了第二种方法，即逐个更换硬盘。这样，即便在数据同步期间（虽然会有性能下降），服务器仍然可以正常使用，这是一大优势。

## 更换硬盘与数据同步

### ZFS 镜像中硬盘更换时的注意事项

在更换 ZFS 镜像中的硬盘时，有一点非常重要：**切勿在更换前执行 ZFS 池的操作，如 `zpool detach zvol0 gpt/ndisk2` 等**。因为如果使用 `zpool detach` 将 ndisk2 从池中分离，ndisk2 上的数据会被视为已删除。若在数据同步过程中，ndisk1 读取出现错误，就无法使用新硬盘来重新同步数据了[2]。如果只是切断电源并拆下 ndisk2，ndisk2 上的数据仍然会与 ndisk1 相同，因此万一出现问题时，可以使用 ndisk2 来进行恢复。

### 更换第一块硬盘

首先更换第一块硬盘，即拆下包含 ndisk2 分区的硬盘。关机并切断电源后，取下旧硬盘并连接新硬盘，然后重新开机。

启动后，检查 zvol0 的状态：

```sh
$ zpool list
NAME    SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT
zroot   228G  16.8G   211G        -         -    12%     7%  1.00x    ONLINE  -
zvol0  1.81T  1.49T   333G        -         -     5%    82%  1.00x  DEGRADED  -
$
```

```sh
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
$
```

此时，虽然硬盘是镜像配置，但由于一侧的 gpt/ndisk2 缺失，状态变为 DEGRADED（降级）。在 `zpool status` 的输出中，`config:` 部分显示 gpt/ndisk2 不再出现，而是用一个 ID（`12897545936258916783`）代替，并且行尾标记为 "was /dev/gpt/ndisk2"。这个 ID 后续将会用到。

### 创建交换分区

为了准备更换用的硬盘，我们首先需要对其进行 GPT 初始化，并为 ZFS 创建一个分区。

首先，我们使用 GPT 初始化硬盘：

```sh
$ gpart show ada1
gpart: No such geom: ada1.
$ gpart create -s gpt ada1
ada1 created
$ gpart show ada1
=>         40  11721045088  ada1  GPT  (5.5T)
           40  11721045088        - free -  (5.5T)

$
```

接下来，我们为 ZFS 创建一个分区。为了便于区分，我们将新分区命名为 `sdisk1`。

```sh
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

### 同步第一台硬盘的数据

新分区创建完成后，接下来用 `sdisk1` 替换旧的 `ndisk2`。我们使用 `zpool replace` 命令来完成此操作。

通常，`zpool replace` 命令的第一个参数是要替换的设备，而第二个参数是新的设备。在本例中，由于硬盘已经被移除，所以原来的设备已经不存在了，因此我们需要指定 `zpool status zvol0` 中显示的 ID。

```sh
$ zpool replace zvol0 12897545936258916783 gpt/sdisk1
$
```

此时，`ndisk1` 的镜像同步将在 `sdisk1` 上开始。

同步过程将在后台进行，并且正如之前提到的，在此期间，服务器可以正常使用。由于数据量大约为 1.5TB，所需的同步时间会比较长。不过，ZFS 可以通过 `zpool status` 来查看同步的进度，这样我们就能得到一个结束时间的大致估算。

```sh
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

在 `zpool status` 中，您会看到 `One or more devices is currently being resilvered`[3]，这表明正在进行 resilver 操作（即同步）。在 `action` 部分，系统会指示您等待 resilver 操作完成。同时，`scan` 部分显示了处理的进度和预计完成时间，但在同步开始时，通常会看到 `no estimated completion time`，这意味着系统无法立即估算预计时间。

稍等片刻，预计时间会逐渐显示，但根据经验，这个估算并不总是准确的。

例如，刚开始时，您可能会看到类似下面的输出：

```sh
$ zpool status zvol0
===== <省略> =====
  scan: resilver in progress since Sat Jan 28 14:25:44 2023
        265G scanned at 664M/s, 6.51G issued at 16.3M/s, 1.49T total
        6.51G resilvered, 0.43% done, 1 days 02:27:43 to go
===== <省略> =====
$
```

过了一段时间，预计完成时间可能会变为：

```sh
$ zpool status zvol0
===== <省略> =====
  scan: resilver in progress since Sat Jan 28 14:25:44 2023
        289G scanned at 488M/s, 18.6G issued at 31.4M/s, 1.49T total
        18.6G resilvered, 1.22% done, 13:38:30 to go
===== <省略> =====
$
```

一小时后，您可能会看到预估的时间减少到 6 小时左右，但最终同步的实际时间可能比预估的要长。最终，第一台硬盘的同步总共花费了大约 15 小时半。

以下是第一台硬盘同步完成时，检查 `zpool list` 和 `zpool status` 的结果：

```sh
$ zpool list -v zvol0
NAME             SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT
zvol0           1.81T  1.49T   333G        -         -     5%    82%  1.00x    ONLINE  -
  mirror-0      1.81T  1.49T   333G        -         -     5%  82.1%      -    ONLINE
    gpt/sdisk1      -      -      -        -         -      -      -      -    ONLINE
    gpt/ndisk1      -      -      -        -         -      -      -      -    ONLINE
$
```

```sh
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

HEALTH 和 STATE 都显示为 ONLINE，且没有出现任何错误，可以确认第一台硬盘的同步已经顺利完成。

### 第二台硬盘的同步

第二台硬盘的更换和同步步骤与第一台完全相同，因此这里只展示相关命令的执行和结果。

更换第二台硬盘并启动后。

```sh
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

准备用于镜像的分区

```sh
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

第二台同步开始

```sh
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

第二台的同步大约用了14个小时。

```sh
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

## 容量的扩展

硬盘交换和数据同步顺利完成，但仍有一些工作需要完成。因为目前仅仅是更换了硬盘，并没有启用 2TB → 6TB 的容量扩展。

重新检查 `zvol0` 的容量时，发现 SIZE 仍为 1.81T，与更换前相同。然而，EXPANDSZ 显示为 3.62T，这意味着还可以扩展 3.62TB 的空间。

```sh
$ zpool list zvol0
NAME    SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT
zvol0  1.81T  1.49T   333G        -     3.62T     5%    82%  1.00x    ONLINE  -
$
```

这个容量扩展是通过设置 ZFS 池的 `autoexpand` 属性来实现的。首先，检查当前 `autoexpand` 的值，发现默认值为 `off`。

```sh
$ zpool get autoexpand zvol0
NAME   PROPERTY    VALUE   SOURCE
zvol0  autoexpand  off     default
$
```

将 `autoexpand` 设置为 `on`。

```sh
$ zpool set autoexpand=on zvol0
$ zpool get autoexpand zvol0
NAME   PROPERTY    VALUE   SOURCE
zvol0  autoexpand  on      local
$
```

但是，仅将 `autoexpand` 设置为 `on` 并不会改变容量。

```sh
$ zpool list zvol0
NAME    SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT
zvol0  1.81T  1.49T   333G        -     3.62T     5%    82%  1.00x    ONLINE  -
$
```

要扩展容量，需要在设置了 `autoexpand=on` 之后，执行 `zpool online -e`。

```sh
$ zpool online -e zvol0
missing device name
usage:
        online [-e] <pool> <device> ...
$
```

忘记指定设备名导致了错误 😅，但实际上，使用 `zpool online` 时必须指定设备名，而当使用 `-e` 选项时，应该指定池内某个设备名才可以执行。

重新指定设备名并执行后，容量成功扩展到 5.45TB[4]。

```sh
$ zpool online -e zvol0 gpt/sdisk1
$ zpool list zvol0
NAME    SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT
zvol0  5.45T  1.49T  3.97T        -         -     1%    27%  1.00x    ONLINE  -
$
```

这次是硬盘更换后才修改 `autoexpand` 属性，如果在更换前就设置，容量会在第二台硬盘同步完成时自动扩展。同时，`autoexpand` 不仅适用于镜像，还可以在 RAIDZ 等结构中使用。

**2023-07-16 补充**：通过 `zpool online -e` 扩展时，其实不需要设置 `autoexpand` 属性。`autoexpand` 属性是为了自动扩展容量而设计的。

## 交换完成后

由于新硬盘的传输速度提高，因此整体性能得到了提升。尽管 CPU 已经使用了 10 多年，但由于主要用途是文件服务器，仍然可以继续使用一段时间。

---

1. 在 ZFS 中，由于采用了 Copy on Write 数据更新机制，当空闲空间不足时，性能通常会比其他文件系统低得多（在硬盘上尤为明显）。因此，将容量使用到极限并不明智。关于多少空间使用会导致性能下降，这取决于工作集，因此无法简单地确定。但即使是其他文件系统认为影响不大的空闲空间，在 ZFS 中也可能导致性能下降。
2. 即使是卸载（detach）了 `ndisk2`，也可以通过 `zpool import -D` 来恢复数据，但我并没有试过。
3. 我是在使用 ZFS 后才知道“resilver”这个词，原本它是指磨光银饰品，在 ZFS 中则表示 RAID 恢复中。
4. 硬盘的 TB 是按十进制容量表示的，因此 6TB 是 6,000,000,000,000 字节，而二进制的 1TB 为 1,099,511,627,776 字节，所以实际容量为 5.45TB。
