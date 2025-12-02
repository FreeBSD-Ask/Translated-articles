# ZFS 启动环境中的其他 FreeBSD 版本

- [Other FreeBSD Version in ZFS Boot Environment](https://vermaden.wordpress.com/2021/10/19/other-freebsd-version-in-zfs-boot-environment/)
- 发布时间：2021/10/19
- 作者：𝚟𝚎𝚛𝚖𝚊𝚍𝚎𝚗

FreeBSD 12.3-PRERELEASE 的首批快照终于 [可用](https://lists.freebsd.org/archives/freebsd-snapshots/2021-October/000005.html) 了。这意味着我们能在一个新的 ZFS 启动环境中尝试它们，而无需打扰当前运行的 13.0-RELEASE 系统。我们无法像平常那样从当前 ZFS 启动环境创建新的启动环境并将其升级到新版本，因为 12.3 的主版本比 13.0 还要旧（译注：旧版 FreeBSD 不识别新版的 ZFS）。

在 FreeBSD 发布流程中，这有点悖论：当 12.3-RELEASE 发布时，它可能包含一些比今年早些时候发布的 13.0-RELEASE 更新的提交和功能。当然，并非所有提交到 HEAD 的内容都会自动进入 12-STABLE 或 13-STABLE，但大多数都会。只有最大的变更会被限制在 14.0-RELEASE，当然，这大概会在 2022 年中期其发布流程进行时出现。

关于 FreeBSD 上的 ZFS 文件系统，有一点需要注意。人们常常将“真正的” ZFS 启动环境与它的替代品混淆，比如 Btrfs 快照或 Ubuntu 使用 **zsysctl(8)** 命令管理的快照。不幸的是，它们只是快照，并非完整的可写克隆（或完整独立的 ZFS 数据集）。它们可以冻结系统状态，从而在更新软件包后能够恢复到工作配置，但你无法像创建另一个独立的 ZFS 启动环境那样，安装其他独立版本的系统作为新的 ZFS 数据集。

## 创建新的 ZFS 数据集

```sh
host # beadm list
BE             Active Mountpoint  Space Created
13.0.w520      NR     /           12.8G 2021-09-14 17:27
13.0.w520.safe -      -            1.2G 2021-10-18 10:01

host # zfs list -r zroot/ROOT
NAME                        USED  AVAIL     REFER  MOUNTPOINT
zroot/ROOT                 12.8G  96.8G       88K  none
zroot/ROOT/13.0.w520       12.8G  96.8G     11.6G  /
zroot/ROOT/13.0.w520.safe     8K  96.8G     11.1G  /

host # zfs create -o mountpoint=/ -o canmount=off zroot/ROOT/12.3

host # beadm list
BE             Active Mountpoint  Space Created
13.0.w520      NR     /           12.8G 2021-09-14 17:27
13.0.w520.safe -      -            1.2G 2021-10-18 10:01
12.3           -      -           96.0K 2021-10-18 13:14
```


## 安装 FreeBSD 12.3-PRERELEASE

```sh
host # beadm mount 12.3 /var/tmp/12.3
Mounted successfully on '/var/tmp/12.3'

host # beadm list
BE             Active Mountpoint     Space Created
13.0.w520      NR     /              12.8G 2021-09-14 17:27
13.0.w520.safe -      -               1.2G 2021-10-18 10:01
12.3           -      /var/tmp/12.3  96.0K 2021-10-18 13:14

host # curl -o - https://download.freebsd.org/ftp/snapshots/amd64/12.3-PRERELEASE/base.txz \
         | tar --unlink -xpf - -C /var/tmp/12.3
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  173M  100  173M    0     0  1889k      0  0:01:33  0:01:33 --:--:-- 2228k

host # exa -1 /var/tmp/12.3
bin
boot
dev
etc
lib
libexec
media
mnt
net
proc
rescue
root
sbin
tmp
usr
var
COPYRIGHT
sys

host # curl -o - https://download.freebsd.org/ftp/snapshots/amd64/12.3-PRERELEASE/kernel.txz \
         | tar --unlink -xpf - -C /var/tmp/12.3
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 43.3M  100 43.3M    0     0  1733k      0  0:00:25  0:00:25 --:--:-- 1663k

host # exa -lh /var/tmp/12.3/boot/kernel/kernel
Permissions Size User Date Modified    Name
.r-xr-xr-x   37M root 2021-10-14 06:31 /var/tmp/12.3/boot/kernel/kernel

host # curl -o - https://download.freebsd.org/ftp/snapshots/amd64/12.3-PRERELEASE/lib32.txz \
         | tar --unlink -xpf - -C /var/tmp/12.3

host # exa -ld /var/tmp/12.3/usr/lib32
drwxr-xr-x - root 2021-10-18 13:45 /var/tmp/12.3/usr/lib32
```

## 安装与主机相同的 Packages

使用 **pkg prime-list**，我们可以获取当前运行系统上手动安装的所有 **pkg(8)** 软件包。你也可以省略此步骤，或者只安装你需要的软件包，而不是全部安装。

```sh
host # pkg prime-list > /var/tmp/12.3/pkg.prime-list

host # chroot /var/tmp/12.3 /bin/sh

(BE) # export PS1="BE # "

BE # mount -t devfs devfs /dev

BE # sed -i '' s/quarterly/latest/g /etc/pkg/FreeBSD.conf

BE # pkg install -y $( cat pkg.prime-list )
Bootstrapping pkg from pkg+http://pkg.FreeBSD.org/FreeBSD:12:amd64/latest, please wait...
Verifying signature with trusted certificate pkg.freebsd.org.2013102301... done
Installing pkg-1.17.2...
Extracting pkg-1.17.2: 100%
Updating FreeBSD repository catalogue...
Fetching meta.conf: 100%    163 B   0.2kB/s    00:01
Fetching packagesite.pkg: 100%    6 MiB   1.3MB/s    00:05
Processing entries: 100%
FreeBSD repository update completed. 31294 packages processed.
All repositories are up to date.
Updating database digests format: 100%
pkg: No packages available to install matching 'chromium' have been found in the repositories
pkg: No packages available to install matching 'drm-fbsd13-kmod' have been found in the repositories
pkg: No packages available to install matching 'geany-gtk2' have been found in the repositories
pkg: No packages available to install matching 'ramspeed' have been found in the repositories
pkg: No packages available to install matching 'vim-console' have been found in the repositories
```

如我们所见，在 FreeBSD 13.0-RELEASE 系统中安装的一些软件包，目前在 FreeBSD 12.3-PRERELEASE 系统的 **pkg(8)** “**latest**” 分支中不可用。这种情况有时会发生，例如某些软件包构建失败——但你可以假设这些软件包在一周左右会重新可用，因为 **pkg(8)** 软件包会在 “**latest**” 分支中进行重建。

接下来我们将移除缺失的软件包，并重命名一些在 FreeBSD 12.x 版本中名称可能不同的软件包。

```sh
BE # sed -i '' \
         -e s/drm-fbsd13-kmod/drm-kmod/g \
         -e s/geany-gtk2/geany/g \
         -e s/vim-console/vim-tiny/g \
         pkg.prime-list

BE # pkg install -y $( cat pkg.prime-list | grep -v -e chromium -e ramspeed )
Updating FreeBSD repository catalogue...
FreeBSD repository is up to date.
All repositories are up to date.
The following 1072 package(s) will be affected (of 0 checked):

New packages to be INSTALLED:
        (...)

Number of packages to be installed: 1072

The process will require 11 GiB more space.
2 GiB to be downloaded.
(...)

BE # rm pkg.prime-list
```

一个小时左右后，我们的软件包已经安装完成。

```sh
BE # pkg stats
Local package database:
        Installed packages: 1073
        Disk space occupied: 11 GiB

Remote package database(s):
        Number of repositories: 1
        Packages available: 31294
        Unique packages: 31294
        Total size of packages: 96 GiB
```

## 复制配置文件

现在你可以重启到一个干净且未配置的 FreeBSD 系统，但你也可以从当前正在使用的安装中复制配置文件。以下是我复制的文件。

首先是来自基本系统 **/etc** 和 **/boot** 目录的文件。

```sh
host # for I in /boot/loader.conf        \
                /etc/ttys                \
                /etc/rc.conf             \
                /etc/rc.local            \
                /etc/sysctl.conf         \
                /etc/hosts               \
                /etc/ethers              \
                /etc/fstab               \
                /etc/jail.conf           \
                /etc/make.conf           \
                /etc/src.conf            \
                /etc/devfs.rules         \
                /etc/exports             \
                /etc/resolv.conf         \
                /etc/localtime           \
                /etc/pf.conf             \
                /etc/resolv.conf         \
                /etc/profile             \
                /etc/csh.cshrc           \
                /etc/wpa_supplicant.conf \
                /etc/freebsd-update.conf \
                /etc/motd.template       \
                /etc/motd                \
                /var/cron/tabs/*
       do
         cp "${I}" /var/tmp/12.3/"${I}"
         echo "${I}"
       done
/boot/loader.conf       
/etc/ttys               
/etc/rc.conf            
/etc/rc.local           
/etc/sysctl.conf        
/etc/hosts              
/etc/ethers             
/etc/fstab              
/etc/jail.conf          
/etc/make.conf          
/etc/src.conf           
/etc/devfs.rules           
/etc/localtime          
/etc/pf.conf            
/etc/resolv.conf        
/etc/profile            
/etc/csh.cshrc          
/etc/wpa_supplicant.conf
/etc/freebsd-update.conf
/etc/motd.template      
/etc/motd               
/var/cron/tabs/vermaden
/var/cron/tabs/root
```

接下来是已安装包在 **/usr/local/etc** 目录下的配置文件。

```sh
host # for I in /usr/local/etc/X11/xdm/Xresources \
                /usr/local/etc/X11/xdm/Xsetup_0   \
                /usr/local/etc/X11/xorg.conf.d/*  \
                /usr/local/etc/devd/*             \
                /usr/local/etc/automount.conf     \
                /usr/local/etc/sudoers            \
                /usr/local/etc/doas.conf          \
                /usr/local/etc/zshrc              \
                /usr/local/etc/smb4.conf          \
                /usr/local/etc/automount.conf     \
                /usr/local/etc/fscd.conf          \
                /usr/local/etc/cups/*             \
                /usr/local/etc/cups/ssl/*         \
                /usr/local/etc/cups/ppd/*
       do
         cp "${I}" /var/tmp/12.3/"${I}"
         echo "${I}"
       done
/usr/local/etc/X11/xdm/Xresources
/usr/local/etc/X11/xdm/Xsetup_0
/usr/local/etc/X11/xorg.conf.d/card.conf
/usr/local/etc/X11/xorg.conf.d/flags.conf
/usr/local/etc/X11/xorg.conf.d/keyboard.conf
/usr/local/etc/X11/xorg.conf.d/touchpad.conf
/usr/local/etc/devd/audio_source.conf
/usr/local/etc/devd/automount_devd.conf
/usr/local/etc/devd/cups.conf
/usr/local/etc/devd/cups.conf.sample
/usr/local/etc/devd/webcamd.conf
/usr/local/etc/automount.conf
/usr/local/etc/sudoers
/usr/local/etc/doas.conf
/usr/local/etc/zshrc
/usr/local/etc/smb4.conf
/usr/local/etc/automount.conf
/usr/local/etc/fscd.conf
/usr/local/etc/cups/classes.conf
/usr/local/etc/cups/command.types
/usr/local/etc/cups/cups-browsed.conf
/usr/local/etc/cups/cups-browsed.conf.sample
/usr/local/etc/cups/cups-files.conf
/usr/local/etc/cups/cups-files.conf.sample
/usr/local/etc/cups/cupsd.conf
/usr/local/etc/cups/cupsd.conf.sample
/usr/local/etc/cups/ppd
/usr/local/etc/cups/printers.conf
/usr/local/etc/cups/printers.conf.O
/usr/local/etc/cups/snmp.conf
/usr/local/etc/cups/snmp.conf.sample
/usr/local/etc/cups/ssl
/usr/local/etc/cups/ppd/HP-M251nw.ppd
/usr/local/etc/cups/ppd/Samsung-ML-1915.ppd
```

## 添加用户并设置密码

现在你应该添加常规用户，并为该用户和 root 账户设置密码。

```sh
BE # pw useradd vermaden -u 1000 -d /home/vermaden -G wheel,operator,video,network,webcamd,vboxusers

BE # passwd root

BE # passwd vermaden
```

## 重启进入新的 ZFS 启动环境

现在你可以退出该 ZFS 启动环境的 **chroot(8)**，然后重启。在 FreeBSD **loader(8)** 菜单中选择 **12.3** 启动环境。

```sh
BE # exit

host # umount /var/tmp/12.3/dev

host # beadm unmount 12.3
Unmounted successfully

host # beadm list -D
BE             Active Mountpoint  Space Created
13.0.w520      NR     /           11.3G 2021-09-14 17:27
13.0.w520.safe -      -           11.1G 2021-10-18 10:01
12.3        -      -            9.5G 2021-10-18 13:14

host # shutdown -r now
```

## 测试新系统

12.3-PRERELEASE 系统对我来说启动正常。我能够登录并像平常一样使用系统。需要注意的一点是 ZFS 池。我有另一个启用了 **zstd** 压缩的新 ZFS 池，但我无法导入该 ZFS 池，因为 FreeBSD 12.3-PRERELEASE 使用的不是 OpenZFS 2.0，而是 FreeBSD 内部的旧版本 ZFS。

```sh
# zpool import data
This pool uses the following feature(s) not supported by this system:
        org.freebsd:zstd_compress (zstd compression algorithm support.)
        com.delphix:log_spacemap (Log metaslab changes on a single spacemap and flush them periodically.)
        org.zfsonlinux:project_quota (space/object accounting based on project ID.)
        org.zfsonlinux:userobj_accounting (User/Group object accounting.)
cannot import 'data': unsupported version or feature
```

请记住这一点……但你也可以从 FreeBSD Ports 安装更新的 OpenZFS，这正是我们接下来要做的。

```sh
# pkg install -y openzfs openzfs-kmod
Updating FreeBSD repository catalogue...
FreeBSD repository is up to date.
All repositories are up to date.
The following 2 package(s) will be affected (of 0 checked):

New packages to be INSTALLED:
        openzfs: 2021090800
        openzfs-kmod: 2021090800

Number of packages to be installed: 2

The process will require 22 MiB more space.
4 MiB to be downloaded.
[1/2] Fetching openzfs-2021090800.pkg: 100%    3 MiB 975.3kB/s    00:03
[2/2] Fetching openzfs-kmod-2021090800.pkg: 100%    1 MiB 591.2kB/s    00:02
Checking integrity... done (0 conflicting)
[1/2] Installing openzfs-kmod-2021090800...
[1/2] Extracting openzfs-kmod-2021090800: 100%
pkg: Cannot open /dev/null:No such file or directory
[2/2] Installing openzfs-2021090800...
[2/2] Extracting openzfs-2021090800: 100%
=====
Message from openzfs-kmod-2021090800:

--
Amend /boot/loader.conf as follows to use this module:

- change zfs_load="YES" to NO
- change opensolaris_load="YES" to NO
- add openzfs_load="YES"
- (for ARM64) add cryptodev_load="YES"
=====
Message from openzfs-2021090800:

--
Ensure that any zfs-related commands, such as zpool, zfs, as used in scripts
and in your terminal sessions, use the correct path of /usr/local/sbin/ and
not the /sbin/ commands provided by the FreeBSD base system.

Consider setting this in your shell profile defaults!
```

接下来我们需要修改 **/boot/loader.conf** 文件。

```sh
host # beadm mount 12.3 /var/tmp/12.3
Mounted successfully on '/var/tmp/12.3'

host # chroot /var/tmp/12.3

BE # cp /boot/loader.conf /boot/loader.conf.ZFS

BE # vi /boot/loader.conf

BE # diff -u /boot/loader.conf.ZFS /boot/loader.conf
--- /boot/loader.conf.ZFS       2021-10-19 10:57:04.180732000 +0000
+++ /boot/loader.conf   2021-10-19 10:57:23.992145000 +0000
@@ -12,7 +12,8 @@

 # MODULES - BOOT
   geom_eli_load=YES
-  zfs_load=YES
+  zfs_load=NO
+  openzfs_load=YES

 # DISABLE /dev/diskid/* ENTRIES FOR DISKS
   kern.geom.label.disk_ident.enable=0

BE # shutdown -r now
```

重启并再次尝试后，我成功导入了那个更新的 ZFS 池。

希望你会觉得这篇指南有用。

欢迎随时提出你的建议。

## 更新 1 – 安装更新版本时的注意事项

本指南撰写时，我尝试在之前使用 FreeBSD 13.0 的系统上安装 FreeBSD 12.3，因此无需更新 *bootcode*。我刚在同一台 13.0 系统上尝试安装 13.1，这时需要执行以下两个步骤来更新 *bootcode*。

### UEFI

对于 UEFI 分区，你需要从 13.1 安装中复制 **/boot/loader.efi** 文件，即 **/var/tmp/13.1** 目录。使用的命令如下。

```sh
host # gpart show -p ada1
=>       40  250069600    ada1  GPT  (119G)
         40     409600  ada1p1  efi  (200M)          <== UEFI BOOT PARTITION
     409640       1024  ada1p2  freebsd-boot  (512K) <== BIOS BOOT PARTITION
     410664        984          - free -  (492K)
     411648    2097152  ada1p3  freebsd-swap  (1.0G)
    2508800  247560192  ada1p4  freebsd-zfs  (118G)
  250068992        648          - free -  (324K)

host # mount_msdosfs /dev/ada1p1 /mnt

host # cp /var/tmp/13.1/boot/loader.efi /mnt/efi/boot/bootx64.efi
```

### BIOS

对于以传统 BIOS 模式启动的系统，你需要使用以下 **gpart(8)** 命令。

```sh
host # cd /var/tmp/13.1/boot
host # pwd
/var/tmp/13.1/boot
host # gpart bootcode -b ./pmbr -p ./gptzfsboot -i 2 ada1
partcode written to ada1p2
bootcode written to ada1
```

由于 FreeBSD 通常安装为 **BIOS+UEFI** 启动模式兼容，因此这两个步骤都需要执行。
