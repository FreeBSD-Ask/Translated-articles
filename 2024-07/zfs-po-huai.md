# ZFS 池破坏实验

- 原文：[ZFS プールを壊してみた](https://qiita.com/belgianbeer/items/477de8ddc64787442c0b)
- 作者：みんみん
- 最后更新于 2022-12-08

## 引言

ZFS 文件系统本身就内置了校验和机制，在读取数据时会实时检查校验和与数据的完整性，确保数据没有损坏。

执行命令 `zpool scrub` 后，会读取所有已写入的数据部分并与校验和进行一致性检查。如果在 scrub 过程中发现了错误，在具有冗余结构（如镜像或 RAID-Z 等）的情况下，错误会被自动修复。

在这里，我们将实验性地修改 ZFS 管理范围外的数据，故意制造校验和不一致的情况，以引发数据错误。

本实验在 FreeBSD 上进行，但如果是在 Ubuntu 等 Linux 系统上的 ZFS，虽然分区创建和磁盘设备名称可能会有所不同，但 ZFS 操作的命令是相同的。


## 为实验磁盘创建分区

首先，创建实验用的 ZFS 池。我们将在 USB 磁盘 da1 上进行实验。

准备一块带 GPT 标签的磁盘。

```sh
$ gpart create -s gpt da1
```

创建一个 4GB 的分区。

```sh
$ gpart add -t freebsd-zfs -a 4k -s 4g -l ttt da1
```

检查创建的分区。

```sh
$ gpart show da1
=>       40  312581728  da1  GPT  (149G)
         40    8388608    1  freebsd-zfs  (4.0G)
    8388648  304193120       - free -  (145G)

$ gpart show -l da1
=>       40  312581728  da1  GPT  (149G)
         40    8388608    1  ttt  (4.0G)
    8388648  304193120       - free -  (145G)
```

## 创建 ZFS 池

通常情况下，为了节省磁盘空间，当创建 ZFS 池时会使用 `-O compression=lz4` 等选项启用压缩功能，但本次实验为了写入零数据而没有启用压缩选项。因为启用压缩后，数据压缩会导致数据大小无法被实际写入。

```sh
$ zpool create -O atime=off ztest gpt/ttt
```

检查创建的 ZFS 池。

```sh
$ zpool status ztest
  pool: ztest
 state: ONLINE
config:

        NAME        STATE     READ WRITE CKSUM
        ztest       ONLINE       0     0     0
          gpt/ttt   ONLINE       0     0     0

errors: No known data errors
```

## 写入虚拟数据

Scrub 只会检查已经写入的数据部分的完整性。为了破坏已写入的数据，我们将首先创建一个内容为零的文件。

进入 ZFS 池内的目录。

```sh
$ cd /ztest
```

使用 `dd` 命令从 `/dev/zero` 读取虚拟数据（即零数据）并填充整个池。

```sh
$ dd if=/dev/zero of=dummy bs=4m
dd: dummy: No space left on device
928+0 records in
927+1 records out
3891134464 bytes transferred in 157.300331 secs (24736976 bytes/sec)
$
```

可以看到，池已被填满。此处我们显式使用 `-b` 选项以便查看块数。

```sh
$ df -b /ztest
Filesystem 512-blocks    Used Avail Capacity  Mounted on
ztest         7601024 7601024     0   100%    /ztest
```

## 故意破坏数据以制造不一致

池已填满后，我们将故意制造 ZFS 校验和错误。由于通过 ZFS 进行的修改无法破坏数据，我们将从 ZFS 管理外部向磁盘写入垃圾数据。

首先导出池。

```sh
$ zpool export ztest
```

接着在相关分区的适当位置写入一个块的垃圾数据。

```sh
$ dd if=/dev/random of=/dev/gpt/ttt count=1 oseek=4000000
1+0 records in
1+0 records out
512 bytes transferred in 0.382043 secs (1340 bytes/sec)
$
```


## 检查是否损坏

导入池并尝试读取之前创建的文件

```sh
$ zpool import ztest
$ cat /ztest/dummy > /dev/null
cat: /ztest/dummy: Input/output error
$
```

如上所示，发生了错误。池的状态如下，乍一看似乎没有错误。

```sh
$ zpool status ztest
  pool: ztest
 state: ONLINE
config:

        NAME        STATE     READ WRITE CKSUM
        ztest       ONLINE       0     0     0
          gpt/ttt   ONLINE       0     0     0

errors: No known data errors
$
```

尝试执行 scrub。

```sh
$ zpool scrub ztest
$ zpool status ztest
  pool: ztest
 state: ONLINE
  scan: scrub in progress since Mon Aug 29 18:17:33 2022
        3.62G scanned at 412M/s, 550M issued at 61.1M/s, 3.62G total
        0B repaired, 14.81% done, 00:00:51 to go
config:

        NAME        STATE     READ WRITE CKSUM
        ztest       ONLINE       0     0     0
          gpt/ttt   ONLINE       0     0     0

errors: No known data errors
$ 
```

等待 scrub 完成后，使用 status 进行确认。

```sh
$ zpool status ztest
  pool: ztest
 state: ONLINE
status: One or more devices has experienced an error resulting in data
        corruption.  Applications may be affected.
action: Restore the file in question if possible.  Otherwise restore the
        entire pool from backup.
   see: https://openzfs.github.io/openzfs-docs/msg/ZFS-8000-8A
  scan: scrub repaired 0B in 00:01:05 with 1 errors on Mon Aug 29 18:18:32 2022
config:

        NAME        STATE     READ WRITE CKSUM
        ztest       ONLINE       0     0     0
          gpt/ttt   ONLINE       0     0     2

errors: 1 data errors, use '-v' for a list
$
```

可以看到，CKSUM 发生了错误。实际损坏的文件可以通过选项 `-v` 进行确认。

```sh
$ zpool status -v ztest
  pool: ztest
 state: ONLINE
status: One or more devices has experienced an error resulting in data
        corruption.  Applications may be affected.
action: Restore the file in question if possible.  Otherwise restore the
        entire pool from backup.
   see: https://openzfs.github.io/openzfs-docs/msg/ZFS-8000-8A
  scan: scrub repaired 0B in 00:01:05 with 1 errors on Mon Aug 29 18:18:32 2022
config:

        NAME        STATE     READ WRITE CKSUM
        ztest       ONLINE       0     0     0
          gpt/ttt   ONLINE       0     0     2

errors: Permanent errors have been detected in the following files:

        /ztest/dummy
$
```


重新读取 `/ztest/dummy` 时，错误依旧存在。

```sh
$ cat /ztest/dummy > /dev/null
cat: /ztest/dummy: Input/output error
$
```

## 修复损坏部分并确认

接下来，通过重新写 0 来恢复损坏的部分。

```sh
$ zpool export ztest
$ dd if=/dev/zero of=/dev/gpt/ttt count=1 oseek=4000000
1+0 records in
1+0 records out
512 bytes transferred in 0.382715 secs (1338 bytes/sec)
$
```

这应该已经将数据恢复到原状态，再次尝试读取。

```sh
$ zpool import ztest
$ cat /ztest/dummy > /dev/null
$
```

如上所示，读取成功且没有错误。虽然 `zpool status` 中的错误信息依然存在，但 CKSUM 的错误计数已经变为 0。

```sh
$ zpool status ztest
  pool: ztest
 state: ONLINE
status: One or more devices has experienced an error resulting in data
        corruption.  Applications may be affected.
action: Restore the file in question if possible.  Otherwise restore the
        entire pool from backup.
   see: https://openzfs.github.io/openzfs-docs/msg/ZFS-8000-8A
  scan: scrub repaired 0B in 00:01:05 with 1 errors on Mon Aug 29 18:18:32 2022
config:

        NAME        STATE     READ WRITE CKSUM
        ztest       ONLINE       0     0     0
          gpt/ttt   ONLINE       0     0     0

errors: 1 data errors, use '-v' for a list
$
```

重新执行 scrub

```sh
$ zpool scrub ztest
```

scrub 完成后，检查池的状态。

```sh
$ zpool status ztest
  pool: ztest
 state: ONLINE
  scan: scrub repaired 0B in 00:01:05 with 0 errors on Mon Aug 29 18:34:27 2022
config:

        NAME        STATE     READ WRITE CKSUM
        ztest       ONLINE       0     0     0
          gpt/ttt   ONLINE       0     0     0

errors: No known data errors
$
```

如上所示，错误已消失。
