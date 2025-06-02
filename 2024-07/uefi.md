# 使用 BIOS 引导和 UEFI 引导的 GPT 分区的区别和制作方法

- 原文：[BIOS ブートと UEFI ブートで GPT パーティションの 違 いと 作 り 方](https://qiita.com/nanorkyo/items/429d7382a418b38de4d3)
- 作者：重村法克
- 2020-11-11

在当前的 FreeBSD 中，不论磁盘大小，几乎都建议使用 GPT（GUID 分区表）来进行分区配置。然而，在传统的 BIOS 启动和 UEFI 启动的情况下，分区配置会有所不同 [1]。以下将介绍两者的区别，并提供可用的启动分区配置示例。

## 前提条件

在这里，我们假设操作的是 SATA 磁盘，设备名称表示为 `/dev/ada0`。如果是 SAS/USB 磁盘，设备名为 `dan`，NVMe 磁盘为 `nvdn`，eMMC 磁盘为 `mmcsdn`（其中 `n` 是大于等于 0 的整数）。

## 通用操作

由于使用的是 GPT 分区，在所有操作步骤中，都必须执行以下命令 [2]：

```sh
gpart create -s GPT ada0
```

为了简化讨论，假设我们分配了以下分区：

| 设备名    | 分区类型               | 用途         |
| ------ | ------------------ | ---------- |
| ada0p1 | freebsd-boot 或 efi | 启动分区       |
| ada0p2 | freebsd-ufs [3]        | FreeBSD 分区 |
| ada0p3 | freebsd-swap       | 交换分区       |

在本讨论中，`ada0p2` 和 `ada0p3` 使用相同的通用步骤。即执行以下命令，虽然这些命令在当前场景中可能并不立即有用 [4][5] [6]：

```sh
gpart add -t freebsd-ufs  ada0
gpart add -t freebsd-swap ada0
```

## 在 BIOS 启动情况下

```sh
gpart add -t freebsd-boot -b 40 -s 984  ada0
```

- 我们为启动分区定义了 freebsd-boot 类型。如果不是 freebsd-boot，引导加载器将无法发现该分区。
- 起始位置（`-b`）指定为从第 40 扇区（LBA 40）开始。

  - GPT 分区表头大小为 34 扇区，但在支持 4KiB 扇区的磁盘上，起始位置可能会被四舍五入为 8 的倍数。
  - 近期版本中，GPT 分区表头的大小已经增加为 40 扇区（具体从哪个版本开始的尚未确认）。
- 分区大小（`-s`）推荐为 984 扇区。

  - 这样可以确保下一分区（freebsd-ufs）从 512KiB 边界开始 [8]。
  - 由于引导加载器的限制，分区大小不得小于 545KiB，所以下面的大小为 492KiB。
  - 因此，调整分区大小没有必要，也没有更多的空间来增大它。

```sh
gpart bootcode -b /boot/pmbr -p /boot/gptboot    -i 1 ada0
        或者
gpart bootcode -b /boot/pmbr -p /boot/gptzfsboot -i 1 ada0
```

以上步骤完成后，引导加载器安装完毕了。引导加载器会查找 FreeBSD 分区 [9][10]，然后将控制权交给 `/boot/loader` 或 `/boot/zfsloader`。

## 在 UEFI 启动情况下

```sh
gpart add -t efi -b 40 -s 409560 ada0
```

- 我们为启动分区定义了 efi 类型。如果不是 efi，UEFI 将无法发现该分区。
- 起始位置（`-b`）指定为从第 40 扇区（LBA 40）开始。

  - GPT 分区表头的大小为 34 扇区 [7]，但在支持 4KiB 扇区的磁盘上，起始位置可能会被四舍五入为 8 的倍数。
  - 近期版本中，GPT 分区表头的大小已调整为 40 扇区（具体从哪个版本开始尚未确认）。
- 分区大小（`-s`）建议设置为 409560 扇区。

  - 这个设置确保下一个分区（freebsd-ufs）从 200MiB 边界开始。
  - 该分区的实际大小略小于 200MiB（少 40 扇区，即 20KB），但可以忽略不计。
  - 目前，11.2-R 的启动分区（`/boot/boot1.efifat`）的大小为 800KiB。
  - 在 10.4-R 或 11.1-R 版本中，推荐为该分区分配 200MiB，以考虑与其他操作系统（如 macOS）的兼容性。如果仅启动 FreeBSD，800KiB 的大小应无问题。
  - 在 [bsdinstall: 增加 EFI 分区大小至 200MB](https://reviews.freebsd.org/D6935) 的讨论中指出，为了支持固件更新工具等 EFI 应用程序，800KiB 的分区大小可能过小。

```sh
newfs_msdos -F 32 -c 1 -L EFISYS /dev/ada0p1
mount -t msdosfs /dev/ada0p1 /mnt
mkdir -p /mnt/EFI/BOOT
cp /boot/loader.efi /mnt/EFI/BOOT/BOOTX64.efi
umount /mnt
        或者
gpart bootcode -p /boot/boot1.efifat -i 1 ada0
```

完成上述步骤后，启动加载器将安装完毕。启动加载器会查找 FreeBSD 分区，并继续启动过程。

## 常见问题与解答

### 问：为什么与 [Bootable UEFI memory stick or Hard Disk](https://wiki.freebsd.org/UEFI#Bootable_UEFI_memory_stick_or_Hard_Disk) 上的步骤不同？

答：没问题，完全没问题。时代已经跟上了。

### 问：通过各种资料查看，似乎有很多方法来写入 `/boot/boot1.efifat` 分区，该采用哪种方式才正确？

答：没问题，任何方法都可以。但我不推荐任何方法。

`/boot/boot1.efifat` 实际上是分区镜像本身。极端地说，可以认为是 `dd if=/dev/ada0p1 of=/boot/boot1.efifat` 的结果。实际构建时也会做类似的操作。

因此，无论是使用 dd(8) 还是 gpart(8) 写入都没问题，但显然，gpart(8) 是更聪明的选择。

不过，最好避免使用 `/boot/boot1.efifat`。正如前面提到的，扩展到 200MiB 后，这个分区只能访问 800KiB 的空间。

尽管分配了 200MiB 的空间，但 EFI 应用程序无法在这 200MiB 中运行。

新安装不会有问题，但更新时，使用这种方式可能会不必要地删除一些内容。

### 问：在操作系统更新时，是否需要更新启动分区？

答：并非每次都需要，但有时需要，具体情况如下：

1. **从 UFS 启动** 时，由于 UFS 的格式稳定性，启动加载器代码通常不会发生变化，所以几乎不会出现问题。即使没有更新启动分区，通常也不会发现问题，除非从 UFS1 升级到 UFS2。
2. **从 ZFS 启动** 时，由于 ZFS 格式的不稳定性，启动加载器代码会发生变化。因此，必须定期维护启动分区，否则可能无法启动。有关这一点，`/usr/src/UPDATING` 中的 `ZFS notes` 提到：“如果更新启动的 ZFS 池的版本，请确保更新启动加载器。”

因此，除了更新步骤外，更新启动加载器的步骤也应纳入操作流程。

对于 freebsd-boot 分区，可以通过 `gpart bootcode -p` 命令覆盖，但对于 EFI 分区，需要先挂载，再通过 `cp` 命令进行覆盖（[后续说明](https://qiita.com/nanorkyo/items/429d7382a418b38de4d3#%E4%BB%98%E9%8C%B2%E3%83%96%E3%83%BC%E3%83%88%E3%83%AD%E3%83%BC%E3%83%80%E3%83%BC%E3%81%AE%E6%9B%B4%E6%96%B0)）。

### 问：如果分区号不是 1，那么是否可以随意设置？

答：不行，最好不要这样做。

由于 MBR 存在 2TiB 的限制，如果使用超过 2TiB 的空间，BIOS 可能无法找到启动加载器（freebsd-boot）。

另外，如果分区号和分区顺序不一致，管理起来会很麻烦。

### 问：我们使用的是 GPT 分区，为什么还要管 MBR？

答：遗憾的是，GPT 是 MBR 的上位兼容，所以不能完全忽略它。

对于 UEFI 启动，GPT 被视为 GPT，不用担心 MBR 的问题。

然而对于 BIOS 启动，GPT 会表现得像 MBR，因此需要在前 2TiB 内有启动加载器（freebsd-boot）。不过，是否使用分区号 1 并不重要。

### 问：我不确定系统是 UEFI 启动还是 BIOS 启动，怎么确定？

答：很简单！在安装 CD 上启动为 LiveCD 后，执行命令 `sysctl machdep.bootmethod`。如果是 UEFI 启动，显示的是 UEFI；如果是 BIOS 启动，则显示的是 BIOS。

## 【附录】启动加载器更新

### BIOS 启动情况下

```sh
gpart bootcode -p /boot/gptboot    -i 1 ada0
        或
gpart bootcode -p /boot/gptzfsboot -i 1 ada0
```

### UEFI 启动的情况

首先，在 `/etc/fstab` 中添加以下设置。

```sh
# Device    Mountpoint FStype  Options Dump Pass #
/dev/ada0p1 /boot/efi  msdosfs rw      0    0
```

接着，创建 `/boot/efi` 目录。

```sh
mkdir -p /boot/efi
```

首次可以手动执行挂载。当然，也可以重启后无需特别关注。

```sh
mount /boot/efi
```

之后的更新操作如下：

```sh
cp /boot/loader.efi /boot/efi/EFI/BOOT/BOOTX64.efi
```

### 参考文献

- [gpart(8) 的手册](https://www.freebsd.org/cgi/man.cgi?gpart%288%29)
- [FreeBSD 的启动过程](https://qiita.com/mzaki/items/76acac14c16ac6789e68)
- [UEFI - FreeBSD Wiki](https://wiki.freebsd.org/UEFI)
- [FreeBSD 10.4 版本说明](https://www.freebsd.org/releases/10.4R/relnotes.html)
- [FreeBSD 11.1 版本说明](https://www.freebsd.org/releases/11.1R/relnotes.html)
- [bsdinstall: 增加 EFI 分区大小至 200MB](https://reviews.freebsd.org/D6935)
- [GPT 和 MBR 的区别是什么？](http://syuu1228.hatenablog.com/entry/20130103/1357165915)
- [通过命令行创建 FreeBSD 启动分区等](http://nao550.hateblo.jp/entry/2018/02/12/214146)
- [FreeBSD/UEFI 相关信息](http://zenno.com/pukiwiki/index.php?FreeBSD%2FUEFI%A4%CB%A4%C4%A4%A4%A4%C6)

---

1. 在 BIOS 中，通常使用“MBR（主引导记录）引导”，但是由于 2 TiB 限制的严格性以及几乎没有多重引导需求，因此已经统一采用了 GPT 引导（？）
2. 执行此命令没有过多或不足之处。从兼容性的角度来看，根本没有修改默认值的余地。
3. 虽然有时可能希望将 freebsd-ufs 改为 freebsd-zfs，但在本次讨论中暂且省略这一点。
4. 实际上，需要使用 `-s 尺寸` 选项来指定大小。如果没有指定，意味着使用剩余的所有空间。也就是说，如果直接执行，将无法为交换空间分配空间。
5. 执行顺序需要注意。如果按此顺序执行，将无法分配启动分区（分区号会错位）。
6. 可以使用 `-i 分区号` 或 `-b 起始位置 -i 分区号` 选项来进行控制，但作为基本说明，建议不使用这些选项。
7. LBA 0 到 LBA 33 是 GPT 头区域。
8. 在现代各种介质（例如 4 KiB 扇区的 HDD 或 NAND 闪存等）中，**边界** 的最小公倍数（如果有 512 KiB 则适用于任何介质）是一个相当合理的数字（应该如此）。
9. gptboot 会查找 freebsd-ufs 分区中的 /boot/loader，gptzfsboot 会查找 freebsd-zfs 分区中的 /boot/zfsloader。
10. 由于“/boot/(zfs)loader”绑定到存储该文件的分区的文件系统，因此在引导时会解释为 UFS，而根文件系统使用 ZFS 的混合环境会解释为“UFS”。
11. 会查找 freebsd-ufs 或 freebsd-zfs 分区，但其优先级和控制方式较难解释，因此请参考 [参考文献](https://qiita.com/mzaki/items/76acac14c16ac6789e68#loaderefi)。虽然并不复杂，但因为需要对文件系统级别进行调整，所以需要进一步深入了解。
