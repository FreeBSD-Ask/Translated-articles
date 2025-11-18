# 在 FreeBSD 上以 lsblk(8) 风格列出块设备

- [List Block Devices on FreeBSD lsblk(8) Style](https://vermaden.wordpress.com/2019/09/27/list-block-devices-on-freebsd-lsblk8-style/)
- 作者：𝚟𝚎𝚛𝚖𝚊𝚍𝚎𝚗
- 2019/09/27

当我需要在 Linux 系统上工作时，我通常会怀念很多 FreeBSD 上很棒的工具，比如仅举几例：

* **sockstat**
* **gstat**
* **top -b -o res**
* **top -m io -o total**
* **usbconfig**
* **rcorder**
* **beadm**/**bectl**
* **idprio**/**rtprio**

……但有时候——虽然很少——Linux 上也有一些非常有用的工具，而 FreeBSD 上则没有。例子就是 **lsblk(8)**，它专注于一件事，而且做得相当好——列出块设备及其内容。它也有一些问题，比如在完全应用 ZFS 池使用的磁盘上，它会显示两个分区，而不是直接显示 ZFS 的信息——但我们都知道，在某些圈子里，CDDL 许可的 ZFS 在那个 GPL 世界里是多么地“不受待见”。

来自 Linux 系统的 **lsblk(8)** 示例输出：

```sh
$ lsblk
NAME                         MAJ:MIN RM   SIZE RO TYPE   MOUNTPOINT
sr0                           11:0    1  1024M  0 rom
sda                            8:0    0 931.5G  0 disk
|-sda1                         8:1    0   500M  0 part   /boot
`-sda2                         8:2    0   931G  0 part
  |-vg_local-lv_root (dm-0)  253:0    0    50G  0 lvm    /
  |-vg_local-lv_swap (dm-1)  253:1    0  17.7G  0 lvm    [SWAP]
  `-vg_local-lv_home (dm-2)  253:2    0   1.8T  0 lvm    /home
sdc                            8:32   0 232.9G  0 disk
`-sdc1                         8:33   0 232.9G  0 part
  `-md1                        9:1    0 232.9G  0 raid10 /data
sdd                            8:48   0 232.9G  0 disk
`-sdd1                         8:49   0 232.9G  0 part
  `-md1                        9:1    0 232.9G  0 raid10 /data
```

而 FreeBSD 在这方面提供了什么？可以使用命令 **camcontrol(8)** 和 **geom(8)**。你也可以使用 **gpart(8)** 命令来列出分区。下面是我在单磁盘笔记本上运行这些命令的输出。

```sh
# camcontrol devlist
<Samsung SSD 860 EVO mSATA 1TB RVT41B6Q>  at scbus1 target 0 lun 0 (ada0,pass0)

% geom disk list
Geom name: ada0
Providers:
1. Name: ada0
   Mediasize: 1000204886016 (932G)
   Sectorsize: 512
   Mode: r1w1e2
   descr: Samsung SSD 860 EVO mSATA 1TB
   lunid: 5002538e402b4ddd
   ident: S41PNB0K303632D
   rotationrate: 0
   fwsectors: 63
   fwheads: 1

# gpart show
=>        40  1953525088  ada0  GPT  (932G)
          40      409600     1  efi  (200M)
      409640        1024     2  freebsd-boot  (512K)
      410664         984        - free -  (492K)
      411648  1953112064     3  freebsd-zfs  (931G)
  1953523712        1416        - free -  (708K)
```

它们以可接受的方式提供了所需信息，但仅适用于磁盘数量较少的系统。如果你想显示所有系统驱动器内容的汇总呢？这时 **lsblk.sh** 就派上用场了。虽然 **lsblk(8)** 拥有许多有趣的功能，如 **--perms**/**--scsi**/**--inverse** 模式，我这里重点提供的只是最基本的功能——列出系统块设备及其内容。由于我在编写 shell 脚本方面有长期且愉快的经验，例如 [**sysutils/beadm**](https://vermaden.wordpress.com/2018/11/15/zfs-boot-environments-reloaded-at-nluug-autumn-conference-2018/) 或 [**sysutils/automount**](https://vermaden.wordpress.com/2018/10/11/freebsd-desktop-part-17-automount-removable-media/)，我认为编写 **lsblk.sh** 是一个不错的主意。我实际上在 2016 年就在这个主题 [**lsblk(8)** Command for FreeBSD](https://forums.freebsd.org/threads/lsblk-8-command-for-freebsd.56008/) 的 *FreeBSD Forums* 上‘开源’或说分享了这个项目/想法，但由于时间有限，这个“副项目”的开发进度非常缓慢。我最终重新回到它，完成了它。

**lsblk.sh** 是一个总体上小巧且简单的 shell 脚本，代码行数不到四百行。

<img width="662" height="701" alt="image" src="https://github.com/user-attachments/assets/150fb2c9-10da-4c2f-be77-a96e5a3e2281" />

下面是我在单硬盘笔记本上运行 **lsblk.sh** 命令的示例输出。

```sh
% lsblk.sh
DEVICE         MAJ:MIN  SIZE TYPE                      LABEL MOUNT
ada0             0:5b  932G GPT                           - -
  ada0p1         0:64  200M efi                    efiboot0 <UNMOUNTED>
  ada0p2         0:65  512K freebsd-boot           gptboot0 -
  <FREE>         -:-   492K -                             - -
  ada0p3         0:66  931G freebsd-zfs                zfs0 <ZFS>
  <FREE>         -:-   708K -                             - -
```

同样的输出在图形窗口中显示。

![](https://vermaden.wordpress.com/wp-content/uploads/2019/09/lolcat.png?w=960)

下面是来自一台拥有两块系统固态硬盘（**da0**/**da1**）和两块机械数据盘（**da2**/**da3**）的服务器的 **lsblk.sh** 输出示例。

```sh
# lsblk.sh
DEVICE         MAJ:MIN SIZE TYPE                      LABEL MOUNT
da0              0:be  224G GPT                           - -
  da0p1          0:15a 200M efi                    efiboot0 <UNMOUNTED>
  da0p2          0:15b 512K freebsd-boot           gptboot0 -
  <FREE>         -:-   492K -                             - -
  da0p3          0:15c 2.0G freebsd-swap              swap0 <UNMOUNTED>
  da0p4          0:15d 221G freebsd-zfs                zfs0 <ZFS>
  <FREE>         -:-   580K -                             - -
da1              0:bf  224G GPT                           - -
  da1p1          0:16a 200M efi                    efiboot1 <UNMOUNTED>
  da1p2          0:16b 512K freebsd-boot           gptboot1 -
  <FREE>         -:-   492K -                             - -
  da1p3          0:16c 2.0G freebsd-swap              swap1 <UNMOUNTED>
  da1p4          0:16d 221G freebsd-zfs                zfs1 <ZFS>
  <FREE>         -:-   580K -                             - -
da2              0:c0   11T GPT                           - -
  da2p1          0:16e  11T freebsd-zfs                   - <ZFS>
  <FREE>         -:-   1.0G -                             - -
da3              0:c1   11T GPT                           - -
  da3p1          0:16f  11T freebsd-zfs                   - <ZFS>
  <FREE>         -:-   1.0G -                             - -
```

下面是我在其他系统上测试 **lsblk.sh** 时的其他示例。

<img width="376" height="538" alt="image" src="https://github.com/user-attachments/assets/4535bb72-87e2-445f-92a0-201199e25f0f" />

虽然 **lsblk.sh** 并不是地球上最快的脚本（因为需要进行大量解析），但它能够很好地完成工作。如果你想在系统中安装它，只需输入以下命令：

```sh
# fetch -o /usr/local/bin/lsblk https://raw.githubusercontent.com/vermaden/scripts/master/lsblk.sh
# chmod +x /usr/local/bin/lsblk
# hash -r || rehash
# lsblk
```

如果有时间，我可以考虑在 **lsblk.sh** 脚本中添加哪些其他原创的 Linux **lsblk(8)** 子命令/选项/参数呢？:🙂:

此致敬礼。

## 更新 1 – 添加 USAGE/HELP 信息

刚刚添加了一些用法信息，可以通过以下任意参数显示：

* **h**
* **-h**
* **--h**
* **help**
* **-help**
* **--help**

依我看，为这么一个简单的工具写 man 页面是没有必要的。我想等 **lsblk.sh** 工具在功能和选项上扩展到可与 Linux **lsblk(8)** 相媲美时，再创建专门的 man 页面。下面是它的显示效果。

```sh
# lsblk.sh --help
usage:

  BASIC USAGE INFORMATION
  =======================
  # lsblk.sh [DISK]

example(s):

  LIST ALL BLOCK DEVICES IN SYSTEM
  --------------------------------
  # lsblk.sh
  DEVICE         MAJ:MIN SIZE TYPE                      LABEL MOUNT
  ada0             0:5b  932G GPT                           - -
    ada0p1         0:64  200M efi                    efiboot0 <UNMOUNTED>
    ada0p2         0:65  512K freebsd-boot           gptboot0 -
    <FREE>         -:-   492K -                             - -
    ada0p3         0:66  931G freebsd-zfs                zfs0 <ZFS>

  LIST ONLY da1 BLOCK DEVICE
  --------------------------
  # lsblk.sh da1
  DEVICE         MAJ:MIN SIZE TYPE                      LABEL MOUNT
  da1              0:80  2.0G MBR                           - -
    da1s1          0:80  2.0G freebsd                       - -
      da1s1a       0:81  1.0G freebsd-ufs                root /
      da1s1b       0:82  1.0G freebsd-swap               swap SWAP

hint(s):

  DISPLAY ALL DISKS IN SYSTEM
  ---------------------------
  # sysctl kern.disks
  kern.disks: ada0 da0 da1
```

此致敬礼。

## 更新 2 – 代码重组与重写 75% 

……至少这是 **git(1)** 在 **commit** 信息中告诉我的内容。

```sh
% git commit (...)
[master 12fd4aa] Rework entire flow. Split code into functions. Add many useful comments. In other words its 2.0 version.
 1 file changed, 494 insertions(+), 505 deletions(-)
 rewrite lsblk.sh (75%)
```

经过几个高效小时的工作，**lsblk.sh** 的新版本现已发布。

它的源代码行数类似，但现在缩小了四分之一……同时功能更多、准确性更高。这是 *“少即是多”* 的绝佳例子。

```sh
% wc scripts/lsblk.sh.OLD
     491    2201   19721 scripts/lsblk.sh.OLD

% wc scripts/lsblk.sh
     494    1871   15472 scripts/lsblk.sh
```

一些没有简单解决方案的问题如下所述。

其中之一是 FAT 文件系统的“双重”标签。我们既有 **/dev/gpt/efiboot0** 标签，也有 FAT 标签 **EFISYS**。必须在两者中做出选择。由于并非所有 FAT 文件系统都有标签，我选择了 GPT 标签。

```sh
% glabel status | grep ada0p1
  gpt/efiboot0     N/A  ada0p1
msdosfs/EFISYS     N/A  ada0p1
```

我也无法覆盖 FUSE 挂载。当你挂载——例如——/dev/da0 设备为 NTFS（使用 **ntfs-3g**）或 exFAT（使用 **mount.exfat**）时，**mount(8)** 输出没有明显区别。

```sh
% mount -t fusefs
/dev/fuse on /mnt/ntfs (fusefs)
/dev/fuse on /mnt/exfat (fusefs)
```

当我通过守护进程（如 **sysutils/automount**）挂载此类文件系统时，我会在 **/var/run/automount.state** 文件中记录设备挂载到的目录。然后，当我收到 **/dev/da0** 设备的 **detach** 事件时，我就知道该卸载哪个挂载点……但当只有 **/dev/fuse** 设备时，这是不可能的。

……或者，也许你知道有什么方法可以从 **/dev/fuse**（或 FUSE 一般）中提取设备挂载位置的信息吗？

下面展示更新后的效果。

这里是各种非 ZFS 文件系统的挂载情况：

```sh
% mount -t nozfs
devfs on /dev (devfs, local, multilabel)
linprocfs on /compat/linux/proc (linprocfs, local)
tmpfs on /compat/linux/dev/shm (tmpfs, local)
/dev/label/ASD on /mnt/tmp (msdosfs, local)
/dev/fuse on /mnt/ntfs (fusefs)
/dev/md0s1f on /mnt/ufs.other (ufs, local)
/dev/gpt/OTHER on /mnt/fat.other (msdosfs, local)
/dev/md0s1a on /mnt/ufs (ufs, local)
```

……现在 **lsblk.sh** 显示它们的方式如下。

```sh
% lsblk.sh
DEVICE         MAJ:MIN SIZE TYPE                      LABEL MOUNT
ada0             0:56  932G GPT                           - -
  ada0p1         0:64  200M efi                gpt/efiboot0 -
  ada0p2         0:65  512K freebsd-boot       gpt/gptboot0 -
  <FREE>         -:-   492K -                             - -
  ada0p3         0:66  931G freebsd-zfs                   - <ZFS>
  <FREE>         -:-   708K -                             - -
md0              0:28f 1.0G MBR                           - -
  md0s1          0:294 512M freebsd                       - -
    md0s1a       0:29a 100M freebsd-ufs                root /mnt/ufs
    md0s1b       0:29b  32M freebsd-swap         label/swap SWAP
    md0s1e       0:29c  64M freebsd-ufs                   - -
    md0s1f       0:29d 316M freebsd-ufs                   - /mnt/ufs.other
  md0s2          0:296 256M ntfs                          - -
  md0s3          0:297 256M fat32               msdosfs/ONE -
md1              0:2a4 1.0G msdosfs                   LARGE 
md2              0:298 2.0G GPT                           - -
  md2p1          0:29f 2.0G ms-basic-data         gpt/OTHER /mnt/fat.other
```

我为此使用了一些基于文件的内存设备。现在，默认情况下 **lsblk.sh** 也会显示内存磁盘的内容。

```sh
% mdconfig.sh -l
md0     vnode    1024M  /home/vermaden/FILE     
md2     vnode    2048M  /home/vermaden/FILE.GPT 
md1     vnode    1024M  /home/vermaden/FILER
```

下面是在 **xterm(1)** 终端中的显示效果。

![](https://vermaden.wordpress.com/wp-content/uploads/2019/09/lsblk.2.0.png?w=960)

此致敬礼。

## 更新 3 – 添加 **geli(8)** 支持

我认为添加 **geli(8)** 支持可能会很有用。最新的 **lsblk.sh** 版本现在避免了 **MOUNT** 和 **LABEL** 检测的代码重复（已移入单一统一函数）。同时添加了更多注释以提高代码可读性，并进行了一些小修复……而且脚本再次变得更小 :🙂:

```sh
% wc lsblk.sh.1.0
     491    2201   19721 lsblk.sh.1.0

% wc lsblk.sh.2.0
     493    1861   15415 lsblk.sh.2.0

% wc lsblk.sh
     488    1820   15332 lsblk.sh
```

此次更新大约修改了 40%（根据 **git commit** 显示：*191 行新增，196 行删除*）。

```sh
# git commit (...)
[master ec9985a] Add geli(8) support. Avoid code duplication and move MOUNT/LABEL detection into function. More comments. Minor fixes.
 1 file changed, 191 insertions(+), 196 deletions(-)
```

还忘了提到，现在得益于智能优化（比如避免重复操作，并将 **grep(1) | awk(1)** 管道聚合为单个 **awk(1)** 查询），**lsblk.sh** 的运行速度比最初版本快了三倍 :🙂:

下面是添加 **geli(8)** 支持后的新输出。

![](https://vermaden.wordpress.com/wp-content/uploads/2019/09/lsblk.2.1.geli_.png?w=960)

此致敬礼。

## 更新 4 – 添加 **fuse(8)** 支持

如我在 **更新 2** 中所述，跟踪 **fuse(8)** 下的挂载设备及其挂载位置非常困难，因为挂载完成后，所有挂载的设备都会神奇地变成 **/dev/fuse**。

经过一些研究，我发现这个信息（在 FreeBSD 下通过 **fuse(8)** 接口实际挂载的设备位置）可以在挂载 **procfs** 文件系统于 **/proc** 后获取。你只需要查看所有 **ntfs-3g** 进程的 **cmdline** 条目。虽然不完美，但至少可以获取到这些信息。

```sh
# mount -t procfs proc /proc

# ps ax | grep ntfs-3g
45995  -  Is      0:00.00 ntfs-3g /dev/md1s2 /mnt/ntfs
59607  -  Is      0:00.00 ntfs-3g /dev/md3 /mnt/ntfs.another
83323  -  Is      0:00.00 ntfs-3g /dev/md3 /mnt/ntfs.another

# pgrep ntfs-3g
59607
83323
45995

% pgrep ntfs-3g | while read I; do cat /proc/$I/cmdline; echo; done
ntfs-3g/dev/md3/mnt/ntfs.another
ntfs-3g/dev/md3/mnt/ntfs.another
ntfs-3g/dev/md1s2/mnt/ntfs
```

这是用于检测 **fuse(8)** 挂载点的代码原型。

```sh
    if [ -e /proc/0/status ]
    then
      FUSE_MOUNTS=$(
        while read PID
        do
          cat /proc/${PID}/cmdline
          echo
        done << ________EOF
          $( pgrep ntfs-3g )
________EOF
)
      FUSE_MOUNTS=$( echo "${FUSE_MOUNTS}" | sort -u )
      FUSE_MOUNTS=$( echo "${FUSE_MOUNTS}" | sed 's|ntfs-3g||g' )
      FUSE_CHECKS=$( echo "${FUSE_MOUNTS}" | grep /dev/${TARGET}/ )
      if [ "${FUSE_CHECKS}" != "" ]
      then
        MOUNT=$( echo "${FUSE_CHECKS}" | sed "s|/dev/${TARGET}||g" )
      fi
    fi
  fi
```

……我刚刚意识到，我找到了获取该信息的新方法（更好），无需挂载 **/proc** 文件系统——你只需要显示 **ntfs-3g** 进程及其命令行参数，例如如下方式：

```sh
% ps -p $( pgrep ntfs-3g | tr '\n' ',' | sed '$s/.$//' ) -o command | sed 1d
ntfs-3g /dev/md1s2 /mnt/ntfs
ntfs-3g /dev/md3 /mnt/ntfs.another
ntfs-3g /dev/md3 /mnt/ntfs.another
```

因此，在我考虑到这最初仅针对 NTFS（**ntfs-3g(8)** 进程）后，我也添加了 exFAT 支持，通过搜索 **mount.exfat** 的 PID。现在 **fuse(8)** 挂载点检测可以同时支持 NTFS 和 exFAT 文件系统……而且支持该功能的代码甚至更简洁。

```sh
  # 尝试从进程中获取 fuse(8) 挂载点
  if [ "${MOUNT_FOUND}" != "1" ]
  then
    FUSE_PIDS=$( pgrep mount.exfat ntfs-3g | tr '\n' ',' | sed '$s/.$//' )
    FUSE_MOUNTS=$( ps -p "${FUSE_PIDS}" -o command | sed 1d | sort -u )
    MOUNT=$( echo "${FUSE_MOUNTS}" |  grep "/dev/${TARGET} " | awk '{print $3}' )
  fi
```

我还修改了 MAJOR 和 MINOR 号的显示方式——从 HEX 改为 DEC，就像 Linux 一样。FreeBSD *Base System* 的 **ls(1)** 会以 HEX 显示，例如你会得到 **0x2af** 值：

```
% ls -l /dev/md4
crw-rw----  1 root  operator  0x2af 2019.09.29 05:18 /dev/md4
```

但使用 FreeBSD Ports 中的 GNU 等价工具 **gls(1)**（来自 **sysutils/coreutils** 包），则以 DEC 值显示 MAJOR 和 MINOR。**gls(1)** 只是 Linux 世界的 **ls(1)**，由于 FreeBSD 的 Base System 已有 **ls(1)**，开发者在名称前加了 ‘g’（代表 GNU）来区分。

```
% gls -l /dev/md4
crw-rw---- 1 root 2, 175 2019-09-29 05:18 /dev/md4
```

使用 **stat(1)** 工具也可以更方便/快速获取：

```
MAJ=$( stat -f "%Hr" /dev/${DEV} )
MIN=$( stat -f "%Lr" /dev/${DEV} )
```

最新的 **lsblk.sh** 如下所示：

![](https://vermaden.wordpress.com/wp-content/uploads/2019/09/lsblk.2.3.fuse_.ntfs_.exfat_.png?w=960)

这也是我还没有将 **lsblk.sh** 添加到 FreeBSD Ports 的原因——几天内就发布了几版带有重要新功能的版本 :🙂:

此致敬礼。

## 更新 5 – 再次重写 69% 

在进一步研究 **gpart(8)** 后，我发现使用参数 **-p** 是个重大改变。使用 **-p** 参数后，它会直接显示带有分区名的输出，不再需要自己寻找 **PREFIX** 并“创建”分区名。

默认 **gpart(8)** 输出：

```sh
# gpart show md0
=>     63  2097089  md0  MBR  (1.0G)
       63  1048576    1  freebsd  (512M)
  1048639   524288    2  ntfs  (256M)
  1572927   524225    3  fat32  (256M)
```

使用参数 **-p** 的输出：

```sh
# gpart show -p md0
=>     63  2097089    md0  MBR  (1.0G)
       63  1048576  md0s1  freebsd  (512M)
  1048639   524288  md0s2  ntfs  (256M)
  1572927   524225  md0s3  fat32  (256M)
```

这一发现导致 **lsblk.sh** 进行了相当大幅重写。**git commit** 估计此次重写达 69%：

```sh
# git commit (...)
(...)
 1 file changed, 487 insertions(+), 501 deletions(-)
 rewrite lsblk.sh (69%)
```

最新 **lsblk.sh** 的特性如下：

* 修复了之前的 BUG。
* 可以检测 exFAT 标签。
* 运行速度提升 20%。
* SLOC 减少 10%。
* 代码量减少 15%。
* 正确处理整个设备上的 **bsdlabel(8)**。
* 正确处理整个设备上的 exFAT。

代码差异如下：

```sh
# wc lsblk.sh
     487    1791   13705 lsblk.sh

# wc lsblk.sh.OLD
     544    1931   16170 lsblk.sh.OLD
```

最新 **lsblk.sh** 看起来仍然如以前，但我现在用 ‘**-**’ 替代了以前的 ‘**<UNMOUNTED>**’ 标记：

![](https://vermaden.wordpress.com/wp-content/uploads/2019/09/lsblk.2.5.gpart_.exfat_.png?w=960)

## 更新 6 – 新的更新与修复版本

**lsblk.sh** 已更新至 3.4 版本——并已在 FreeBSD Ports 树中更新，在 **sysutils/lsblk** Port 中可用。

该版本的 *变更日志* 如下：

* 在磁盘列表中添加 **sysctl -n kern.disks**。
* 在 **__gpart_present** 函数中重置 **LABEL**。
* 修复 **gpart(8)** 的 **[bootme]** 和 **[bootonce]** 标志行为。
* 禁用 **GPTID** 显示标签。
* 添加 **-d|–disks** 选项，仅列出整个磁盘。

请注意，**lsblk.sh** 使用 **diskinfo(8)**，为了正常工作，你需要属于 **operator** 组。你可以这样将自己添加到该组：

```
# pw groupmod operator -m yourself
```

……或者通过编辑 **/etc/group** 文件。

示例输出如下：

![](https://vermaden.wordpress.com/wp-content/uploads/2019/09/lsblk.update6.png?w=960)

## 更新 7 – 更多修复

**lsblk.sh** 已更新至 3.5 版本——并已在 FreeBSD Ports 树中更新，在 **sysutils/lsblk** Port 中可用。

该版本的 *变更日志* 如下：

* 列出磁盘时移除输出中的控制序列和颜色。
* **diskinfo(8)** 仅用于 **md(4)** 磁盘，因为 **geom(4)** 不支持它们。
* 添加新注释并重新整理部分旧注释。
* 为 SIZE 收集和显示增加额外检查。
* 当整个设备没有分区时，正确显示 exFAT 文件系统标签。
* 修复 NTFS-3G 挂载点显示。
* 检查 **automount(8)** 的 **/var/run/automount.state** 以处理 **fusefs(5)** 文件系统。

仅 **md(4)** 磁盘的大小需要属于 **operator** 组。所有其他磁盘大小现在通过 **geom(8)** 命令收集。

## 更新 8 – 优化标签

现在 3.9 版本的 **lsblk(8)** 比以往更好。

![](https://vermaden.wordpress.com/wp-content/uploads/2019/09/lsblk.update8.png)

该版本的 *变更日志* 如下：

* 改进 **glabel(8)** 标签搜索。
* 改进 **gpart(8)** 输出标签处理。
* 将冗长的 Microsoft 分区名称替换为合理名称。
* 改进 exFAT 处理。
* 改进 **md(4)** 磁盘大小处理。
* 移除标签中的冗余空格。
* 为 **-d** 选项添加 **TOTAL SYSTEM STORAGE**。
* 移除主设备检查循环中的子 shell。

已提交 PR [283268](https://bugs.freebsd.org/bugzilla/show_bug.cgi?id=283268) ，因此在 Ports 中更新前请稍作等待。


