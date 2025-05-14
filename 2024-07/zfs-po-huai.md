# ZFS 池破坏实验

- <https://qiita.com/belgianbeer/items/477de8ddc64787442c0b>

* [Linux](https://qiita.com/tags/linux)
* [FreeBSD](https://qiita.com/tags/freebsd)
* [ZFS](https://qiita.com/tags/zfs)

最后更新于 2022-12-08 发布于 2022-12-08

## 开始

ZFS 文件系统本身包含校验和，读取数据时会始终验证校验和，以确保数据没有损坏。

执行 zpool scrub 命令会读取写入的所有数据块，并验证校验和。如果在 scrub 过程中发现错误，并且在具有冗余的镜像或 RAID-Z 等配置下可以修复，那么错误将自动修复。

这里，我故意在 ZFS 管理之外的地方实验性地写入数据，以产生与检验和不一致后故意引发数据错误。

在这个实验中使用了 FreeBSD，但在类似 Ubuntu 等 Linux 系统上的 ZFS 中，分区创建和磁盘设备名称可能会发生变化，但 ZFS 操作相同命令仍然有效。

## 冷静地创建并破坏 ZFS 池


首先创建实验用池。在这里，我们将在 USB 磁盘 da1 上进行实验。

 准备一个带 GPT 标签的磁盘。

```
$ gpart create -s gpt da1
```

 创建一个 4GB 的分区。

```
$ gpart add -t freebsd-zfs -a 4k -s 4g -l ttt da1
```

 检查创建好的分区。

```
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

通常情况下，为了节省磁盘空间，创建 ZFS 池时通常要指定压缩功能，但这次不指定，因为将写入零数据。这样做是因为一旦指定了数据压缩功能，写入的数据将不再占用数据大小的空间。

```
$ zpool create -O atime=off ztest gpt/ttt
```

 检查已创建的 ZFS 池。

```
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

使用 scrub 命令来验证数据的一致性只针对实际写入数据的部分。为了破坏已写入的数据，预先创建内容为 0 的文件。

迁移到准备好的 ZFS 池内的目录。

```
$ cd /ztest
```

使用 dd 命令，从 /dev/zero 读取的虚拟数据（即零数据）填充整个池。

```
$ dd if=/dev/zero of=dummy bs=4m
dd: dummy: No space left on device
928+0 records in
927+1 records out
3891134464 bytes transferred in 157.300331 secs (24736976 bytes/sec)
$ 
```

这就像是用完整个游泳池一样。在这里，我明确指定了 -b 选项，以便检查块数。

```
$ df -b /ztest
Filesystem 512-blocks    Used Avail Capacity  Mounted on
ztest         7601024 7601024     0   100%    /ztest
```

## 让数据发生不一致性

在填满游泳池后，我们将破坏 ZFS 数据完整性，以便引发错误。无法通过 ZFS 进行更改以进行破坏，因此在这里，我们将尝试从 ZFS 外部向磁盘写入垃圾数据。

 一旦导出池

```
$ zpool export ztest
```

在相关分区的适当位置写入 1 块垃圾数据。

```
$ dd if=/dev/random of=/dev/gpt/ttt count=1 oseek=4000000
1+0 records in
1+0 records out
512 bytes transferred in 0.382043 secs (1340 bytes/sec)
$ 
```

## 检查是否损坏

导入池，尝试读取之前创建的文件

```
$ zpool import ztest
$ cat /ztest/dummy > /dev/null
cat: /ztest/dummy: Input/output error
$ 
```

出现了错误。池的状态如下，乍看之下似乎没有错误。

```
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

```
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

等待 scrub 完成，然后通过 status 进行确认。

```
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

通过 CKSUM 可以发现发生了错误。可以使用 -v 选项实际确认已损坏的文件。

```
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

再次尝试读取 /ztest/dummy，错误仍然存在。

```
$ cat /ztest/dummy > /dev/null
cat: /ztest/dummy: Input/output error
$
```

## 损坏部分的修复和确认

现在我们尝试将 0 重新写入损坏的部分，以恢复原始数据。

```
$ zpool export ztest
$ dd if=/dev/zero of=/dev/gpt/ttt count=1 oseek=4000000
1+0 records in
1+0 records out
512 bytes transferred in 0.382715 secs (1338 bytes/sec)
$
```

现在应该已经将数据恢复到原始状态，我们再次尝试读取它。

```
$ zpool import ztest
$ cat /ztest/dummy > /dev/null
$
```

无法读取时，没有错误。虽然 zpool status 显示错误消息，但 CKSUM 错误计数为 0。

```
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

 再次运行 scrub。

```
$ zpool scrub ztest
```

检查池状态。

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

 这样错误就消失了。
