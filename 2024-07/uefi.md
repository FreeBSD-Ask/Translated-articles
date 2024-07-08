# 使用 BIOS 引导和 UEFI 引导的 GPT 分区的区别和制作方法

* [FreeBSD](https://qiita.com/tags/freebsd)
* [BIOS](https://qiita.com/tags/bios)
* [UEFI](https://qiita.com/tags/uefi)

最后更新于 2020-11-11 发布于 2018-12-13

近年来在 FreeBSD 中，“分区”几乎总是使用 GPT（GUID Partition Table）无论磁盘大小如何，这几乎是不言而喻的选择。但是，“BIOS 引导”和“UEFI 引导”在分区方案上有细微差异^1^。本文将阐明两者之间的区别，并展示可用于引导的分区配置示例。

## 前提条件

本文假定操作的是 SATA 硬盘，设备名称表示为（/dev/）ada0。如果操作的是 SAS/USB 硬盘，则设备名称应为 daｎ；对于 NVMe 硬盘应为 nvdｎ；对于 eMMC 存储则为 mmcsdｎ（其中ｎ为 0 或更大的整数）。

## 共同事项

因为它是 GPT 分区，所以在任何步骤中，都有一些必须执行的命令^ 2^。

```
gpart create -s GPT ada0
```

为了简化讨论，假定要划分如下所示的分区。

| 设备名称 | 分区类型             | 用途            |
| ---------- | ---------------------- | ----------------- |
| ada0p1   | FreeBSD 引导或者 EFI | 引导区域        |
| ada0p2   | 免费 bsd-ufs^ 3^     | 自由的 BSD 領域 |
| ada0p3   | 免费 bsd 交换        | 交換空間        |

在本次講話中，ada0p2 和 ada0p3 被視為相同的共同步驟。換句話說，執行以下命令在「當前時刻無實際用處」^4^^5^^6^。

```
gpart add -t freebsd-ufs  ada0
gpart add -t freebsd-swap ada0
```

## BIOS 启动的情况下

```
gpart add -t freebsd-boot -b 40 -s 984  ada0
```

* 定义“freebsd-boot”分区作为引导分区。如果不是“freebsd-boot”，引导程序将无法找到该区域。
* 从逻辑块地址（LBA）40 处开始，指定起始位置为 40 个扇区（LBA 40）。

  * GPT 分区头大小（GPT 头大小）为 34 扇区^ 7 ^，但对于 4KiB 扇区兼容磁盘，扇区起始位置将被舍入为 8 的倍数。
  * 可能为了考虑这一点，最近 GPT 头大小变成了 40 扇区（从哪个版本开始尚未确认）。
* 个人建议指定区域大小（ -s ）为 984 个扇区。

  * 下一个区域（freebsd-ufs）从 512Kib 起始位置开始有好处 ^ 8 ^。
  * 由于引导加载程序的限制，必须将大小限制在 545Kib 以下（本次大小为 492Kib）。
  * 因此，缩小没有意义，也没有必要进一步扩大。

```
gpart bootcode -b /boot/pmbr -p /boot/gptboot    -i 1 ada0
        または
gpart bootcode -b /boot/pmbr -p /boot/gptzfsboot -i 1 ada0
```

完成引导加载程序的安装。引导加载程序将在ＦｒｅｅＢＳＤ分区^ 9^^ 10^上搜索，并将控制转移到 /boot/loader 或 /boot/zfsloader。

## 对于ＵＥＦＩ启动

```
gpart add -t efi -b 40 -s 409560 ada0
```

* 定义一个名为「efi」的分区作为引导分区。如果不是「efi」，ＵＥＦＩ将无法检测到这个区域。
* 从扇区 40（LBA40）指定开始位置（ -b ）。

  * GPT 分区标头大小（GPT 标头大小）为 34 个扇区^7^，但是对于 4Kib 扇区对齐的磁盘来说，扇区起始位置会被舍入到 8 的倍数。
  * 最近，为了考虑这一点，GPT 标头大小可能变为了 40 个扇区（版本未知）。
* 领域大小的指定（ -s ）建议指定为 40960 个扇区。

  * 对于这个设置，freebsd-ufs 的下一个区域的起始位置将从 200 MiB 的边界开始 ^ 8 ^。
  * 领域大小严格来说略小于 200 MiB（少了 40 个扇区，即 20 KB），不过这个误差可以忽略。
  * 目前（11.2-R 版本）/boot/boot1.efifat 引导区的大小为 800KiB。
  * 在 10.4-R 或 11.1-R 版本中，建议将此区域保留 200MiB，考虑到与其他操作系统（如 macOS）的兼容性设置，因此如果单独启动 FreeBSD，当前的 800KiB 大小是没有问题的。
  * 在 bsdisnstall: increase EFI partition size to 200MB 的讨论中，认识到对于类似固件更新工具的 EFI 应用程序还需要空间，因此认为 800KiB 的大小实在是太少了。

```
newfs_msdos -F 32 -c 1 -L EFISYS /dev/ada0p1
mount -t msdosfs /dev/ada0p1 /mnt
mkdir -p /mnt/EFI/BOOT
cp /boot/loader.efi /mnt/EFI/BOOT/BOOTX64.efi
umount /mnt
        または
gpart bootcode -p /boot/boot1.efifat -i 1 ada0
```

到此为止，启动加载程序的安装就完成了。启动加载程序会查找 FreeBSD 区域^ 11^ 来继续引导过程。

# 常见问题及解答

## 问：怎么跟 Bootable UEFI U 盘或硬盘上的步骤不一样啊？

一切都好。沒問題。時代已經追上我了。

## 問：經過一番調查，似乎對於在 /boot/boot1.efifat 目錄中寫入內容，有許多不同的步驟，哪一個才是正確的？

一切都好。沒問題。但是不建議使用任何這些步驟。

/boot/boot1.efifat 是分区镜像本身。极端来说可以认为是 dd if=/dev/ada0p1 of=/boot/boot1.efifat 的内容。实际在构建时正在做类似的事情。

因此，无论是使用 dd(8)还是 gpart(8)写入都是一样的。但毫无疑问，更智能的选择是使用 gpart(8)。

但最好不要使用/boot/boot1.efifat。关于之前提到的 200 MiB 扩展的问题，使用它会将 200 MiB 区域中的 800 KiB 变成只能访问的区域。

尽管已经保留了 200 MiB 的空间，但却无法运行 EFI 应用程序。

在新安装中没有问题，但如果想要更新，可能会走入不必要的方向而消失。

## 在进行操作系统更新时，是否需要更新引导分区？

 常常需要但并非总是必要。然而…

1. 如果从 UFS 启动，由于 UFS 格式的稳定性，bootloader 代码没有更改，因此几乎不会出错。如果出现问题，可能是在从 UFS1 升级到 UFS2 时。因此，即使未更新，也不会出现问题，或者可能不会被发现。
2. 如果从 ZFS 启动，由于 ZFS 格式的不稳定性，bootloader 代码会发生变化，因此需要经常维护，否则可能无法启动。在这方面， /usr/src/UPDATING 的 ZFS notes 中提到了“当升级启动的 ZFS 池版本时，需要更新 bootloader”的说明。

从这个角度来看，更新引导程序的步骤也应该进行系统化。

对于 freebsd-boot 分区，只需使用 gpart bootcode -p 进行覆盖即可，但对于 efi 分区，则需要在挂载后，使用 cp 进行覆盖（详见后文）。

## 问：如果分区号不是 1，那么任何号码都可以吗？

 不要紧，最好不要做。

MBR 有 2 TiB 的限制，如果使用后续区域，BIOS 可能无法找到引导加载程序（freebsd-boot）。此外，如果分区编号与区域顺序不匹配，将会变得很麻烦。

## 我们处理的是 GPT，所以 MBR 应该不相关。

很遗憾。因为 GPT 是 MBR 的更高兼容性版本，所以别想了。

就 UEFI 启动来说，GPT 会被解释为 GPT，所以可以断然地说“无关紧要”。

但对于 BIOS 启动，GPT 会被看作 MBR 来操作。因此，在前 2TiB 的区域内需要存在引导加载程序（freebsd-boot）。不过，仅仅存在是问题的关键，并不关心分区号是多少。

## 问：系统是 UEFI 启动还是 BIOS 启动我不知道！

答：启动 Live CD 后执行 sysctl machdep.bootmethod 命令，可以确认是 UEFI 启动还是 BIOS 启动。

# 【附录】更新引导加载程序

## BIOS 启动时

```
gpart bootcode -p /boot/gptboot    -i 1 ada0
        または
gpart bootcode -p /boot/gptzfsboot -i 1 ada0
```

## UEFI 启动时

首先，在 /etc/fstab 中添加以下设置。

```
# Device    Mountpoint FStype  Options Dump Pass #
/dev/ada0p1 /boot/efi  msdosfs rw      0    0
```

接下来创建 /boot/efi 目录。

```
mkdir -p /boot/efi
```

手动挂载一次，当然也可以忽略重启后的问题。

```
mount /boot/efi
```

 以后的更新如下。

```
cp /boot/loader.efi /boot/efi/EFI/BOOT/BOOTX64.efi
```

# 参考文献

* [ gpart(8)的手册](https://www.freebsd.org/cgi/man.cgi?gpart(8))
* [ FreeBSD 的启动过程](https://qiita.com/mzaki/items/76acac14c16ac6789e68)
* [UEFI - FreeBSD 维基](https://wiki.freebsd.org/UEFI)
* [FreeBSD 10.4 发布说明](https://www.freebsd.org/releases/10.4R/relnotes.html)
* [FreeBSD 11.1 发布说明](https://www.freebsd.org/releases/11.1R/relnotes.html)
* [bsdinstall：将 EFI 分区大小增加到 200MB](https://reviews.freebsd.org/D6935)
* [ GPT 和 MBR 有什么区别？](http://syuu1228.hatenablog.com/entry/20130103/1357165915)
* [通过命令行创建 FreeBSD 启动分区等](http://nao550.hateblo.jp/entry/2018/02/12/214146)
* [ 有关 FreeBSD/UEFI](http://zenno.com/pukiwiki/index.php?FreeBSD%2FUEFI%A4%CB%A4%C4%A4%A4%A4%C6)

---

1. 在 BIOS 中通常被称为“MBR（Master Boot Record）引导”，但由于 2TB 限制和几乎消失了多重引导需求，现在已经统一为 GPT 启动（？）
2. 此命令的执行无需任何更改。从兼容性的角度看，根本不应修改默认值。
3. 虽然有时可能想将 freebsd-ufs 更改为 freebsd-zfs，但本次讨论将略过此内容。
4. 实际上，需要使用 -s サイズ 选项来指定大小。如果未指定，则表示将剩余的全部内容作为意味着不能分配交换空间。
5. 注意执行顺序。如果继续进行，将无法分配启动区域（分区编号将错一位）。
6. 尽管可以使用 -i パーティション番号 或 -b 開始位置 -i パーティション番号 选项进行控制，但作为基本说明，我们愿意放弃。
7. LBA 0 到 LBA 33 是 GPT 头区域。
8. 近来，各种媒体（包括 4Kibytes HDD 和 NAND Flash 等）中充当边界的最小公倍数（即使有 512KiB，也适用于任何媒体）并不是一个糟糕的数字（应该）。
9. gptboot 在 freebsd-ufs 区域中查找 /boot/loader，gptzfsboot 在 freebsd-zfs 区域中查找 /boot/zfsloader。 ↩
10. 仅与存储着"/boot/(zfs)loader"的领域的文件系统相关联，因此在引导时会被视为 UFS，而在根目录中会被视为 ZFS，因此对于混合环境中的解释将作为"UFS"来处理。
11. 尽管在搜索 freebsd-ufs 或 freebsd-zfs 领域时，它们的优先级和控制在这里很难解释，因此请参考参考资料。虽然并不复杂。但是由于需要对文件系统级别进行调整，因此需要深入了解一下。
